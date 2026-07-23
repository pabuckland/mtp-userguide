# Tutorial 7 – MTP Repository Overview

## Overview

In the previous tutorials, you learned how to connect to DRAC, navigate a Linux environment, and submit jobs using Slurm.

This tutorial introduces the software repositories used throughout the MTP workflow. These repositories contain the source code, examples, and tutorials needed to train and use **Moment Tensor Potentials (MTPs)**.

By the end of this tutorial, you will understand:

- What repositories are used for MTP calculations
- The purpose of each repository
- How the repositories relate to each other
- How to organize an MTP project on DRAC

---

# What is a Repository?

A repository (repo) is a folder that contains all the files needed for a software project.

For MTP, repositories contain the code, examples, and documentation needed to train and use Moment Tensor Potentials.

Repositories can be downloaded to DRAC using Git, allowing users to access and work with the same software and examples provided by the developers.

---

# MTP Software Workflow

The general workflow for using Moment Tensor Potentials is:

```
Training Data
      |
      ↓
MLIP-2 / MLIP-3
      |
      ↓
Generate MTP Potential
      |
      ↓
LAMMPS + MTP
      |
      ↓
Run Molecular Dynamics Simulation
```

Each repository supports a different stage of this workflow.

---

# Main MTP Repositories

The following repositories are used throughout this guide:

## MLIP-2 Tutorials

Repository:

```
https://gitlab.com/ashapeev/mlip-2-tutorials/
```

The MLIP-2 tutorial repository provides examples and instructions for learning the original Moment Tensor Potential implementation.

It introduces concepts such as:

- Preparing training data
- Training an MTP model
- Using input files
- Evaluating potentials

This repository is the recommended starting point for learning the MTP workflow.

---

## MLIP-3

Repository:

```
https://gitlab.com/ashapeev/mlip-3
```

MLIP-3 is the newer version of the MLIP software package.

It builds upon MLIP-2 and includes updated functionality for developing and applying machine-learning interatomic potentials.

This repository is used to explore:

- Newer MTP features
- Updated workflows
- Improvements compared with MLIP-2

The MLIP-3 workflow will be introduced after the MLIP-2 tutorial.

---

## MTP Demo Repository

Repository:

```
https://github.com/RichardZJM/mtp-demo
```

The MTP demo repository provides practical examples showing how MTP can be used in simulations.

It is useful for understanding:

- Example workflows
- Input file structures
- Running MTP calculations
- Connecting MTP with simulation software

---

# How the Repositories Work Together

The repositories are not separate programs that must all be run at once.

Instead, they represent different stages of learning and using MTP:

```
MLIP-2 Tutorials
        |
        ↓
Learn MTP training workflow

        |
        ↓

MLIP-3
        |
        ↓
Explore newer MTP capabilities

        |
        ↓

MTP Demo
        |
        ↓
Apply MTP in practical simulations
```

---

# Organizing MTP Projects on DRAC

A recommended project structure is:

```
MTP_project/

├── mlip-2/
│
├── mlip-3/
│
├── mtp-demo/
│
├── training_data/
│
├── potentials/
│
├── simulations/
│
└── results/
```

This organization separates:

- Software repositories
- Training files
- Generated potentials
- Simulation outputs

Keeping files organized is especially important when running multiple simulations on an HPC system.

---

# Cloning a Repository

Repositories can be downloaded using Git.

Example:

```bash
git clone https://gitlab.com/ashapeev/mlip-2-tutorials/
```

This creates a local copy of the repository on the cluster.

After cloning, enter the repository:

```bash
cd mlip-2-tutorials
```

View the contents:

```bash
ls
```

---

# Checking Repository Contents

Most repositories contain files such as:

```
README.md
examples/
scripts/
source/
input files
```

The `README.md` file usually contains important instructions from the developers.

Before running any code:

1. Read the README file.
2. Understand the purpose of each folder.
3. Identify required software dependencies.
4. Test example calculations first.

---

# Working with Repositories on DRAC

When using DRAC, repositories should normally be stored in your project or scratch directory rather than your home directory.

Example:

```
~/projects/MTP_project/
```

or:

```
~/scratch/MTP_project/
```

The location depends on the size of your calculations and storage requirements.

---

## Next Tutorial

The next tutorial introduces the **MLIP-2 tutorial workflow**, including installation, training data preparation, and generating Moment Tensor Potentials.
