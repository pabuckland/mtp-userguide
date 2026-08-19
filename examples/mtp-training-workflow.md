````markdown
# MTP Training Workflow (Trillium / Alliance Canada)

This tutorial walks through training a Moment Tensor Potential (MTP) from a training dataset using the standard MLIP-3 `mlp` executable on a SLURM cluster (Trillium).

This tutorial is separate from the MTP pruning workflow. It demonstrates how to take a training dataset, prepare an MTP template, train the MTP, and verify that the trained potential was successfully produced.

## Prerequisites

- Access to the shared MTP install:
  `/home/<user>/links/projects/def-belandl1/shared/mtp/apps/mlip-3/bin/`
- The standard MLIP-3 `mlp` executable.
- A training dataset in `.cfg` format (MLIP configuration format).
- An untrained MTP template (`.almtp`).

> **Note:** all commands that actually run `mlp` should be executed on a **compute node**, not the login node. The MLIP-3 executable uses MPI/InfiniBand and can produce errors such as `Failed to modify UD QP to INIT on mlx5_0: Operation not permitted` when run directly on a login node.

---

## 1. Create a Directory for the Training Example

Start by creating a separate directory for your MTP training example.

From the Trillium login node:

```bash
mkdir -p ~/scratch/mtp-training-example/data
cd ~/scratch/mtp-training-example
```

Check that you are in the correct directory:

```bash
pwd
```

You should see something similar to:

```
/home/<user>/links/scratch/mtp-training-example
```

The exact path may differ depending on your account.

## 2. Add Your Training Dataset

For this tutorial, use your own `.cfg` training dataset rather than the Unified Training Set used in the pruning tutorial.

For example, if your dataset is already located in your scratch space:

```bash
cp "/home/<user>/links/scratch/<your-dataset>.cfg" data/
```

Check that the dataset is present:

```bash
ls -lh data/
```

You should see your `.cfg` file.

For the remainder of this tutorial, replace:

```
data/training_set.cfg
```

with the actual name of your dataset.

## 3. Check the Number of Configurations

Before training, it is useful to check how many configurations are contained in your dataset.

Run:

```bash
grep -c "BEGIN_CFG" data/training_set.cfg
```

For example:

```
1159
```

would indicate that the dataset contains 1159 configurations.

This is a useful sanity check to make sure the dataset was copied correctly and contains the expected number of structures.

## 4. Check the Dataset Format

MLIP-3 expects the training data to be in its configuration (`.cfg`) format.

A configuration should look similar to:

```
BEGIN_CFG
 Size
    128
 Supercell
        14.080000      0.000000      0.000000
         0.000000     14.080000      0.000000
         0.000000      0.000000     14.080000
 AtomData:  id type       cartes_x      cartes_y      cartes_z           fx          fy          fz
             1    0       ...
             2    0       ...
             ...
 Energy
       -235.183097220000
 PlusStress:  xx          yy          zz          yz          xz          xy
        ...
END_CFG
```

The important components are:

- `BEGIN_CFG` / `END_CFG`
- `Size`
- `Supercell`
- `AtomData`
- atomic positions
- forces, if available
- `Energy`
- stress information, if available

If your dataset is already in this format, no conversion is necessary.

## 5. Check the Atomic Species

The atomic type values in the `.cfg` file determine how many species the MTP needs to support.

For example, to see the unique type values:

```bash
grep -oE "^[[:space:]]*[0-9]+[[:space:]]+[0-9]+" data/training_set.cfg | awk '{print $2}' | sort -u
```

If the output is:

```
0
```

then the dataset contains one species.

If the output is:

```
0
1
```

then the dataset contains two species.

Record the number of unique types because the MTP template must use the same number of species.

## 6. Obtain an MTP Template

The shared MTP installation contains MTP templates that can be used as the starting point for training.

For example:

```bash
cp /home/<user>/links/scratch/mlip-3-prune/MTP_templates/18.almtp data/18_template.almtp
```

Check that it was copied:

```bash
ls -lh data/18_template.almtp
```

The number 18 refers to the MTP level.

Different MTP levels provide different model complexities. Higher levels generally provide more expressive potentials but require more computational resources.

## 7. Check the Template's Species Count

Check the species count in the template:

```bash
grep species_count data/18_template.almtp
```

You may see something similar to:

```
species_count = 1
```

This value must correspond to the number of species in your training dataset.

For example, if your dataset contains two species and the template currently has:

```
species_count = 1
```

change it to:

