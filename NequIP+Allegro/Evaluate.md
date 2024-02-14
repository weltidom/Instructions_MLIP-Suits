# Evaluate trained potential on test data

[Integrated command for evaluation](https://github.com/mir-group/nequip?tab=readme-ov-file#evaluating-trained-models-and-their-error) produces .xyz output not containing original data and thus makes analysis cumbersome.

## Evaluate training set utilizing NequIP ASE calculator
Using the integrated ASE calculator, the model's inferences results can be added onto the originial .xyz data file.
```python
from ase.io import read, write
import numpy as np

from nequip.ase import NequIPCalculator


splits=np.asplits(0,10)

for split in splits:
    # paths to
    model = f'results/model_{split}.pth' # needs to be a deployed model
    data = f'/home/st/st_us-031400/st_st179390/traj/hea/valid/valid_{split}.xyz'
    output = f'eval/standard/test_{split}.xyz'


    calculator = NequIPCalculator.from_deployed_model(model_path=model)
    atoms_lst = read(data, index=':', format='extxyz')

    for atoms in atoms_lst: 
        atoms.set_calculator(calculator)
        e=atoms.get_total_energy()
        f=atoms.get_forces()
        atoms.calc=None
        atoms.info['NequIP_energy'] = e
        atoms.arrays['NequIP_forces'] = f

    write(output, images=atoms_lst, format='extxyz')
    print(f"Split {split} evaluation finished")
```
*Initialization of NequIP calculator for each configuration is quite inefficient*

## Example sbatch file to submit evaluation job on JUSTUS2

```bash
#!/usr/bin/env bash
#SBATCH -J nequip-eval_ase_0
#SBATCH --ntasks=12
#SBATCH --gres=gpu:1
#SBATCH --time 0-05:00:00
#SBATCH --output=eval-%x.%j.out

source activate nequip_cuda
module load compiler/intel
module load mpi/impi

mpirun -n 1 python /home/st/st_us-031400/st_st179390/nequip/ta-v-cr-w/evalNequipAse.py
```