---
title: "Blocking and buffering"
teaching: 10
exercises: 15
questions:
- Is there an easier way to manage multiple processes?
objectives:
- Describe variants on MPI_Ssend
- Describe buffering and blocking
- Use MPI_SendRecv
keypoints:
- There are advanced MPI routines that solve most common problems. Don't reinvent the wheel.
- Function MPI_SendRecv
---

Why did we get a deadlock with periodic boundaries but not when we didn't? To help answer that question lets take a look at the messaging passing process in a little more detail.

## Different ways to send messages
What happens when a message is sent between processes? The data stored in memory of one process must be copied to the memory of the other process. This coping is not instantaneous especially since the two processes are very likely not starting to send and receive the message at exactly the same time. So how is this copying of memory between processes coordinated?

There are two main ways the copying of memory between processes can be coordinated:

1. **Synchronous**: the sending and receiving process must coordinate their sending and receiving so that no buffering of data is needed and a minimal amount of copying of data occurs.
2. **Buffered**: a buffer is used to store the data to be sent until the matching receive call happens.

In addition to synchronous or buffered methods of sending data there is also the question of when the send, or even the receive, function should return. Should they return as quickly as possible or should the return once the messages sending has been completed? This leads to the idea of **blocking**, if the send or receive function waits until the matching send or receive call has reached some point in its execution before returning this is known as a **blocking** send or receive. In the case of buffered sends the **blocking**/**non-blocking** nature refers waiting until data is coped into the buffer and not that the message has been received. For more details on the topics of synchronous, buffered, and blocking point-to-point MPI messages see [MPI: The Complete Reference](http://www.netlib.org/utk/papers/mpi-book/node22.html).

Buffering requires data to be copied into and out of a buffer which means more memory will be used and it will take more processor time to copy the data into a buffer as compared to a synchronous send. However, using a buffer means that you can modify the original data as soon as it has been copied into the buffer. Synchronous sends however, require that the original data being sent is not modified until the data has been sent.

Using **non-blocking** sends and receives allows one to **overlap computations with communication**, however **DO NOT** modify the send buffer or read from the receive buffer until the messages have completed. To know if a message has completed check with [`MPI_Wait`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Wait.3.php) or [`MPI_Test`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Test.3.php). To allow the greatest flexibility to optimize the message passing for particular use cases a number of send and receive functions have been defined which incorporate different combinations of **Synchronous**, **Buffered**, and **Blocking**/**Non-blocking** messaging.

Lets have a look at some of the available send and receive functions.
#### Sends
- [`MPI_Send`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Send.3.php): blocking, buffering or synchronous send. Buffering or synchronous depend on size of message and MPI library implementor's choice.
- [`MPI_Ssend`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Ssend.3.php): blocking, synchronous send with no buffering.
- [`MPI_Bsend`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Bsend.3.php): blocking, buffered send. It will block until the data is copied to the buffer. You are responsible for supplying the buffer with [`MPI_Buffer_attach`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Buffer_attach.3.php) and knowing how big it has to be. **WARNING**: it is bigger than just the data you want to send.
- [`MPI_Isend`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Isend.3.php) : non-blocking, maybe buffered or synchronous. All non-blocking functions require [`MPI_Wait`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Wait.3.php) or [`MPI_Test`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Test.3.php) to test when a message completes so that send data can be modified or freed.
- [`MPI_Rsend`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Rsend.3.php): ready send, meaning a matching receive call must have already been posted at the time the send has been initiated. You are responsible for ensuring this; it is an error if this is not the case.
- [`MPI_Issend`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Ibsend.3.php): non-blocking, synchronous send.
- [`MPI_Ibsend`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Ibsend.3.php): non-blocking, buffered send.
- [`MPI_Irsend`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Irsend.3.php): non-blocking, ready send.

#### Recvs
- [`MPI_Recv`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Recv.3.php): blocking receive.
- [`MPI_Irecv`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Irecv.3.php): non-blocking receive.

Wow there are a lot of different sends. Luckily after this we are going to keep it pretty simple. No buffering or checking for messages to complete will happen in this workshop.

## Why did we deadlock?

