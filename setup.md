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

``` $ git clone https://github.com/acenet-arc/mpi-tutorial.git ```

Then, start an interactive shell with four CPUs on a compute node.
The `--x11` flag is required to allow some visualizations to work
later in the workshop:

``` $ salloc --ntasks=4 --time=2:00:00 --x11```

Finally, ensure you have suitable environment modules loaded so you'll
be able to run `mpirun, mpicc` or `mpif90`, and use `pgplot`:

```$ module purge; module load arch/avx2 StdEnv nixpkgs/16.09  intel/2016.4 pgplot```
