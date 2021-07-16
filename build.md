# Build the library

The prerequisites can be installed with a couple of commands on linux systems. 
- MPI, Most of our testing is done with openmpi but mpich should also work.
- build tools: CMake, python3
- compilers: fortran
- libraries: HDF5, FFTW3, LAPACK
- documentation (optional): Doxygen
- visualisation : gnuplot, paraview, VisIt

## Install dependencies on Ubuntu

```bash
sudo apt-get install cmake git
sudo apt-get install libopenmpi-dev openmpi-bin libhdf5-openmpi-dev
sudo apt-get install  libfftw3-dev liblapack-dev libopenblas-dev
sudo apt-get install doxygen texlive graphviz
```

## Install dependencies on Fedora

```bash
sudo dnf install git-all
sudo dnf install cmake python3
sudo dnf install gcc-gfortran
sudo dnf install hdf5-openmpi-devel openmpi-devel fftw-devel openblas-devel
sudo dnf install doxygen texlive-latex graphviz
```

Don't forget to load the mpi module

```
module load mpi/openmpi-x86_64
```

```{warning}
mpich on Fedora is not supported.
```
## Install dependencies on MACOSX

The easiest way is to use [Homebrew](https://brew.sh)

```
brew install git cmake gfortran open-mpi
brew install hdf5-mpi openmpi fftw openblas
brew install doxygen graphviz
```

## Download the sources
```bash
git clone https://github.com/selalib/selalib
cd selalib
```

## Build the library
```bash
mkdir build
cd  build
cmake ..
make 
```
## Check your installation
```bash
ctest --output-on-failure
```

To open an issue, follow this url: <https://github.com/selalib/selalib/issues/new>

## Run a similation

In [simulations directory](https://github.com/selalib/selalib/tree/main/simulations), you will find examples to use the library, every program should have a README file with instructions.

## Use as external library

If selalib is already installed on your server or if you just want to build and run a simulation. Copy files in one of the simulations directory and use the Makefile below:

```makefile
SIM_NAME = bsl_dk_3d1v_polar
NML_FILE = dksim4d_polar_input.nml
SLL_DIR ?= /opt/selalib
FC = h5pfc    # hdf5 wrapper for mpif90
FFLAGS = -w -ffree-line-length-none -fall-intrinsics -O3 -fPIC -march=native -I${SLL_DIR}/include/selalib
FLIBS = -L${SLL_DIR}/lib -lselalib -lfftw3 -ldfftpack

${SIM_NAME}: sll_m_sim_${SIM_NAME}.o ${SIM_NAME}.o
	${FC} ${FFLAGS} -o ${SIM_NAME} $^ -I${SLL_DIR}/include/selalib ${FLIBS}

run: ${SIM_NAME}
	mpirun -np 8 $^ ${NML_FILE}

clean:
	rm -f *.o ${SIM_NAME} *.mod

selalib:
	git clone https://github.com/selalib/selalib.git

sll_build: selalib
	mkdir -p $@
	cd $@
	cmake ../selalib -DCMAKE_INSTALL_PREFIX=${SLL_DIR} \
 	       -DBUILD_TESTING=OFF -DBUILD_SIMULATIONS=OFF
	make install

.SUFFIXES: $(SUFFIXES) .F90

.F90.o:
	$(FC) $(FFLAGS) -c $<

.mod.o:
	$(FC) $(FFLAGS) -c $*.F90
```

Download, build and install the library with:
```bash
make sll_build
```
Run your simulation :
```
make run
```
