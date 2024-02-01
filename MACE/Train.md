# Train MACE model

Requires extended-xyz configuration data as input (readable by [ASE](https://wiki.fysik.dtu.dk/ase/ase/io/io.html)). See [documentation](https://github.com/ACEsuit/mace?tab=readme-ov-file#training) provided by MACE developers for explanation of parameters.

## CUDA
Example sbatch file to submit job on Slurm.
```bash
#!/usr/bin/env bash
#SBATCH -J mace_9_combined
#SBATCH --ntasks=4
#SBATCH --gres=gpu:1
#SBATCH --time 1-15:00:00

source activate mace_cuda
module load compiler/intel
module load mpi/impi


mpirun -np 1 python /home/st/st_us-031400/st_st179390/anaconda3/mace/scripts/run_train.py \
    --name="MACE_9_combined" \
    --train_file="./data/train+valid/combined_9.xyz" \
    --valid_fraction=0.05 \
    --E0s="average" \
    --model="MACE" \
    --r_max=5.0 \
    --energy_key="Energy" \
    --forces_key="forces" \
    --batch_size=5 \
    --max_num_epochs=1000 \
    --start_swa=600 \
    --swa \
    --ema \
    --amsgrad \
    --default_dtype="float32"\
    --device=cuda \
```

## CPU
Example sbatch file to submit job on Slurm.
```bash
#!/usr/bin/env bash
#SBATCH -J mace_0_cpu
#SBATCH -n 1
#SBATCH --cpus-per-task=48
#SBATCH --nodes=1
#SBATCH --exclusive
#SBATCH --time 3-12:00:00

source activate mace_cpu
module load compiler/intel
module load mpi/impi


mpirun -n $SLURM_NTASKS python /home/st/st_us-031400/st_st179390/anaconda3/mace/scripts/run_train.py \
    --name="MACE_0_cpu" \
    --train_file="/home/st/st_us-031400/st_st179390/mace/ta-v-cr-w/data/train/train_0.xyz" \
    --valid_fraction=0.1 \
    --E0s="average" \
    --model="MACE" \
    --r_max=5.0 \
    --energy_key="Energy" \
    --forces_key="forces" \
    --batch_size=15 \
    --max_num_epochs=50 \
    --start_swa=20 \
    --swa \
    --ema \
    --amsgrad \
    --default_dtype="float32" \
    --device=cpu \
```

## Issues
`—dtype=float64 \ `
Leads to the following error: *RuntimeError: Tensors of the same index must be on the same device and the same dtype except `step` tensors that can be CPU and float32 notwithstanding*

Utilizing `srun` leads to *torch.cuda.OutOfMemoryError: CUDA out of memory*. Fixed by using `mpirun` instead.