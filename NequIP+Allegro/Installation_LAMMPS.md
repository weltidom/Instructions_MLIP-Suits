# NequIP-LAMMPS installation

Install LAMMPS from GitHub (requires verification e.g. via [GitHub CLI](https://github.com/cli/cli?tab=readme-ov-file#conda), HTTPS cloning used here). Adopted from [developer's guide](https://github.com/mir-group/pair_nequip).

Download specific LAMMPS release
```bash
git clone -b stable_29Sep2021_update2 --depth 1 https://github.com/lammps/lammps.git
```

Download Pair_NequIP
```bash
git clone https://github.com/mir-group/pair_nequip.git
```

Download LibTorch (pre-cxx11 ABI)
```bash
wget https://download.pytorch.org/libtorch/cu121/libtorch-shared-with-deps-2.2.0%2Bcu121.zip
unzip libtorch-shared-with-deps-2.2.0+cu121.zip
mv libtorch libtorch-gpu
```

Go to folder `pair_nequip` and edit file `patch_lammps.sh` (using C++17 instead of C++14 used in developer's guide for newer LibTorch version):

Replace `set(CMAKE_CXX_STANDARD 14)` with `set(CMAKE_CXX_STANDARD 17)`

Run patch
```bash
./patch_lammps.sh ../lammps
```

Switch working directory
```bash
cd lammps
mkdir build
cd build
```

Request interactive GPU session to proceed with installation
```bash
srun --ntasks-per-node=4 --gres=gpu:1 --time=1:00:00 --pty bash -i
```

Load relevant modules
```bash
ml purge
ml compiler/gnu/12.2
ml mpi/openmpi
ml numlib/mkl
ml lib/cudnn/.9.0.0_cuda-12.3
```

```bash
cmake ../cmake \
	-DCMAKE_PREFIX_PATH=$PWD/../../libtorch-gpu \
	-DCAFFE2_USE_CUDNN=1
```

Build LAMMPS
```Bash
make -j$(nproc)
```

Should result in executable `lmp`.