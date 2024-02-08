# MACE-LAMMPS installation (CPU)
Instructions for installing LAMMPS with enabled MACE potentials on JUSTUS2 cluster. Based on [MACE documentation](https://mace-docs.readthedocs.io/en/latest/guide/lammps.html). Requires verification e.g. via [GitHub CLI](https://github.com/cli/cli?tab=readme-ov-file#conda).

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
``bash
mkdir lammps-mace-gpu
cd lammps-mace-gpu
git clone --branch=mace --depth=1 https://github.com/ACEsuit/lammps
wget https://download.pytorch.org/libtorch/cu121/libtorch-shared-with-deps-2.2.0%2Bcu121.zip
unzip libtorch-shared-with-deps-2.2.0+cu121.zip
mv libtorch libtorch-gpu
```

Going for Pre-cxx11 ABI version of LibTorch, since utilizing cxx11 ABI leads to the following type of errors when installing: `undefined reference to exp@GLIBC_2.29'`.

```bash
srun --ntasks-per-node=4 --gres=gpu:1 --time=1:30:00 --pty bash -i
```

```bash
ml purge
ml compiler/gnu/12.2
ml mpi/openmpi
ml numlib/mkl
ml lib/cudnn/.9.0.0_cuda-12.3
```

```bash
cd lammps
mkdir build-kokkos-cuda
cd build-kokkos-cuda
cmake \
    -D CMAKE_BUILD_TYPE=Release \
    -D CMAKE_INSTALL_PREFIX=$(pwd) \
    -D CMAKE_CXX_STANDARD=17 \
    -D BUILD_MPI=yes \
    -D BUILD_OMP=yes \
    -D BUILD_SHARED_LIBS=yes \
    -D LAMMPS_EXCEPTIONS=yes \
    -D PKG_KOKKOS=yes \
    -D Kokkos_ARCH_AMDAVX=yes \
    -D Kokkos_ARCH_AMPERE100=yes \
    -D Kokkos_ENABLE_CUDA=yes \
    -D Kokkos_ENABLE_OPENMP=yes \
    -D Kokkos_ENABLE_DEBUG=no \
    -D Kokkos_ENABLE_DEBUG_BOUNDS_CHECK=no \
    -D Kokkos_ENABLE_CUDA_UVM=no \
    -D CMAKE_CXX_COMPILER=$(pwd)/../lib/kokkos/bin/nvcc_wrapper \
    -D PKG_ML-MACE=yes \
    -D CMAKE_PREFIX_PATH=$(pwd)/../../libtorch-gpu \
    ../cmake
```

- Utilizing `-D Kokkos_ARCH_AMPERE100=yes` even though Volta architecture on cluster; correct VOLTA70 architecture is still selected during process.
- Ignores cuDNN for whatever reason, maybe rebuild with `-D CAFFE2_USE_CUDNN=1 \` added.

```bash
make -j$(nproc)
```

Executable `lmp` should now be available in your folder.