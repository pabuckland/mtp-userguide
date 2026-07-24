# Tutorial 5 – Understanding High Performance Computing (HPC)

## Overview

In the previous tutorial, you learned the basic Linux commands needed to work on a DRAC cluster. The next step is understanding how High Performance Computing (HPC) systems are organized and why they are used for research.

By the end of this tutorial, you will understand:

- What High Performance Computing (HPC) is
- Why researchers use HPC systems
- The structure of a DRAC cluster
- The difference between login nodes and compute nodes
- The role of computing resources such as CPU cores and memory

---

# What is High Performance Computing?

High Performance Computing (HPC) uses many powerful computers working together to solve problems that would be difficult or time-consuming for a personal computer.

These computers, called **compute nodes**, are connected through a high-speed network and work together as part of a larger system called a **cluster**.

Researchers use HPC systems for tasks such as:

- Molecular dynamics simulations
- Machine learning
- Materials science
- Climate modelling
- Computational chemistry
- Large-scale data analysis

Many of these calculations require significant computing power and can take hours, days, or even weeks to complete on a personal computer.

---

# Why Use an HPC Cluster?

Although modern laptops and desktop computers are powerful, they have limitations.

An HPC cluster provides access to:

- Thousands of CPU cores
- Large amounts of memory (RAM)
- High-speed storage
- Specialized research software
- Multiple computers working together

These resources allow researchers to run larger and more complex simulations than would be practical on a personal computer.

---

# Understanding the Cluster Structure

When working on Trillium, you are using a collection of computers that each have a specific purpose.

A simplified workflow looks like this:

```text
Your Computer
      │
      ▼
Login Node
      │
      ▼
Slurm Job Scheduler
      │
      ▼
Compute Node(s)
      │
      ▼
Simulation Results
```

You connect to a **login node**, prepare your files, and submit jobs using **Slurm**. Slurm then sends your job to one or more **compute nodes**, where the calculations are performed.

---

# Login Nodes

When you connect to Trillium through SSH or Visual Studio Code, you are connected to a **login node**.

Login nodes are intended for tasks such as:

- Editing files
- Writing scripts
- Compiling software
- Organizing project directories
- Submitting jobs

Because login nodes are shared by many users, computationally intensive programs should **not** be run directly on them.

---

# Compute Nodes

Compute nodes perform the actual calculations.

Once a job is submitted, Slurm assigns available resources and runs the job on one or more compute nodes.

Depending on the simulation, a job may use:

- A single CPU core
- Multiple CPU cores
- Multiple compute nodes

Users generally do not connect directly to compute nodes. Instead, Slurm automatically manages where jobs are run.

---

# CPU Cores

A **CPU core** is a processing unit capable of performing calculations.

Most compute nodes contain many CPU cores, allowing multiple calculations to be performed at the same time.

Some simulations use only one core, while others can take advantage of many cores to reduce computation time.

When submitting a job, you will often specify how many CPU cores your simulation requires.

---

# Memory (RAM)

Memory, or **Random Access Memory (RAM)**, stores the data a program needs while it is running.

For molecular dynamics simulations, memory is used to store information such as:

- Atom positions
- Velocities
- Forces
- Neighbor lists
- Simulation data

Larger simulations generally require more memory because they contain more atoms and generate more data.

When submitting jobs through Slurm, you may be asked to request a specific amount of memory for your simulation.

---

# Shared Resources

A DRAC cluster is shared by many researchers.

To ensure fair access for everyone:

- Use login nodes only for lightweight tasks.
- Run simulations on compute nodes.
- Request only the resources your job requires.
- Remove unnecessary files when they are no longer needed.

Using resources responsibly helps the cluster remain efficient for all users.

---

# Preparing for Slurm

Before a simulation can run on a compute node, resources must be requested through the **Slurm Workload Manager**.

Slurm is responsible for:

- Receiving job requests
- Allocating available resources
- Starting jobs
- Monitoring running jobs
- Releasing resources after jobs finish

The next tutorial introduces Slurm and explains how to submit and manage jobs on a DRAC cluster.

---

## Next Tutorial

Now that you understand how HPC clusters are organized, the next tutorial introduces the **Slurm Workload Manager**, which is used to request resources and run jobs on DRAC clusters.
