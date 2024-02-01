# Running ASE MD with MACE potential

The device (CPU or CUDA) used for training is also needed for ASE MD runs (as opposed to LAMMPS).

## Python script example
Langevin thermostat MD run.
```python
from ase import units
from ase.md.langevin import Langevin
from ase.io import read, write
import numpy as np
import time

from mace.calculators import MACECalculator

calculator = MACECalculator(model_paths='/home/st/st_us-031400/st_st179390/mace/ta-v-cr-w/inference_ase/mace_split_0.model', device='cuda')
init_conf = read('4comp_0k_432atoms.xyz', '0') # first frame/configuration from input data as starting point
init_conf.set_calculator(calculator)

dyn = Langevin(init_conf, 0.5*units.fs, temperature_K=500, friction=5e-3)
def write_frame():
        dyn.atoms.write('inference_gpu_ase.xyz', append=True)
dyn.attach(write_frame, interval=50)
dyn.run(10000)
print("MD finished!")
```

## Batch script example
```bash
#!/usr/bin/env bash
#SBATCH --nodes=1
#SBATCH --gres=gpu:1
#SBATCH --ntasks=1
#SBATCH --exclusive
#SBATCH --time=0-05:00:00
#SBATCH --job-name="ase-mace-cuda"
#SBATCH --output=%j.out
#SBATCH --error=%j.err

source activate mace_cuda
module load compiler/intel
module load mpi/impi


mpirun -n $SLURM_NTASKS python /home/st/st_us-031400/st_st179390/mace/ta-v-cr-w/inference_ase/ase_run.py
```