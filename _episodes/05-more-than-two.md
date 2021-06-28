---
title: "More than two processes"
teaching: 20
exercises: 15
questions:
- How do we manage communication among many processes?
objectives:
- Write code to pass messages along a chain
- Write code to pass messages around a ring
keypoints:
- A typical MPI process calculates which other processes it will communicate with.
- If there is a closed loop of procs sending to one another, there is a risk of deadlock.
- All sends and receives must be paired, **at time of sending**.
- Types MPI\_DOUBLE (C), MPI\_DOUBLE\_PRECISION (Ftn)
- Constants MPI\_PROC\_NULL, MPI\_ANY\_SOURCE
---

MPI programs routinely work with thousands of processors. In our first message we used `if`s to write an `MPI_Ssend` and an `MPI_Recv` for each process. Applying this idea niavely to large numbers of processes would lead to something like:
~~~
if(rank==0){...}
if(rank==1){...}
if(rank==2){...}
...
~~~
{: .code}

which gets a little tedious somewhere at or before rank=397. More typically programs calculate the ranks they need to communicate with. Let's imagine processors in a line. How could we calculate the left and right neighbour ranks?

![](../fig/processes_in_a_line.svg)

One way to calculate our neighbour's ranks cloud be `left=rank-1`, and `right=rank+1`. Lets think for a moment what would happen if we wanted to send a message to the right. Each process would have an `MPI_Ssend` call to send to their right neighbour and each process would also have an `MPI_Recv` call to receive a message from their left neighbour. But what will happen at the ends? It turns out that there are special constants that can be used as sources and destinations of messages. If `MPI_PROC_NULL` is used the send or receive is skipped.

> ## `MPI_ANY_SOURCE`
> `MPI_ANY_SOURCE` is similar to `MPI_PROC_NULL` but can only be used when receiving a message. It will allow a receive call to accept a message sent from any process.
{: .callout}

Lets try out sending a set of messages to the processes on the right.

~~~
$ cp firstmessage.c secondmessage.c
~~~
{: .language-bash}
**Remember to replace `.c` with `.f90` if working with Fortran code.**


~~~
#include <stdio.h>
#include <mpi.h>

int main(int argc, char **argv) {
  
  int rank, size, ierr;
  int left, right;
  int tag=1;
  double msgsent, msgrcvd;
  MPI_Status rstatus;
  
  ierr=MPI_Init(&argc, &argv);
  ierr=MPI_Comm_size(MPI_COMM_WORLD, &size);
  ierr=MPI_Comm_rank(MPI_COMM_WORLD, &rank);
  
  left=rank-1;
  if(left<0){
    left=MPI_PROC_NULL;
  }
  
  right=rank+1;
  if(right>=size){
    right=MPI_PROC_NULL;
  }
  
  msgsent=rank*rank;
  msgrcvd=-999.;
  int count=1;
  ierr=MPI_Ssend(&msgsent, count, MPI_DOUBLE, right,tag,
                 MPI_COMM_WORLD); 
  
  ierr=MPI_Recv(&msgrcvd, count, MPI_DOUBLE, left, tag, MPI_COMM_WORLD,
                &rstatus);
  
  printf("%d: Sent %lf and got %lf\n", rank, msgsent, msgrcvd);
  
  ierr=MPI_Finalize();
  return 0;
}
~~~
{: .language-c}
**NOTE**  `MPI_DOUBLE` is used in C to map to the C `double` datatype, while `MPI_DOUBLE_PRECISION` is used in Fortran to map to Fortran `double precision` datatype.
~~~
program secondmessage
use mpi
implicit none

  integer :: ierr, rank, comsize
  integer :: left, right
  integer :: tag
  integer :: status(MPI_STATUS_SIZE)
  double precision :: msgsent, msgrcvd

  call MPI_INIT(ierr)
  call MPI_COMM_RANK(MPI_COMM_WORLD,rank,ierr)
  call MPI_COMM_SIZE(MPI_COMM_WORLD,comsize,ierr)

  left = rank-1
  if (left < 0) left = MPI_PROC_NULL 
  right = rank+1
  if (right >= comsize) right = MPI_PROC_NULL 

  msgsent = rank*rank
  msgrcvd = -999.
  tag = 1

  call MPI_Ssend(msgsent, 1, MPI_DOUBLE_PRECISION, right, &
                 tag, MPI_COMM_WORLD, ierr)
            
  call MPI_Recv(msgrcvd, 1, MPI_DOUBLE_PRECISION, left, & 
                tag, MPI_COMM_WORLD, status, ierr)
                
  print *, rank, 'Sent ', msgsent, 'and recvd ', msgrcvd

  call MPI_FINALIZE(ierr)

end program secondmessage
~~~
{: .language-fortran}

Now lets compile our program with either

