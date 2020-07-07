---
title: "mpirun"
teaching: 10
exercises: 5
questions:
- "How do I run an MPI program?"
objectives:
- "Use `mpirun` and `srun`"
- "Request resources for MPI from Slurm"
keypoints:
- "`mpirun` starts *any* program"
- "`srun` does same as `mpirun` inside a Slurm job"
- "`mpirun` and `srun` get task count from Slurm context"
---

## MPI is a standard

**MPI** stands for **Message Passing Interface**,
a "portable message-passing standard designed by a group of researchers from
academia and industry to function on a wide variety of parallel computing
architectures. The standard defines the syntax and semantics of a core of
library routines useful to a wide range of users writing portable
message-passing programs in C, C++, and Fortran. There are several well-tested
and efficient implementations of MPI, many of which are open-source or in the
public domain." 
[<a href="https://en.wikipedia.org/wiki/Message_Passing_Interface">Wikipedia</a>]
- Open MPI implementation available by default on Compute Canada clusters
- MPICH and Intel MPI also available, use `module spider ...` if interested

## mpirun runs *any* program
- `mpirun` starts *N* copies of a program
- For multinode run, has a list of nodes, ssh's (or equivalent) to each node and launches the program

```
$ ls
$ mpirun -np 2 ls
$ hostname  
$ mpirun -np 4 hostname  
```

![What mpirun does](../fig/mpirun.png)

## What else does mpirun do?
- Assigns an MPI *rank* to each process
- Sets variables that programs written *for* MPI recognize so that they know which process they are

## Number of Processes
- Number of processes to use is almost always equal to the number of processors
- You can choose different numbers if you want, but think about why you want to!

## Interaction with the job scheduler
`mpirun` behaves a little differently in a job context than outside it.
A cluster scheduler allows you to specify how many processes you want,
and optionally other information like how many nodes to put those
processes on.  With <a href="https://slurm.schedmd.com/sbatch.html">Slurm,</a>
the most relevant options are
* `--ntasks`
* `--nodes`
* `--ntasks-per-node`

> ## Try this
> What happens if you run the following directly on a login node?
> 
> ```
> $ mpirun -np 2 hostname
> $ mpirun hostname
> ```
> How do you think `mpirun` decides on the number of processes if you
> don't provide `-np`?
> 
> Now get an interactive shell from the job scheduler and try again:
> ```
> $ salloc --ntasks=4 --time=2:0:0 
> $ mpirun hostname
> $ mpirun -np 2 hostname
> $ mpirun -np 6 hostname
> ```
{: .challenge}