```bash
sed -i 's/species_count = 1/species_count = 2/' data/18_template.almtp
```

Check it again:

```bash
grep species_count data/18_template.almtp
```

It should now show:

```
species_count = 2
```

**Important:** the template's `species_count` must match the species present in the training dataset. Otherwise, MLIP-3 can fail when trying to train or calculate properties.

## 8. Understand the Training Command

The basic MLIP-3 training command is:

```bash
mlp train <potential> <training-data> --save_to=<output-potential>
```

For example:

```bash
mlp train \
    data/18_template.almtp \
    data/training_set.cfg \
    --save_to=out/18_trained.almtp
```

The arguments are:

| Argument | Meaning |
|---|---|
| `train` | Tells MLIP-3 to train an MTP |
| `data/18_template.almtp` | Initial MTP template |
| `data/training_set.cfg` | Training dataset |
| `--save_to=...` | File where the trained MTP will be saved |

The template is the starting model, while the `.cfg` dataset provides the reference energies, forces, and stresses used to fit the MTP parameters.

## 9. Create the Training Job

Because MTP training can take longer than a short interactive allocation, submit the training as a SLURM batch job.

Create the job script:

```bash
cat > train_job.sh << 'EOF'
#!/bin/bash
#SBATCH --account=def-belandl1
#SBATCH --time=03:00:00
#SBATCH --cpus-per-task=1
#SBATCH --job-name=mtp_train
#SBATCH --output=train_%j.out

export OPENBLAS_NUM_THREADS=1
export OMP_NUM_THREADS=1
export MKL_NUM_THREADS=1

mkdir -p out

srun -n 1 /home/<user>/links/projects/def-belandl1/shared/mtp/apps/mlip-3/bin/mlp train \
    data/18_template.almtp \
    data/training_set.cfg \
    --save_to=out/18_trained.almtp \
    --iteration_limit=100 \
    --al_mode=nbh
EOF
```

The important part of the script is:

```bash
srun -n 1 /home/<user>/links/projects/def-belandl1/shared/mtp/apps/mlip-3/bin/mlp train \
    data/18_template.almtp \
    data/training_set.cfg \
    --save_to=out/18_trained.almtp \
    --iteration_limit=100 \
    --al_mode=nbh
```

Here:

- `mlp train` starts MTP training.
- `data/18_template.almtp` is the initial template.
- `data/training_set.cfg` is the training dataset.
- `--save_to=out/18_trained.almtp` specifies the trained MTP output.
- `--iteration_limit=100` limits the number of training iterations for this example.
- `--al_mode=nbh` enables the neighborhood-based active-learning mode.

**Note:** the iteration limit can be increased for a production-quality MTP. A value such as 100 is useful for demonstrating the workflow without requiring a very long training run.

## 10. Submit the Training Job

Submit the job from the login node:

```bash
sbatch train_job.sh
```

You should receive a message similar to:

```
Submitted batch job 1234567
```

The job ID will be different for your run.

Check the status:

```bash
squeue -u <user>
```

You can also inspect the job output:

```bash
ls -lh train_*.out
```

Once the job finishes, the output file should contain information about the training process.

## 11. Check That the Trained MTP Was Created

After the job finishes, check the output directory:

```bash
ls -lh out/
```

You should see:

```
18_trained.almtp
```

You can also check the file directly:

```bash
ls -lh out/18_trained.almtp
```

If the file exists and has a non-zero size, the training executable successfully produced an MTP.

## 12. Inspect the Trained MTP

You can inspect the beginning of the trained potential:

```bash
head -30 out/18_trained.almtp
```

You should see information describing the trained MTP, including its species and fitted parameters.

The important distinction is that:

```
data/18_template.almtp
```

is the initial/untrained template, while:

```
out/18_trained.almtp
```

is the fitted potential produced by the training process.

## 13. Verify the MTP with mlp

Before using the potential for further calculations, verify that MLIP-3 can read it.

Start a compute-node allocation:

```bash
debugjob
```

Then run:

```bash
srun -n 1 /home/<user>/links/projects/def-belandl1/shared/mtp/apps/mlip-3/bin/mlp list
```

This should list the available MLIP-3 commands.

You can also use the trained potential in an MLIP-3 calculation such as:

```bash
srun -n 1 /home/<user>/links/projects/def-belandl1/shared/mtp/apps/mlip-3/bin/mlp calculate_efs \
    out/18_trained.almtp \
    data/training_set.cfg \
    out/calculated.cfg
```

This calculates the MTP energies and forces for the configurations in the dataset.

