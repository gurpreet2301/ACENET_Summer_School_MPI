---
title: "Diffusion simulation"
teaching: 10
exercises: 30
questions:
- What is involved in adapting a whole program to MPI?
objectives:
- Use a finite-difference simulation to explore MPI parallelization
keypoints:
- Finite differences, stencils, guard cells
---

In `mpi-tutorial/diffusion` is some serial code which simulates
the heat flow in a thin wire where we've suddenly heated up the middle
very hot, then removed the heat source and let the wire equilibrate.

The temperature profile at some point in time will look like this:

![1D diffusion - serial result](../fig/id_diffusion.png)

## Build & run serial version

The code uses a simple graphics library called `pgplot` to draw the 
temperature profile.  You'll need to have X11 running in your SSH
client in order to see the plot.  In either case, it will emit a 
whole lot of timesteps and error estimates onto standard output.

```
$ cd mpi-tutorial/diffusion
$ git pull origin master   # get latest code updates
$ module load pgplot
$ make diffusionc    # or make diffusionf 
$ ./diffusionc       # or ./diffusionf
```

## Diffusion Equation
- 1-dimensional partial differential equation
- dT/dt = kappa * d2T/dx2
- ...where T is temperature, t is time, x is position, kappa is a given constant, the "diffusion coefficient"

## Discretizing Derivatives

- Approximate the derivatives with finite differences
- Implicitly or explicitly involves interpolating data and taking derivative of the interpolant
- Determine accuracy by how many data points are used
- The arrangement of data points is called a "stencil", larger stencil --> greater accuracy
- This code uses the cheapest and simplest, the 3-point stencil

![Discretizing Derivatives](../fig/Discretizing_Derivatives.png)

- This mean that at each timestep, the new temperature at some position, T[i], is computed from the
previous temperate at T[i+1], T[i],T[i-1] like so:

![diffusion equation](../fig/diffusion_eq.png)

To solve this in parallel, each process must be responsible
for some segment of space, i.e. a chunk of the wire. 

## Domain Decomposition
- A very common approach to parallelizing on distributed memory computers
- Maintain Locality; need local data mostly, this means only surface data needs to be sent
between processes.
- Pictures of a few problems being solved with domain decomposition:

![domain decomposition](../fig/domain_decomposition.png)

## Implement a diffusion equation in MPI

- Need one neighboring number per neighbor per timestep

![diffusion equation in mpi](../fig/diffusionequationmpi.png)

## Guardcells

- Job 1 needs info on Job 2's first zone, Job 2 needs info on Job 1's last zone
- Pad array with `guardcells' and fill them with the info from the appropriate node by messagepassing or shared memory

![guardcells](../fig/guardcells_1.png)

- Do computation
- guardcell exchange: each cell has to do 2 sendrecvs
	- its rightmost cell with neighbor's leftmost
	- its leftmost cell with neighbor's rightmost
	- Need a convention to avoid deadlock
	- Everyone do right-filling first, then left-filling (say)
	- What happens at absolute left and absolute right ends of everything?  Pick one:
	- Periodic boundary conditions, as if the wire were a loop
	- Fixed-temperature boundary conditions, temperature in first, last zones some fixed value

## Hands-on: MPI diffusion

- cp diffusionf.f90 diffusionfmpi.f90 or
- cp diffusionc.c diffusionc-mpi.c or
- Make an MPI version of diffusion equation
- (Build: make diffusionf-mpi or make diffusionc-mpi)
- Test on 1..8 procs
- add standard MPI calls: init, finalize, comm_size, comm_rank
- Figure out how many points each proc is responsible for (~totpoints/size)
- Figure out neighbors
- Start at 1, end at totpoints/size
- At end of each time step, exchange guardcells; use sendrecv
- Get total error

**MPI routines we know so far - C**

```
MPI_Status status;

ierr = MPI_Init(&argc,&argv);	
ierr = MPI_Comm_size(communicator, &size);
ierr = MPI_Comm_rank(communicator, &rank);
ierr = MPI_Send(sendptr, count, MPI_TYPE, destination, tag, communicator);
ierr = MPI_Recv(recvptr, count, MPI_TYPE, source,      tag, communicator, &status);

ierr = MPI_Sendrecv(sendptr, count, MPI_TYPE, destination, tag,
                    recvptr, count, MPI_TYPE, source,      tag, communicator, &status);

ierr = MPI_Allreduce(&mydata, &globaldata, count, MPI_TYPE, MPI_OP, Communicator);

Communicator -> MPI_COMM_WORLD
MPI_Type -> MPI_FLOAT, MPI_DOUBLE, MPI_INT, MPI_CHAR...
MPI_OP -> MPI_SUM, MPI_MIN, MPI_MAX,...

```

**MPI routines we know so far - Fortran**

``` 
integer status(MPI_STATUS_SIZE)

call MPI_INIT(ierr)
call MPI_COMM_SIZE(Communicator, size, ierr)
call MPI_COMM_RANK(Communicator, rank, ierr)
call MPI_SSEND(sendarr, count, MPI_TYPE, destination, tag, Communicator, ierr)
call MPI_RECV(recvarr,  count, MPI_TYPE, destination, tag, Communicator, status, ierr)

call MPI_SENDRECV(sendptr, count, MPI_TYPE, destination, tag, &
                  recvptr, count, MPI_TYPE, source,      tag, Communicator, status, ierr)
	
call MPI_ALLREDUCE(mydata, globaldata, count, MPI_TYPE, MPI_OP, Communicator, ierr)
	
Communicator -> MPI_COMM_WORLD
MPI_Type -> MPI_REAL, MPI_DOUBLE_PRECISION, MPI_INTEGER, MPI_CHARACTER,...
MPI_OP -> MPI_SUM, MPI_MIN, MPI_MAX,...

```
