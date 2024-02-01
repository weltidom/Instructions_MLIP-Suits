# Running MACE-LAMMPS

Based on [MACE documentation](https://mace-docs.readthedocs.io/en/latest/guide/lammps.html).

## Prepare model
```bash
python <mace_repo_dir>/mace/cli/create_lammps_model.py my_mace.model
```

## Example sbatch script for submitting job on JUSTUS2
```bash
#!/usr/bin/env bash
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=24
#SBATCH --exclusive
#SBATCH --time=2-00:00:00
#SBATCH --job-name="lammps-mace-test_2x24"
#SBATCH --output=%j.out
#SBATCH --error=%j.err

module load compiler/gnu/9.3
module load mpi/impi
module load numlib/mkl


mpirun -np 1 /home/st/st_us-031400/st_st179390/codes/lammps-mace/lammps/build/lmp < lammps_input
```
## Example LAMMPS input file using MACE potential
```
#SOF
variable idump equal 0
variable dt equal 0.001
variable Ndt_equilib equal 2000
variable Ndt_stat equal 1
variable tempset equal 500
variable presset equal 0.0
variable seed equal 521
variable thermofac equal 1000.0
variable tempdamp equal ${thermofac}*${dt}
variable presfac equal 5000.0
variable presdamp equal ${presfac}*${dt}
#
#
# ---------- Initialize Simulation ---------------------
clear
units metal
dimension 3
boundary p p p
atom_style atomic
atom_modify map yes
# ---------- Create Atoms ---------------------
box tilt large
read_data lammps.datafile_forspeed.txt
labelmap atom 1 Ta 2 V 3 Cr 4 W
#read_dump dump_atom 33000 x y z
#dump myDump all atom 1000 dump_atom
# ---------- Define Interatomic Potential ---------------------
pair_style mace
pair_coeff * * mace_split_0.model-lammps.pt Ta V Cr W
mass Ta 180.95 #Ta amu/atom = grams/mol
mass V 50.942 #V
mass Cr 51.996 #Cr
mass W 183.84 #W
neighbor 2.0 bin
neigh_modify delay 10 check no
# ---------- Define computations and variables ---------------------
timestep ${dt}
thermo 100
thermo_style custom step vol temp etotal pe press
thermo_modify format 1 %12d
thermo_modify format 2 %22.12f
thermo_modify format 3 %22.12f
thermo_modify format 4 %22.12f
thermo_modify format 5 %22.12f
compute BOBR all pressure thermo_temp virial
compute EN all pe
velocity all create ${tempset} ${seed} dist gaussian mom yes rot yes
fix 122 all nvt temp ${tempset} ${tempset}  ${tempdamp}
#fix 122 all npt temp ${tempset} ${tempset} ${tempdamp} iso ${presset} ${presset} ${presdamp}
fix 3 all ave/time 10 1000 10000 c_BOBR[*] file pressure.txt
#overt tempr
change_box all x scale 1.00 y scale 1.00 z scale 1.00 remap
run ${Ndt_equilib}
```