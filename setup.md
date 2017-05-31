---
layout: page
title: Setup
permalink: /setup/
---
The exercises in this lesson will be carried out on
an ACENET cluster. You must have an account with ACENET
already. If you do not, then visit
<a href="https://www.ace-net.ca/wiki/Get_an_Account">Get an Account</a>
and apply for one now. 
**THIS PROCESS TYPICALLY TAKES AT LEAST TWO BUSINESS DAYS.**

A small number of guest accounts will be available the first
day of the workshop in case anyone is unable to obtain an ACENET account
in time.

You should obtain the example programs and exercise templates by
cloning the following repository from GitHub into your ACENET account:

``` $ git clone https://github.com/rmdickson/mpi-tutorial.git ```

Then, start an interactive shell on an ACENET computer node:

``` $ qrsh -cwd -pe openmp 4 -l h_rt=10:00:00 bash ```

Finally, ensure you have suitable environment modules loaded so you'll
be able to run mpirun and mpicc or mpif90:

``` $ module purge; module load gcc openmpi/gcc ```
