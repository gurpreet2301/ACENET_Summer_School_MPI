---
title: "MPI Hello World"
teaching: 15
exercises: 10
questions:
- What does a basic MPI program look like?
- How do I compile an MPI program?
- How can I find out about more MPI functions?
objectives:
- Compile and run either C or Fortran "Hello MPI World"
- Recognize MPI header files and function calls
- Know how to find more information about more MPI functions.
keypoints:
- C requires `#include <mpi.h>` and is compiled with `mpicc`
- Fortran requires `use mpi` and is compiled with `mpif90`
- commands `mpicc, mpif90` and others are *wrappers* around compilers
---

Lets create our first MPI program. If following along with C use the file name `hello-world.c` or if following along replace the file name extension with `.f90`. You will also need to add the Fortran code rather than C code to your files.

~~~
$ nano hello-world.c
~~~
{: .language-bash}

**or**

~~~
$ nano hello-world.f90
~~~
{: .language-bash}


and add the following C/Fortran code.

~~~
#include <stdio.h>
#include <mpi.h>

int main(int argc, char **argv) {
  
  int rank, size, ierr;
  
  ierr=MPI_Init(&argc,&argv);
  ierr=MPI_Comm_size(MPI_COMM_WORLD, &size);
  ierr=MPI_Comm_rank(MPI_COMM_WORLD, &rank);
  
  printf("Hello, World, from task %d of %d\n",rank, size);
  
  ierr=MPI_Finalize();
  return 0;
}
~~~
{: .language-c}

~~~
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
~~~
{: .language-fortran}

Notice the MPI function calls and constants (beginning with `MPI_`) in the following C and Fortran sample codes. Let save and exit nano.

## Compiling

MPI is a ***Library*** for ***Message-Passing***. MPI is not built into the compiler. It is a library that can be linked in while building the final executable as you would link in any other library. It is available for the C,C++ and Fortran languages. There are even ways to use MPI with python and R (see [mpi4py](https://mpi4py.readthedocs.io/en/stable/) and [Rmpi](https://cran.r-project.org/web/packages/Rmpi/index.html) ).

However, there are compiler wrappers (`mpicc`, `mpic++`, `mpif90`, `mpif77`) which automatically include the needed options to compile and link MPI programs.

Once the MPI program is compiled they can be launched using the `mpirun` command.

As previously mentioned, the easiest way to compile an MPI program is to use one of the compiler wrappers. For our C code we use the `mpicc` compiler wrapper. For our Fortran code we use the `mpif90` compiler wrapper. These compiler wrappers add various `-I`, `-L`, and `-l` options as appropriate for MPI. Including the `--showme` option shows which underlying compiler and options are being used by the compiler wrapper.

~~~
$ mpicc --showme hello-world.c -o hello-world
~~~
{: .language-bash}
~~~
gcc hello-world.c -o hello-world
  -I/cvmfs/soft.computecanada.ca/easybuild/software/2017/avx2/Compiler/gcc7.3/openmpi/3.1.4/include
  -L/cvmfs/soft.computecanada.ca/easybuild/software/2017/avx2/Compiler/gcc7.3/openmpi/3.1.4/lib
  -lmpi
~~~
{: .output}

So `mpicc` runs `gcc hello.world.c -o hello-world` with a number of options (`-I`, `-L`, `-l`) to make sure the right libraries and headers are available for compiling MPI programs. Lets take a look at this again for `mpif90` compiler.


Finally lets do the actual compilation with
~~~
$ mpicc hello-world.c -o hello-world
~~~
{: .language-bash}
**or**
~~~
$ mpif90 hello-world.f90 -o hello-world
~~~
{: .language-bash}
And then run it with two processes.
~~~
$ mpirun -np 2 hello-world
~~~
{: .language-bash}
~~~
Hello, World, from task 0 of 2
Hello, World, from task 1 of 2
~~~
{: .output}

## MPI Library reference

So far we have seen the following MPI library functions:

* [`MPI_Init()`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Init.3.php)
* [`MPI_Comm_size()`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Comm_size.3.php)
* [`MPI_Comm_rank()`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Comm_rank.3.php)
* [`MPI_Finalize()`](https://www.open-mpi.org/doc/v3.1/man3/MPI_Finalize.3.php)

Each of the above functions listed above are a link to a page about them on [OpenMPI docs](https://www.open-mpi.org/doc/v3.1/). On these pages you can get information about what functions do and what the expected parameters for them are along with any errors they might return.

If you go to the main page of [OpenMPI docs](https://www.open-mpi.org/doc/v3.1/) you can see that there are many MPI library functions (>200). However, there are not nearly so many concepts. In this workshop we will cover only a handful of these functions but with these few functions we can accomplish some pretty powerful things.

> ## Varying the number of processors
> Try different numbers of processes with the `-np` option.
> 
> > ## Solution
> > ~~~
> > $ mpirun -np 2 hello-world
> > ~~~
> > {: .language-bash}
> > ~~~
> > Hello, World, from task 1 of 2
> > Hello, World, from task 0 of 2
> > ~~~
> > {: .output}
> > ~~~
> > $ mpirun -np 4 hello-world
> > ~~~
> > {: .language-bash}
> > ~~~
> > Hello, World, from task 0 of 4
> > Hello, World, from task 2 of 4
> > Hello, World, from task 3 of 4
> > Hello, World, from task 1 of 4
> > ~~~
> > {: .output}
> > ~~~
> > $ mpirun -np 8 hello-world
> > ~~~
> > {: .language-bash}
> > ~~~
> > --------------------------------------------------------------------------
> > There are not enough slots available in the system to satisfy the 8 slots
> > that were requested by the application:
> >   hello-world
> > 
> > Either request fewer slots for your application, or make more slots available
> > for use.
> > --------------------------------------------------------------------------
> > ~~~
> > {: .output}
> > As might be expected, each core generates its own line output. If however, you try to run with more processes than cores available you will get the above message and the program will not run.
> {: .solution}
{: .challenge}

> ## Running multiple times
> Try running the program multiple times with the same number of processes
> 
> > ## Solution
> > ~~~
> > $ mpirun -np 4 hello-world
> > ~~~
> > {: .language-bash}
> > ~~~
> > Hello, World, from task 0 of 4
> > Hello, World, from task 1 of 4
> > Hello, World, from task 2 of 4
> > Hello, World, from task 3 of 4
> > ~~~
> > {: .output}
> > ~~~
> > $ mpirun -np 4 hello-world
> > ~~~
> > {: .language-bash}
> > ~~~
> > Hello, World, from task 1 of 4
> > Hello, World, from task 2 of 4
> > Hello, World, from task 3 of 4
> > Hello, World, from task 0 of 4
> > ~~~
> > {: .output}
> > Notice that the order the messages are printed out does not remain the same from run to run. This is because different processes are not all executing the lines of a program in lockstep with each other. You should be careful not to write your program to depend on the order in which processes execute statements relative to each other as it is not dependable.
> {: .solution}
{: .challenge}

