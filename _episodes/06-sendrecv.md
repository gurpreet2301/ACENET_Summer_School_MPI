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
- Functions: MPI_SendRecv
---

Why did we get a deadlock in the last exercise?

Because `MPI_Ssend` **blocks** until the matching `MPI_Recv` has happened.
So everyone was sending but no one was receiving.

Quoting from <a href="http://www.netlib.org/utk/papers/mpi-book/node22.html">MPI: The Complete Reference</a>:

<blockquote>
...The question arises: if the send has completed, does this tell us anything
about the receiving process? Can we know that the receive has finished, or
even, that it has begun?

Such questions of semantics are related to the nature of the underlying
protocol implementing the operations. If one wishes to implement a protocol
minimizing the copying and buffering of data, the most natural semantics might
be the "rendezvous" version, where completion of the send implies the receive
has been initiated (at least). On the other hand, a protocol that attempts to
block processes for the minimal amount of time will necessarily end up doing
more buffering and copying of data and will have "buffering" semantics.

The trouble is, one choice of semantics is not best for all applications, nor
is it best for all architectures. Because the primary goal of MPI is to
standardize the operations, yet not sacrifice performance, the decision was
made to include all the major choices for point to point semantics in the
standard.

The above complexities are manifested in MPI by the existence of modes for
point to point communication. Both blocking and nonblocking communications have
modes. The mode allows one to choose the semantics of the send operation and,
in effect, to influence the underlying protocol of the transfer of data. 

In standard mode the completion of the send does not necessarily mean that the
matching receive has started, and no assumption should be made in the
application program about whether the out-going data is buffered by MPI. In
buffered mode the user can guarantee that a certain amount of buffering space
is available. The catch is that the space must be explicitly provided by the
application program. In synchronous mode a rendezvous semantics between sender
and receiver is used...
</blockquote>

## Different versions of SEND (& RECV)

- Two orthogonal ideas: Buffering and blocking.
- "Blocking" means the routine does not return (it "blocks") until some condition is met. For a send, until the matching recieve happens. For a receive, until the matching send has happened.
- "Buffering" means that the message data is stored somewhere temporary
- SSEND : Synchronous send; doesn't return until receive has started. Blocking, no buffering.
- SEND/RECV: Blocking. Buffering or not, may depend on size of message or implementor's choice
- ISEND/IRECV : Non-blocking, no buffering, but needs MPI_Wait or MPI_Test
- IBSEND (no IBRECV): Unblocking, buffering

## Why not use buffering all the time?
- Buffering poses a subtle kind of danger: It will *usually* work.
- Think voice mail; message sent, reader reads when ready
- But voice mail boxes do fill
- Message fails.
- Program fails/hangs mysteriously.
- You can manage your own buffers if you like.

# Non-blocking
- The non-block (ISEND/IRECV) could be used to to solve the blocking issue.
- Requires use of additional routines
  - MPI_Wait/MPI_Waitall to determine when communication has completed
- Onus on the programmer to ensure that communication is complete before altering buffers (unlike buffered versions).
- Allow overlap of computation with communication (but **DO NOT** alter the buffers being sent until after communication is finished)
- With great power comes great responsibility.
- For this example there are simpler solutions, lets explore some.

# WITHOUT using new MPI routines, how can we fix this?
- How about this?
- First: evens send, odds receive
- Then: odds send, evens receive
- Will this work with an odd # of processes?
- How about 2 procs? 1 proc?

![Re-arranging communications to avoid deadlock](../fig/sendreciev_1.png)

```
program fourthmessage
use mpi
implicit none

    integer :: ierr, rank, comsize
    integer :: left, right
    integer :: tag
    integer :: status(MPI_STATUS_SIZE)
    double precision :: msgsent, msgrcvd
    
    call MPI_Init(ierr)
    call MPI_Comm_size(MPI_COMM_WORLD, comsize, ierr)
    call MPI_Comm_rank(MPI_COMM_WORLD, rank, ierr)

    left = rank-1
    if (left <0) left = comsize-1
    right = rank+1
    if (right >=comsize) right = 0

    msgsent= rank*rank
    msgrcvd= -999.
    tag=1

    if (mod(rank,2) == 0) then
        call MPI_Ssend(msgsent, 1, MPI_DOUBLE_PRECISION,right, &
                      tag, MPI_COMM_WORLD,ierr)  
        call MPI_Recv(msgrcvd, 1, MPI_DOUBLE_PRECISION,left, &
                      tag, MPI_COMM_WORLD,status,ierr)
     else 
        call MPI_Recv(msgrcvd, 1, MPI_DOUBLE_PRECISION,left, &
                      tag, MPI_COMM_WORLD,status,ierr)	    
        call MPI_Ssend(msgsent, 1, MPI_DOUBLE_PRECISION,right, &
                      tag, MPI_COMM_WORLD,ierr)
    end if
 
    print *, rank, 'Sent ', msgsent, 'and recvd ', msgrcvd
    call MPI_Finalize(ierr)
end program fourthmessage
```

