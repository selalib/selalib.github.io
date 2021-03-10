# Ubuntu or Debian

The prerequisites can be installed with a couple of commands on Ubuntu. 

- Install MPI, Most of our testing is done with openmpi but mpich should also work:

```bash
sudo apt-get install libopenmpi-dev openmpi-bin libhdf5-openmpi-dev
```

- Install build tools :

```bash
sudo apt-get install cmake
```

- Install compilers :

```bash
sudo apt-get install gfortran python3
```

- Install libraries (MPI, HDF5, FFTW3, LAPACK) :

```bash
sudo apt-get install  libfftw3-dev liblapack-dev libopenblas-dev
```

- Install software for documentation:

```bash
sudo apt-get install doxygen texlive-latex3
```

For some outputs you can install also:

```bash
sudo apt-get install gnuplot
```
