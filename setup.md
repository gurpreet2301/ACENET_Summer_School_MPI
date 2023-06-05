---
layout: page
title: Setup
permalink: /setup/
---
The exercises in this lesson will be carried out on the virtual cluster we have
been using the rest of the course, `pcs2023-3.ace-net.training`.  If you have a
Compute Canada account, you should be able to follow along equally well on any
of the Compute Canada general-purpose clusters 
[(Béluga, Cedar, Graham)](https://docs.computecanada.ca/wiki/Compute_Canada_Documentation).

You should obtain the example programs and exercise templates by logging in to
the cluster and then cloning the following repository from GitHub:

~~~
$ git clone https://github.com/acenet-arc/mpi-tutorial.git
$ cd mpi-tutorial
~~~
{: .language-bash}

Ensure you have suitable environment modules loaded so you'll
be able to run `mpirun, mpicc` or `mpif90`, and use `pgplot`.
If you run `which mpicc` it should respond with the path to a file.
If it responds with `/usr/bin/which: no mpicc in ...` a great
long list of directories, then load the following environment 
modules and try again:

~~~
$ module --force purge all
$ module load StdEnv/2020
$ module load arch/avx2
$ module load gcc openmpi pgplot
~~~
{: .language-bash}

Later on the instructor may ask you to start an interactive shell
using the job scheduler, like so:

~~~
$ salloc --ntasks=4 --time=2:00:00 --x11
~~~
{: .language-bash}

The `--x11` flag may be required to allow some visualizations to work
in the last exercise of the workshop.
