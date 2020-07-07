---
title: "Hello MPI World"
teaching: 10
exercises: 5
questions:
- How do I compile an MPI program?
objectives:
- Compile and run either C or Fortran "Hello MPI World"
- Recognize MPI header files and function calls 
keypoints:
- C requires `#include <mpi.h>` and is compiled with `mpicc`
- Fortran requires `use mpi` and is compiled with `mpif90`
- `mpicc, mpif90` and others are *wrappers* around compilers
---

## MPI is a *Library* for Message-Passing

- Not built in to compiler
- Function calls that can be made from any compiler, many languages
- Just link to it
- Compiler wrappers `mpicc, mpic++, mpif90, mpif77` 
- Launch programs with `mpirun` or `mpiexec`

Notice the function calls and constants in the following sample code:

**C**   

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


**Fortran**  

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

## What mpicc/ mpif90 do

- Just wrappers for other C or Fortran compilers that add various -I, -L clauses as appropriate
- `--showme` (an Open MPI option) shows which options are being used, e.g.:
```
$ mpicc --showme hello-world.c -o hello-world

gcc -I/usr/local/openmpi.gcc-1.2.9/include
  -pthread hello-world.c -o hello-world
  -L/usr/local/openmpi.gcc-1.2.9/lib
  -lmpi -lopen-rte -lopen-pal -ldl
   -Wl,--export-dynamic -lnsl -lutil -lm -ldl
```
- Just runs `gcc hello.world.c -o hello-world` with a number of options (-I, -L, -l) to make sure right libraries, headers are available.


## MPI is a Library for *Message-Passing*
- Communication/coordination between tasks done by sending and receiving messages.
- Each message involves a function call from each of the instances of the program.
 
![message passing](../fig/message_passing.png)

 - Three basic sets of functionality: 
   - Pairwise communications via messages
   - Collective operations via messages
   - Efficient routines for getting data from memory into messages and vice versa

## Messages
- Messages have a **sender** and a **receiver**
- When you are sending a message, don't need to specify sender (it's the current processor),
- A sent message has to be actively received by the receiving process

![messages](../fig/messages.png)

- MPI messages are a string of length __count__ all of some fixed MPI __type__
- MPI types exist for characters, integers, floating point numbers, etc.
- An arbitrary integer __tag__ is also included - helps keep things straight if lots of messages are sent. 
 
## Size of MPI Library   
- Many, many functions (>200)
- Not nearly so many concepts
- We'll get started with just 10-12, use more as needed.

```
MPI_Init()  
MPI_Comm_size()  
MPI_Comm_rank()  
MPI_Ssend()  
MPI_Recv()  
MPI_Finalize()  
```

> ## Hello World
> Pick a language and type in the sample code from above.
> You'll learn more if you don't cut and paste!
> Compile it with `mpicc` or `mpif90` then run the result with `mpirun`. 
> Try different numbers of processes with the `-np` option.
> Remember what we learned about running in a job context last episode.
>
> > ## Solution
> > ```
> > $ cd mpi-tutorial/mpi-intro
> > 
> > $ mpicc hello-world.c -o hello-world
> > or 
> > $ mpif90 hello-world.f90 -o hello-world
> > 
> > $ mpirun hello-world
> > $ mpirun -np 2 hello-world
> > $ mpirun -np 8 hello-world
> > ```
> {: .solution}
{: .challenge}
