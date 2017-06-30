---
title: "Send and receive"
teaching: 15
exercises: 10
---

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

- Sends `count` of some `Type` to (`destination`, `Communicator`)
- Labeled with a non-negative tag
- Receive: special status information can use to look up information about the message.

## "Hello world" passes no messages (C)
- Let's fix this
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
    int sendto, recvfrom;       /* task to send,recv from */
    int ourtag=1;		/* shared tag to label msgs */
    char sendmessage[]="Hello"; /* text to send */
    char getmessage[6];         /* text to recieve */
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

## "Hello world" passes no messages (Fortran)
- Let's fix this
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

