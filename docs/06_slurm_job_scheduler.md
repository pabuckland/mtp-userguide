# Tutorial 6 – Using the Slurm Job Scheduler

## Overview

In the previous tutorial, you learned how High Performance Computing (HPC) clusters are organized. The next step is learning how to run programs on those clusters.

DRAC clusters use the **Slurm Workload Manager (Slurm)** to manage computing resources. Instead of running programs directly on a login node, users submit jobs to Slurm, which schedules them to run on available compute nodes.

By the end of this tutorial, you will understand:

- What Slurm is
- Why Slurm is used
- The difference between interactive and batch jobs
- Common Slurm commands
- How to submit and manage jobs

---

# What is Slurm?

Slurm is a job scheduler used on many HPC systems around the world. It manages access to the cluster's computing resources by:

- Receiving job requests
- Placing jobs in a queue
- Allocating available compute nodes
- Starting jobs when resources become available
- Monitoring running jobs
- Releasing resources after jobs finish

Without a scheduler, multiple users could attempt to use the same resources at the same time, causing conflicts and reducing performance.

---

# Why Use Slurm?

Unlike a personal computer, a DRAC cluster is shared by many researchers.

Instead of immediately running a program, users request the resources needed for their calculation, such as:

- CPU cores
- Memory
- Maximum runtime
- Number of compute nodes

Slurm then schedules the job and runs it when the requested resources are available.

This allows many researchers to efficiently share the cluster.

---

# Interactive Jobs

An **interactive job** provides temporary access to a compute node where commands can be run directly.

Interactive jobs are useful for:

- Testing software
- Debugging scripts
- Running short calculations
- Compiling programs

To request an interactive session, use:

```bash
salloc --time=01:00:00 --ntasks=1
```

This command requests:

- 1 hour of runtime
- 1 task (CPU)

Once resources are available, you will be connected to a compute node.

When finished, leave the interactive session using:

```bash
exit
```

---

# Batch Jobs

Most large simulations are submitted as **batch jobs**.

A batch job uses a script that contains the commands you want the cluster to execute. This allows simulations to run automatically without requiring you to remain connected.

Example batch script:

```bash
#!/bin/bash
#SBATCH --job-name=test_job
#SBATCH --time=01:00:00
#SBATCH --ntasks=1

echo "Hello from Trillium!"
hostname
```

Save this file as:

```text
job.sh
```

Submit the job using:

```bash
sbatch job.sh
```

Slurm will assign the job a Job ID and place it into the queue.

---

# Checking the Job Queue

To view your current jobs, use:

```bash
squeue -u $USER
```

This displays information such as:

- Job ID
- Job name
- Job status
- Runtime
- Assigned compute node

Example output:

```text
JOBID   NAME       ST   TIME
123456  test_job   R    0:05
```

The job status indicates whether your job is:

- Running (`R`)
- Pending (`PD`)
- Completed

---

# Viewing Completed Jobs

After a job finishes, information about the job can be viewed using:

```bash
sacct
```

This provides information such as:

- Job ID
- Completion status
- Runtime
- Resource usage

This is useful for checking whether a simulation completed successfully.

---

# Cancelling a Job

If a job is no longer needed, it can be cancelled using:

```bash
scancel JOB_ID
```

For example:

```bash
scancel 123456
```

Replace `123456` with the Job ID shown by:

```bash
squeue -u $USER
```

---

# Typical HPC Workflow

A typical workflow on a DRAC cluster is:

1. Connect to the cluster using SSH or VS Code.
2. Navigate to your project directory.
3. Prepare input files and scripts.
4. Request computing resources using Slurm.
5. Run the simulation on compute nodes.
6. Analyze the output files.

For small tests, an interactive job (`salloc`) is useful.

For longer simulations, a batch job (`sbatch`) is recommended.

---

# Best Practices

When using Slurm:

- Do not run (large) simulations on login nodes.
- Request only the resources your job requires.
- Test new scripts with small jobs first.
- Monitor your jobs regularly.
- Cancel jobs that are no longer needed.

These practices help maintain efficient and fair use of shared HPC resources.

---

## Next Tutorial

Now that you understand the DRAC environment and how to run jobs, the next tutorial introduces the **Moment Tensor Potential (MTP) repositories** and explains how the code is organized before running simulations.