Lets examine what happened before we added the periodic boundaries. In this case processor 3 was sending to `MPI_PROC_NULL` while processor 0 was receiving from `MPI_RPOC_NULL`. We used `MPI_Ssend` which is a blocking send. This means that the function will **block** program execution until at least some part of the matching `MPI_Recv` call has been initiated. This means that all processes end up waiting at the `MPI_Ssend` function for the matching `MPI_Recv` to begin. Luckily process 3 was sending to `MPI_PROC_NULL` and so skips the send and proceeds to the `MPI_Recv`. After this each process can proceed in turn after the call to the matching `MPI_Recv` has initiated.

![Results of secondmessage](../fig/sending_right.svg)

With this in mind, what would happen when we don't have any processes sending to `MPI_PROC_NULL`? All process will be waiting in the `MPI_Ssend` functions for the matching `MPI_Recv` function to be called. However, if they are all waiting in the `MPI_Ssend` no processes will be able to call an `MPI_Recv` function as those happen afterwards. This is why our periodic send deadlocked. So how would we fix this?

## How to avoid deadlock?

One way we could avoid the deadlock only using the MPI functions we have already used is to split up the message passing into two sets of messages where we pair send and receives.

1. even ranks send, odd ranks receive
2. odd ranks send, even ranks receive

Another way we could solve this issue is by using the non-blocking sends, however, we would then need to later check for the completeness of the message to continue.

There is another MPI function which is very useful for situations like this [`MPI_Sendrecv`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Sendrecv.3.php). This is a blocking call combining both a send and receive. However the send and receive within that call will not block each other, only that the entire function will block. The send and receive buffers must be different and may not overlap. The source and destination processes do not have to be the same nor do the data type, tag or size of the messages. The send and receive components of this call could be handling two totally unrelated messages, except for perhaps the time at which the both the sending and receiving of those messages occur.

This is typical of MPI; with the very basics you can do almost anything, even if you have to jump through some hoops - but there are often more advanced routines which can help do things more easily and faster.


### `MPI_Sendrecv` = Send + Recv

#### C syntax

![send recieve](../fig/sendrecv_args.png)

#### FORTRAN syntax

![send reciev](../fig/sendrecv_args2.png)

> ## Use `MPI_Sendrecv` to solve deadlock
> ~~~
> $ cp thirdmessage.c fourthmessage.c
> $ nano fourthmessage.c
> ~~~
> {: .language-bash}
> **Remember to replace .c with .f90 if working with Fortran code.**
> > ## Solution
> > ~~~
> >   . . .
> >   msgsent=rank*rank;
> >   msgrcvd=-999.;
> >   int count=1;
> >   ierr=MPI_Sendrecv(&msgsent, count, MPI_DOUBLE, right, tag,
> >                     &msgrcvd, count, MPI_DOUBLE, left,  tag,
> >                     MPI_COMM_WORLD, &rstatus);
> >   
> >   printf("%d: Sent %lf and got %lf\n", rank, msgsent, msgrcvd);
> >   . . .
> > ~~~
> > {: .language-c}
> > 
> > ~~~
> >   . . .
> >   msgsent= rank*rank
> >   msgrcvd= -999.
> >   tag=1
> >   
> >   call MPI_Sendrecv(msgsent, 1, MPI_DOUBLE_PRECISION, right, tag, &
> >                     msgrcvd, 1, MPI_DOUBLE_PRECISION, left,  tag, &
> >                     MPI_COMM_WORLD, status, ierr)
> >   
> >   print *, rank, 'Sent ', msgsent, 'and recvd ', msgrcvd
> >   . . . 
> > ~~~
> > {: .language-fortran}
> > Then compile with either
> > ~~~
> > $ mpicc -o fourthmessage fourthmessage.c
> > ~~~
> > {: .language-bash}
> > **or**
> > ~~~
> > $ mpif90 -o fourthmessage fourthmessage.f90
> > ~~~
> > {: .language-bash}
> > 
> > and run with
> > ~~~
> > $ mpirun -np 4 ./fourthmessage
> > ~~~
> > {: .language-bash}
> > ~~~
> > 0: Sent 0.000000 and got 9.000000
> > 1: Sent 1.000000 and got 0.000000
> > 3: Sent 9.000000 and got 4.000000
> > 2: Sent 4.000000 and got 1.000000
> > ~~~
> > {: .output}
> > 
> {: .solution}
{: .challenge}
