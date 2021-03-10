# Fedora

- Install source code manager:

```bash
sudo dnf install git-all
```

- Install build tools

```bash
sudo dnf install cmake python3
```

- Install compilers ::

```bash
sudo dnf install gcc-gfortran
```

- Install libraries (MPI, HDF5, FFTW3, LAPACK) ::

```bash
sudo dnf install hdf5-openmpi-devel openmpi-devel fftw-devel openblas-devel
```

- Install software for documentation::

``bash
sudo dnf install doxygen texlive-latex
```

Don't forget to load the mpi module ::

```
module load openmpi-x86_64
```

```{warning}
mpich on Fedora is not supported.
```
