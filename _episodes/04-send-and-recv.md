---
title: "Send and receive"
teaching: 15
exercises: 10
questions:
- How are messages passed?
objectives:
- Write code to pass a message from one process to another
keypoints:
- Functions: MPI\_Ssend, MPI\_Recv
- Types: MPI\_CHAR, MPI\_Status
---

Our "Hello World" program doesn't actually pass any messages.  
Let's fix this, but first we need to know a little more about messages.

## Messages
- Messages have a **sender** and a **receiver**
- When you are sending a message, don't need to specify sender (it's the current processor),
- A sent message has to be actively received by the receiving process

![messages](../fig/messages.png)

- MPI messages are a string of length __count__ all of some fixed MPI __type__
- MPI types exist for characters, integers, floating point numbers, etc.
- An arbitrary integer __tag__ is also included - helps keep things straight if lots of messages are sent, but usually unnecessary

## Send and Receive

**C**:

```
MPI_Status status;
ierr = MPI_Ssend(sendptr, count, MPI_TYPE, destination,tag, Communicator);
ierr = MPI_Recv(rcvptr, count, MPI_TYPE, source, tag,Communicator, status);
```

**Fortran**:

```
integer status(MPI_STATUS_SIZE)
call MPI_SSEND(sendarr, count, MPI_TYPE, destination,tag, Communicator, ierr)
call MPI_RECV(rcvarr, count, MPI_TYPE, source, tag,Communicator, status, ierr)
```

- Sends `count` of some `Type` to process of rank `destination` in `Communicator`
- Tag must be non-negative, must be the same at send and at receive
- Return argument `status` contains information about the message. 

## firstmessage.c
- `cp hello.c firstmessage.c`
- Edit with me
- `mpicc -o firstmessage firstmessage.c`
- `mpirun -np 2 ./firstmessage`
- Note: C - MPI_CHAR

```
#include <stdio.h>
#include <mpi.h>

int main(int argc, char **argv)
 {
    int rank, size,ierr;
    int sendto, recvfrom;       /* tasks to send to and receive from */
    int ourtag=1;		/* shared tag to label msgs */
    char sendmessage[]="Hello"; /* text to send */
    char getmessage[6];         /* place to receive */
    MPI_Status rstatus;         /* MPI_Recv status info */

    ierr = MPI_Init(&argc,&argv);
    ierr = MPI_Comm_size(MPI_COMM_WORLD, &size);
    ierr = MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    if (rank == 0) {
        sendto = 1;
        ierr = MPI_Ssend(sendmessage, 6, MPI_CHAR, sendto, ourtag, MPI_COMM_WORLD);
        printf("%d: Sent message <%s>\n",rank, sendmessage);
    }
    else if (rank == 1) {
        recvfrom = 0;
        ierr = MPI_Recv(getmessage, 6, MPI_CHAR, recvfrom, ourtag, MPI_COMM_WORLD, &rstatus);
        printf("%d: Got message  <%s>\n",rank, getmessage);
    }
  
    MPI_Finalize();
    return 0;
}
```

## firstmessage.f90
- `cp hello.f90 firstmessage.f90`
- Edit with me
- `mpif90 -o firstmessage firstmessage.f90`
- `mpirun -np 2 ./firstmessage`
- FORTRAN -MPI_CHARACTER

```
program firstmessage
    use mpi
    implicit none
    integer :: rank, comsize, ierr
    integer :: sendto, recvfrom  ! task to send,recv from 
    integer :: ourtag=1          ! shared tag to label msgs
    character(5) :: sendmessage  ! text to send
    character(5) :: getmessage   ! text to recieve 
    integer, dimension(MPI_STATUS_SIZE) :: rstatus
        
    call MPI_Init(ierr)
    call MPI_Comm_size(MPI_COMM_WORLD, comsize, ierr)
    call MPI_Comm_rank(MPI_COMM_WORLD, rank, ierr)
    
    if(rank == 0) then
        sendmessage='hello'
        sendto = 1;
        call MPI_Ssend(sendmessage, 5, MPI_CHARACTER, sendto, ourtag,MPI_COMM_WORLD,ierr) 
        print *,rank, 'sent message <',sendmessage,'>'
    else if(rank == 1) then
        recvfrom = 0;
        call MPI_Recv(getmessage, 5, MPI_CHARACTER, recvfrom, ourtag, MPI_COMM_WORLD, rstatus,ierr)
        print *,rank, 'got message  <',getmessage,'>'
    endif
    
    call MPI_Finalize(ierr)
end program firstmessage
```

## Wildcard sources and destinations

* Not used here, but worth knowing:
* MPI_PROC_NULL basically ignores the relevant operation; can lead to cleaner code.
* MPI_ANY_SOURCE is a wildcard; matches any source when receiving.

