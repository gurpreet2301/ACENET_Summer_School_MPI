---
layout: page
title: Setup
permalink: /setup/
---
The exercises in this lesson will be carried out on
a Compute Canada cluster. You must have an account with Compute Canada
already. If you do not, then visit
<a href="https://www.ace-net.ca/wiki/Get_an_Account">Get an Account</a>
and apply for one now. 
**THIS PROCESS TYPICALLY TAKES AT LEAST TWO BUSINESS DAYS.**

A small number of guest accounts will be available the first
day of the workshop in case anyone is unable to obtain an Compute Canada account
in time.

You should obtain the example programs and exercise templates by
cloning the following repository from GitHub into your ACENET account:

``` $ git clone https://github.com/acenet-arc/mpi-tutorial.git ```

Then, start an interactive shell on an Compute Canada computer node:

``` $ salloc --ntasks=4 --time=4:00:00 --account=acenet-wa --reservation=acenet-wr_cpu --x11```

Finally, ensure you have suitable environment modules loaded so you'll
be able to run mpirun, mpicc or mpif90, and use pgplot:

```$ module purge; module load arch/avx2 StdEnv nixpkgs/16.09  intel/2016.4 pgplot```
