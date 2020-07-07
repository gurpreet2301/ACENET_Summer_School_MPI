---
layout: page
title: Setup
permalink: /setup/
---
The exercises in this lesson will be carried out on the virtual cluster we have
been using the rest of the course, `nova.acenetsummerschool.ca`.  If you have a
Compute Canada account, you should be able to follow along equally well on any
of the Compute Canada general-purpose clusters 
<a href="https://docs.computecanada.ca/wiki/Compute_Canada_Documentation">(Béluga, Cedar, Graham)</a>.

You should obtain the example programs and exercise templates by logging in to
the cluster and then cloning the following repository from GitHub:

```
$ git clone https://github.com/acenet-arc/mpi-tutorial.git
$ cd mpi-tutorial
```

Ensure you have suitable environment modules loaded so you'll
be able to run `mpirun, mpicc` or `mpif90`, and use `pgplot`.
If you run `which mpicc` it should respond with the path to a file.
If it responds with `/usr/bin/which: no mpicc in ...` a great
long list of directories, then load the following environment 
modules and try again:

```
$ module purge; module load gcc/7 openmpi/3 pgplot
```

Later on the instructor may ask you to start an interactive shell
using the job scheduler, like so:

```
$ salloc --ntasks=4 --time=2:00:00 --x11
```

The `--x11` flag may be required to allow some visualizations to work
in the last exercise of the workshop. 
