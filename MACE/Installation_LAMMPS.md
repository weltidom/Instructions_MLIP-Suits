# MACE-LAMMPS installation
Instructions for installing LAMMPS with enabled MACE potentials on JUSTUS2 cluster. Based on [MACE documentation](https://mace-docs.readthedocs.io/en/latest/guide/lammps.html).

## CPU version
Load modules
```bash
ml purge
ml compiler/gnu/9.3
ml mpi/impi
ml numlib/mkl
```

Download LibTorch
```bash
wget https://download.pytorch.org/libtorch/cpu/libtorch-shared-with-deps-1.13.0%2Bcpu.zip
unzip libtorch-shared-with-deps-1.13.0+cpu.zip
rm libtorch-shared-with-deps-1.13.0+cpu.zip
```

Install
```bash
git clone --branch mace --depth=1 https://github.com/ACEsuit/lammps
cd lammps; mkdir build; cd build
cmake -DCMAKE_INSTALL_PREFIX=$(pwd) \
      -DBUILD_MPI=ON \
      -DBUILD_OMP=ON \
      -DPKG_OPENMP=ON \
      -DPKG_ML-MACE=ON \
      -DCMAKE_PREFIX_PATH=$(pwd)/../../libtorch \
      ../cmake
make -j 4
make install
```

## CUDA version
Currently no supported combination of PyTorch/CUDA/cuDNN versions available on JUSTUS2 cluster.