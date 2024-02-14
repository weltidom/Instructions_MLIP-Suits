# MACE installation

MACE installation on JUSTUS2 cluster utilizing Conda. Requires [GitHub CLI](https://cli.github.com) login.

## GitHub 
### Installation
*I recommend to install in base environment*
```bash
conda install gh --channel conda-forge
```

### Login
```bash
gh auth login
```
HTTPS via web browser: copy link into local browser and enter code

## MACE on CUDA
```bash
conda create --name mace_cuda
conda activate mace_cuda
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia

gh repo clone ACEsuit/mace
pip install ./mace
```

## MACE on CUDA with multi-GPU branch
```bash
gh repo clone ACEsuit/mace -- --branch multi-GPU
```

## MACE on CPU
```bash
conda create --name mace_cpu
conda activate mace_cpu
conda install pytorch torchvision torchaudio cpuonly -c pytorch

gh repo clone ACEsuit/mace 
pip install ./mace
```