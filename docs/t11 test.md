# Tutorial 11 – Running LAMMPS with MTP

## Overview

In the previous tutorials, you learned how MTP models are created using MLIP-2 and MLIP-3.

The final step in the MTP workflow is applying a trained MTP model in a molecular dynamics simulation.

This tutorial introduces how **LAMMPS** uses a trained MTP potential to calculate atomic interactions and perform simulations on DRAC clusters.

By the end of this tutorial, you will understand:

- What LAMMPS is
- How LAMMPS connects with MTP
- The role of the MTP potential file
- The structure of an MTP-LAMMPS simulation
- How to submit an MTP simulation using Slurm

---

# What is LAMMPS?

LAMMPS (**Large-scale Atomic/Molecular Massively Parallel Simulator**) is a molecular dynamics simulation software package used to model materials at the atomic scale.

LAMMPS can simulate systems containing:

- Metals
- Alloys
- Ceramics
- Polymers
- Nanomaterials
- Other molecular systems

During a simulation, LAMMPS calculates atomic motion by evaluating forces between atoms.

---

# Why Use MTP with LAMMPS?

Traditional molecular dynamics simulations require a method for calculating atomic interactions.

These interactions are determined by a **potential**.

Examples include:

- Lennard-Jones potentials
- Embedded Atom Method (EAM)
- Machine-learning potentials such as MTP

For MTP simulations:

```text
Trained MTP Potential
          |
          ↓
LAMMPS
          |
          ↓
Atomic Forces and Energies
          |
          ↓
Molecular Dynamics Simulation
```

The MTP replaces expensive first-principles calculations with a trained machine-learning model.

---

# The MTP Potential File

After training an MTP using MLIP-2 or MLIP-3, a potential file is generated.

Example:

```text
potential.mtp
```

This file contains the learned parameters required for LAMMPS to calculate atomic interactions.

The potential file must be provided to LAMMPS during the simulation.

---

# MTP-LAMMPS Workflow

A typical workflow is:

```text
1. Train MTP using MLIP
          |
          ↓
2. Generate potential.mtp
          |
          ↓
3. Prepare LAMMPS input files
          |
          ↓
4. Submit simulation using Slurm
          |
          ↓
5. Analyze simulation results
```

---

# Required Simulation Files

A typical MTP-LAMMPS simulation contains:

```
simulation/

├── input.lammps
│
├── structure.data
│
├── potential.mtp
│
└── job.slurm
```

Each file has a different purpose:

| File | Purpose |
|------|---------|
| `input.lammps` | LAMMPS simulation instructions |
| `structure.data` | Atomic structure information |
| `potential.mtp` | Trained MTP model |
| `job.slurm` | Slurm submission script |

---

# LAMMPS Input File Structure

A LAMMPS input file defines how the simulation is performed.

A simplified example:

```bash
units metal
atom_style atomic

read_data structure.data

pair_style mlip mlip.ini
pair_coeff *

thermo 100

run 10000
```

This tells LAMMPS to:

1. Use metal units.
2. Read the atomic structure.
3. Use the MTP potential.
4. Run the simulation.

The exact commands depend on the material system and simulation type.

---

# Running LAMMPS Through Slurm

Because simulations can require significant computing resources, they should be submitted through Slurm.

Example Slurm script:

```bash
#!/bin/bash
#SBATCH --job-name=mtp_test
#SBATCH --time=01:00:00
#SBATCH --ntasks=1
#SBATCH --mem=4G

module load lammps

srun lmp -in input.lammps
```

This script:

- Gives the job a name
- Requests one hour of runtime
- Requests one CPU task
- Requests 4 GB of memory
- Loads LAMMPS
- Runs the simulation

Submit the job using:

```bash
sbatch job.slurm
```

---

# Monitoring the Simulation

After submitting a job, check its status:

```bash
squeue -u $USER
```

Example:

```text
JOBID   NAME       ST   TIME
123456  mtp_test   R    0:15
```

Once the simulation finishes, output files can be analyzed.

Completed job information can be viewed using:

```bash
sacct
```

---

# Simulation Output

LAMMPS produces output files containing information about the simulation.

Common outputs include:

- Energies
- Temperatures
- Atomic positions
- Forces
- Trajectories

These results can be analyzed to study material properties such as:

- Stability
- Diffusion
- Mechanical behaviour
- Thermal properties

---

# Choosing Resources for MTP Simulations

The required Slurm resources depend on the simulation.

Important factors include:

- Number of atoms
- Simulation length
- Complexity of the MTP
- Required accuracy

Larger simulations may require:

- More CPU cores
- More memory
- Longer runtime

It is recommended to begin with smaller test simulations before increasing system size.

---

## Next Tutorial

The next tutorial introduces **training MTPs for new materials**, including modifying training datasets and adapting MTP models for different metals, alloys, and ionic systems.
