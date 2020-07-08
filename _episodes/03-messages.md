---
title: "What's a communicator?"
teaching: 10
exercises: 0
questions:
- What makes up an MPI message?
objectives:
- Understand the MPI "boilerplate" code
- Know the parts of an MPI message
keypoints:
- A message has a sender, a receiver, a type, a size, and a tag
- MPI processes have rank from 0 to size-1
- MPI\_COMM\_WORLD is the ordered set of all processes
- Functions MPI\_Init, MPI\_Finalize, MPI\_Comm\_size, MPI\_Comm\_rank
---

## What the code does
- (FORTRAN version; C is similar)

```
program helloworld
use mpi
implicit none
integer :: rank, comsize, ierr

call MPI_Init(ierr)
call MPI_Comm_size(MPI_COMM_WORLD, comsize, ierr)
call MPI_Comm_rank(MPI_COMM_WORLD, rank, ierr)

print *,'Hello World, from task ', rank, 'of', comsize

call MPI_Finalize(ierr)
end program helloworld

```

- `use mpi` : Imports declarations for MPI function calls
- `call MPI_INIT(ierr)`: Initialize MPI. Must come first. Returns error code in ierr, if any.
- `call MPI_FINALIZE(ierr)`: Finish and clean up MPI. Must come last. Returns error code in ierr.
- `call MPI_COMM_RANK, call MPI_COMM_SIZE`: Requires a little more explanation.

## Communicators
- MPI groups processes into "communicators".
- Each communicator has some size, i.e. the number of processes
- Each process has a rank in 0..size-1
- Call MPI\_COMM\_SIZE to get the size of the communicator
- Call MPI\_COMM\_RANK to get this process's rank within the communicator

- Every MPI program starts with a default communicator called MPI\_COMM\_WORLD
- All processes belong to it

We *can* create our own communicators, either breaking up the processes into
subsets, or just re-ordering the ranks within an existing communicator.
We won't need to do this in this workshop.

## Rank and Size much more important in MPI than OpenMP
- In OpenMP, compiler assigns jobs to each thread; don't need to know which one you are.
- MPI: processes determine amongst themselves which piece of puzzle to work on, then communicate with appropriate others.

![Rank and Size much](../fig/message_passing.png)

**Let's look at the same program in C:**

```
#include <stdio.h>
#include <mpi.h>

int main(int argc, char **argv) {
    int rank, size;

    MPI_Init(&argc,&argv);
    MPI_Comm_size(MPI_COMM_WORLD, &size);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    printf("Hello, World, from task %d of %d\n",rank,  size);

    MPI_Finalize();
    return 0;
}
```

- In C `#include <mpi.h>`, in Fortran `use mpi`
- In C, functions **return** ierr
- In Fortran, ierr is an argument
- Difference in parameters to MPI_Init
- Beware!  Standard does not require that every process can write to stdout!
