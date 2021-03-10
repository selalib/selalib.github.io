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

## Run a similation

In simulations directory, you will find examples to use the library, every program
must have a README file with instructions.

Please open an issue <https://github.com/selalib/selalib/issues/new>
