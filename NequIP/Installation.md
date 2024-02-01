# NequIP installation on JUSTUS2 cluster
Installation of CUDA-dependent (NequIP)[https://github.com/mir-group/nequip] utilizing Conda.

```bash
conda create --name nequip_cuda
conda activate nequip_cuda
```

```bash
conda install pytorch==1.11.0 -c pytorch
conda install conda-forge::wandb
conda install conda-forge::nequip
```