**Important:** run MLIP-3 commands such as `calculate_efs` from a compute node. Running the executable directly on a login node can produce errors involving `mlx5`, `UD QP`, or InfiniBand.

When finished with the interactive allocation:

```bash
exit
```

## 14. Check the Calculated Results

After `calculate_efs` completes:

```bash
ls -lh out/
```

You should see the calculated configuration file:

```
calculated.cfg
```

You can inspect it with:

```bash
head -50 out/calculated.cfg
```

The calculated file contains the configurations evaluated using your trained MTP.

## 15. What Has Been Accomplished

At this point, you have completed the basic MTP training workflow:

```
Training dataset (.cfg)
        |
        v
MTP template (.almtp)
        |
        v
     mlp train
        |
        v
Trained MTP (.almtp)
        |
        v
   mlp calculate_efs
        |
        v
Calculated configurations (.cfg)
```

The important output from the training stage is:

```
out/18_trained.almtp
```

This is the trained MTP that can subsequently be used for calculations, validation, LAMMPS simulations, or as the starting potential for the pruning workflow.

## 16. Training vs. Pruning

It is important to distinguish the standard MLIP-3 training workflow from the pruning workflow.

### Standard MTP Training

The standard MLIP-3 executable is:

```
mlp
```

and training is performed using:

```
mlp train
```

The general workflow is:

```
Dataset
   |
   v
MTP template
   |
   v
mlp train
   |
   v
Trained MTP
```

### MTP Pruning

The pruning fork uses:

```
mlp_prune
```

and provides additional commands such as:

```
mlp_prune extract_problem
```

and:

```
mlp_prune prune
```

The pruning workflow starts from an already trained MTP:

```
Training dataset
       |
       v
   mlp train
       |
       v
Trained MTP
       |
       v
extract_problem
       |
       v
Pruning matrices
       |
       v
     prune
       |
       v
Pareto-optimal reduced MTPs
```

Therefore, training an MTP and pruning an MTP are separate stages.

The trained potential produced by this tutorial can be used as the input potential for the pruning tutorial.

## 17. Troubleshooting

| Symptom | Cause / Fix |
|---|---|
| `Failed to modify UD QP to INIT on mlx5_0: Operation not permitted` | The MLIP executable was run on a login node. Use `debugjob` or submit the calculation with `sbatch`. |
| `unknown link speed 0x80` | MPI/Open MPI is probing network hardware on the login node. Run the MLIP executable on a compute node instead. |
| Segmentation fault immediately after running `mlp convert` | `mlp convert` requires input and output filenames. Running only `mlp convert` does not perform a conversion. |
| `cannot open ... for reading` | Check that the input filename/path exists and that you are using the correct working directory. |
| `Atomic number ... is not present in the MTP potential` | The MTP template's species configuration does not match the species in the dataset. |
| `out/18_trained.almtp` does not exist | Check `train_<jobid>.out` for errors and confirm that the SLURM job completed successfully. |
| `sbatch` gives a compute-node submission error | Run `sbatch` from the login node, not from inside a `debugjob` allocation. |
| Training takes longer than expected | Increase the requested SLURM wall time and/or use fewer/more iterations depending on the purpose of the training run. |

## 18. Important Note About `mlp convert`

The `convert` command is not required when your dataset is already in MLIP `.cfg` format.

The MLIP-3 source code shows that the command requires both an input filename and an output filename:

```
mlp convert [options] inputfilename outputfilename
```

For example, converting a VASP OUTCAR to MLIP `.cfg` format would use:

```bash
mlp convert --input_format=outcar OUTCAR out/relax.cfg
```

Therefore, this command:

```
mlp convert
```

is incomplete and should not be used by itself.

If your training dataset is already an MLIP `.cfg` file, such as:

```
training_set.cfg
```

you can use it directly with:

```
mlp train
```

There is no need to convert it first.

## 19. Final Directory Structure

After completing the tutorial, your directory should look approximately like:

```
mtp-training-example/
├── data/
│   ├── training_set.cfg
│   └── 18_template.almtp
├── out/
│   ├── 18_trained.almtp
│   └── calculated.cfg
├── train_job.sh
└── train_<jobid>.out
```

The exact files may vary depending on which calculations you perform.

The most important files are:

```
data/training_set.cfg
data/18_template.almtp
out/18_trained.almtp
```

where:

- `training_set.cfg` = your reference training data
- `18_template.almtp` = your starting MTP template
- `18_trained.almtp` = your trained MTP

````
