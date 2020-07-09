---
Title: "Where to go next"
teaching: 5
exercises: 0
questions:
- Where can I learn more about MPI?
---

## References

MPI standards can be found at the <a href="https://www.mpi-forum.org/docs/">MPI Forum</a>.
In particular, the MPI 3.1 standard is laid out in 
<a href="https://www.mpi-forum.org/docs/mpi-3.1/mpi31-report.pdf">this PDF</a>.

The default MPI implementation on Compute Canada clusters is Open MPI.
Manual pages can be found at
<a href="https://www.open-mpi.org/doc/">www.open-mpi.org</a>

The default version of Open MPI may differ from one cluster to another,
but you can find what version and change it with `module` commands.
See <a href="https://docs.computecanada.ca/wiki/Utiliser_des_modules/en">Using modules</a>
at the Compute Canada documentation wiki.

There are online tutorials for learning MPI, including these from:
* <a href="https://mpitutorial.com/tutorials/">mpitutorial.com</a> 
* <a href="https://computing.llnl.gov/tutorials/mpi/">Lawrence Livermore National Labs</a>

And finally, an anti-reference.  Or something.  In late 2018/early 2019
Jonathan Dursi, who created the original version of this workshop, wrote an
essay post entitled: 

* <a href="https://www.dursi.ca/post/hpc-is-dying-and-mpi-is-killing-it.html">HPC is dying, and MPI is killing it</a>

Its intended audience is people already working in high-performance computing,
but has some very cogent things to say about what's wrong with MPI.  Too long;
didn't read?  MPI is over 25 years old, and showing its age.  Computing is a 
<a href="https://xkcd.com/1428/">fast-moving field</a>.  If you are lucky
enough to be starting a brand-new project that requires massive parallelism,
look carefully at newer technologies than MPI.