- In the `if` block, even ranks send first
- In the `else` block, the odd ranks send

**Following is a C program for fourth message**
  
```
#include <stdio.h>
#include <mpi.h>

int main(int argc, char **argv) {
    int rank, size, ierr;
    int left, right;
    int tag=1;
    double msgsent, msgrcvd;
    MPI_Status rstatus;

    ierr = MPI_Init(&argc, &argv);
    ierr = MPI_Comm_size(MPI_COMM_WORLD, &size);
    ierr = MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    left = rank-1;
    if (left < 0) left =size-1;
    right = rank+1;
    if (right == size) right = 0;
    msgsent = rank*rank;
    msgrcvd = -999.;

    if(rank % 2 ==0) {
	ierr = MPI_Ssend(&msgsent, 1, MPI_DOUBLE, right,tag, MPI_COMM_WORLD); 
        ierr = MPI_Recv(&msgrcvd, 1, MPI_DOUBLE, left,tag, MPI_COMM_WORLD, &rstatus); 
	}
    else {
	ierr = MPI_Recv(&msgrcvd, 1, MPI_DOUBLE, left,tag, MPI_COMM_WORLD, &rstatus); 
	ierr = MPI_Ssend(&msgsent, 1, MPI_DOUBLE, right,tag, MPI_COMM_WORLD); 
	}
    printf("%d: Sent %lf and got %lf\n", rank, msgsent, msgrcvd);

    ierr = MPI_Finalize();
    return 0;
}
```

## Something new: Sendrecv
- This sort of logic works, but is quite complicated for such a simple pattern
- Other MPI tools let us do this more easily.
- <a href="https://www.open-mpi.org/doc/v3.1/man3/MPI_Sendrecv.3.php">MPI_Sendrecv</a>: A blocking (send and receive) built in together, as opposed to a blocking send followed by a blocking receive.
- From the programmer's point-of-view they happen simultaneously
- Automatically pairs the sends/recvs!
- This is typical of MPI: With the very basics you can do almost anything, even if you have to jump through some hoops - but there are often more advanced routines which can help do things more easily and faster.
- MPI_Sendrecv(sendbuf, sendcount, sendtype, destination, sendtag, recvbuf, recvcount, recvtype, source, recvtag, communicator, status)
- destination and source do not have to be same; nor do type, size, or tag.  

```
#include <stdio.h>
#include <mpi.h>

int main(int argc, char **argv) {
    int rank, size, ierr;
    int left, right;
    int tag = 1;
    double msgsent, msgrcvd;
    MPI_Status rstatus;

    ierr = MPI_Init(&argc, &argv);
    ierr = MPI_Comm_size(MPI_COMM_WORLD, &size);
    ierr = MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    left = rank-1;
    if (left < 0) left = size-1;
    right = rank+1;
    if (right == size) right = 0;

    msgsent = rank*rank;
    msgrcvd = -999.;

    ierr=MPI_Sendrecv(&msgsent, 1, MPI_DOUBLE, right, tag, &msgrcvd, 1,
                      MPI_DOUBLE, left, tag, MPI_COMM_WORLD, &rstatus );
	
    printf("%d: Sent %lf and got %lf\n", rank, msgsent, msgrcvd);

    ierr = MPI_Finalize();
    return 0;
}
```

**Following is a fortran program for sendrecieve.**

```
program fifthmessage
use mpi
implicit none

    integer :: ierr, rank, comsize
    integer :: left, right
    integer :: tag
    integer :: status(MPI_STATUS_SIZE)
    double precision :: msgsent, msgrcvd
    
    call MPI_Init(ierr)
    call MPI_Comm_size(MPI_COMM_WORLD, comsize, ierr)
    call MPI_Comm_rank(MPI_COMM_WORLD, rank, ierr)

    left = rank-1
    if (left <0) left = comsize-1
    right= rank+1
    if (right >=comsize) right = 0

    msgsent= rank*rank
    msgrcvd= -999.
    tag=1
    
    call MPI_Sendrecv(msgsent, 1, MPI_DOUBLE_PRECISION, right, tag, &
                      msgrcvd, 1, MPI_DOUBLE_PRECISION, left,  tag, &
                      MPI_COMM_WORLD, status, ierr)
 
    print *, rank, 'Sent ', msgsent, 'and recvd ', msgrcvd
    call MPI_Finalize(ierr)
end program fifthmessage
```

## Sendrecv = Send + Recv

**C syntax**

![send recieve](../fig/sendrecv_args.png)

**FORTRAN syntax**
![send reciev](../fig/sendrecv_args2.png)

Why are there two different tags/types/counts?