~~~
$ mpicc -o secondmessage secondmessage.c
~~~
{: .language-bash}
**or**
~~~
$ mpif90 -o secondmessage secondmessage.f90
~~~
{: .language-bash}
and run with
~~~
$ mpirun -np 4 ./secondmessage
~~~
{: .language-bash}
~~~
0: Sent 0.000000 and got -999.000000
1: Sent 1.000000 and got 0.000000
2: Sent 4.000000 and got 1.000000
3: Sent 9.000000 and got 4.000000
~~~
{: .output}
<!--
Lets explore this a little to see the order in which messages are sent and received. Computers are complex and there is a lot going on and we can't assume that the order of print outs necessarily represents the order of send and receives. Lets make some minor modifications to our code to test this out. If we add a sleep and prints between the send and receive we can get some more insight as to what is happening.

~~~
#include <unistd.h> /*new line*/
  . . .
  ierr=MPI_Ssend(&msgsent,count,MPI_DOUBLE,right,tag,
                 MPI_COMM_WORLD); /*existing line*/
  
  printf("%d: after send before sleep\n",rank); /*new line*/
  sleep(1);                                    /*new line*/
  
  ierr=MPI_Recv(&msgrcvd,count,MPI_DOUBLE,left,tag,MPI_COMM_WORLD,
                &rstatus); /*existing line*/
  
  printf("%d: Sent %lf and got %lf\n", rank, msgsent,
         msgrcvd); /*existing line*/
  . . .
~~~
{: .language-c}

~~~
. . .
  call MPI_Ssend(msgsent, 1, MPI_DOUBLE_PRECISION, right, &
                 tag, MPI_COMM_WORLD, ierr) !existing line
  
  print *, rank, 'after send before recv' !newline
  call sleep(1)                           !newline
  
  call MPI_Recv(msgrcvd, 1, MPI_DOUBLE_PRECISION, left, &
                tag, MPI_COMM_WORLD, status, ierr) !existing line
  
  print *, rank, 'Sent ', msgsent, 'and recvd ', msgrcvd !existing line
. . .
~~~
{: .language-fortran}

Lets compile with either
~~~
$ mpicc -o secondmessage secondmessage.c
~~~
{: .language-bash}
**or**
~~~
$ mpif90 -o secondmessage secondmessage.f90
~~~
{: .language-bash}
and run with
~~~
$ mpirun -np 4 ./secondmessage
~~~
{: .language-bash}
~~~
3: after send before recv
2: after send before recv
3: Sent 9.000000 and got 4.000000
1: after send before recv
2: Sent 4.000000 and got 1.000000
0: after send before recv
1: Sent 1.000000 and got 0.000000
0: Sent 0.000000 and got -999.000000

~~~
{: .output}
-->

## Periodic boundaries

In many problems it is helpful to wrap data around at the boundaries of the grid being modelled. For example later in our 1D diffusion exercise, we will use periodic boundary conditions which allows heat flow out of one end of our thin wire to enter the other end making our thin wire a ring. This sort of periodic boundary condition often works well in physical simulations.

Lets try having our messages wrap around from our right most process to our left most process.
![Periodic Send Right](../fig/periodic_sending_right.svg)

> ## Add periodic message passing
> 
> ~~~
> $ cp secondmessage.c thirdmessage.c
> $ nano thirdmessage.c
> ~~~
> {: .language-bash}
> **Remember to replace .c with .f90 if working with Fortran code.**
> And modify the code so that `left` is either `size-1` for C or `comsize-1` for Fortran when at the inner most edge and so that `right` is 0 at the outer most edge.
>
> Then compile and run.
>
> Hint: you may need to press `ctrl+c` after running the program.
> > ## Solution
> > ~~~
> >   . . .
> >   left=rank-1;
> >   if(left<0){
> >     left=size-1;
> >   }
> >   
> >   right=rank+1;
> >   if(right>=size){
> >     right=0;
> >   }
> >   . . .
> > ~~~
> > {: .language-c}
> > ~~~
> >   . . .
> >   left = rank-1
> >   if (left < 0) left = comsize-1
> >   right = rank+1
> >   if (right >= comsize) right = 0
> >   . . .
> > ~~~
> > {: .language-fortran}
> > 
> > Then compile with either
> > ~~~
> > $ mpicc -o thirdmessage thirdmessage.c
> > ~~~
> > {: .language-bash}
> > **or**
> > ~~~
> > $ mpif90 -o thirdmessage thirdmessage.f90
> > ~~~
> > {: .language-bash}
> > 
> > and run with
> > ~~~
> > $ mpirun -np 4 ./thirdmessage
> > ~~~
> > {: .language-bash}
> > 
> > The program hangs and must be killed by pressing `ctrl+c`. This is because the `MPI_Ssend` call blocks until a matching `MPI_Recv` has been initiated.
> > 
> > 
> {: .solution}
{: .challenge}

<!--
## Deadlock
Our program locked up. But why?



* A classic parallel bug
* Occurs when two (or more!) tasks are each waiting for the other to finish
* Whenever you see a closed cycle, you likely have (or risk) deadlock.

![deadlock](../fig/deadlock.png)

# Big MPI Lesson #1

All sends and receives must be paired, **at time of sending**
-->
