# MTP Training Workflow

This tutorial walks through training a Moment Tensor Potential (MTP) from a training dataset using the standard MLIP-3 `mlp` executable on a SLURM cluster (Trillium).

This is separate from the MTP pruning workflow: it covers taking a training dataset, preparing an MTP template, training the MTP, and confirming the trained potential was produced.

## Prerequisites

- Access to the shared MTP install:
  `/home/<user>/links/projects/def-belandl1/shared/mtp/apps/mlip-3/bin/`
- A training dataset in `.cfg` format (MLIP configuration format).
- An untrained MTP template (`.almtp`).

> **Note:** all commands that actually run `mlp` should be executed on a **compute node**, not the login node. On a login node, MLIP-3 can produce errors such as `Failed to modify UD QP to INIT on mlx5_0: Operation not permitted` due to MPI/InfiniBand.

---

## 1. Set Up the Directory

> **Important:** use `~/links/scratch/...`, not `~/scratch/...`. On Trillium, `~/links/scratch` is a symlink to your real, compute-node-writable `$SCRATCH` (e.g. `/scratch/<user>`). A plain `~/scratch` folder lives inside your home directory, which is **read-only on compute nodes** — SLURM jobs will fail trying to write output there.

```bash
mkdir -p ~/links/scratch/mtp-training-example/data
cd ~/links/scratch/mtp-training-example
```

## 2. Add Your Training Dataset

Copy your `.cfg` dataset into `data/`:

```bash
cp "/home/<user>/links/scratch/<your-dataset>.cfg" data/
```

For the rest of this tutorial, replace `data/training_set.cfg` with your dataset's actual name.

## 3. Check the Number of Configurations

```bash
grep -c "BEGIN_CFG" data/training_set.cfg
```

This confirms the dataset was copied correctly and contains the expected number of structures (e.g. `1159`).

## 4. Dataset Format Reference

MLIP-3 expects `.cfg` format:

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
 Energy
       -235.183097220000
 PlusStress:  xx          yy          zz          yz          xz          xy
        ...
END_CFG
```

Key components: `BEGIN_CFG`/`END_CFG`, `Size`, `Supercell`, `AtomData`, `Energy`, and stress info. If your dataset already looks like this, no conversion is needed.

## 5. Check the Atomic Species

```bash
grep -oE "^[[:space:]]*[0-9]+[[:space:]]+[0-9]+" data/training_set.cfg | awk '{print $2}' | sort -u
```

The unique type values (e.g. `0` and `1`) tell you how many species the MTP template needs to support — you'll need this for step 6.

## 6. Obtain and Configure an MTP Template

Copy a template from the shared install (18 = MTP level; higher levels are more expressive but costlier):

```bash
cp /home/<user>/links/scratch/mlip-3-prune/MTP_templates/18.almtp data/18_template.almtp
```

Check and set the species count so it matches your dataset (from step 5):

```bash
grep species_count data/18_template.almtp
sed -i 's/species_count = 1/species_count = 2/' data/18_template.almtp   # only if it needs changing
```

**Important:** a mismatched `species_count` will cause training or calculation failures.

## 7. Training Command

The basic MLIP-3 training command:

```bash
mlp train <potential> <training-data> --save_to=<output-potential>
```

| Argument | Meaning |
|---|---|
| `train` | Tells MLIP-3 to train an MTP |
| `data/18_template.almtp` | Initial MTP template |
| `data/training_set.cfg` | Training dataset |
| `--save_to=...` | File where the trained MTP will be saved |

## 8. Create and Submit the Training Job

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

sbatch train_job.sh
```

Notes:
- `--iteration_limit=100` is fine for demonstrating the workflow; increase it for a production-quality MTP.
- `--al_mode=nbh` enables neighborhood-based active learning.

## 9. Confirm the Trained MTP Was Created

Once the job finishes:

```bash
ls -lh out/18_trained.almtp
```

If the file exists with a non-zero size, training succeeded. If not, check `train_<jobid>.out` for errors.

## 10. Verify and Use the MTP

From a compute node (`debugjob`):

```bash
srun -n 1 /home/<user>/links/projects/def-belandl1/shared/mtp/apps/mlip-3/bin/mlp calculate_efs \
    out/18_trained.almtp \
    data/training_set.cfg \
    out/calculated.cfg
```

This calculates MTP energies and forces for the dataset, and also confirms the potential loads correctly. Run this from a compute node, not the login node.

## 11. Workflow Summary

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

The key output is `out/18_trained.almtp` — usable for further calculations, LAMMPS simulations, or as the input to the pruning workflow.

## 12. Training vs. Pruning

- **Training** uses `mlp` / `mlp train`: `Dataset → MTP template → mlp train → Trained MTP`
- **Pruning** uses `mlp_prune` (`extract_problem`, `prune`) and starts from an already-trained MTP: `Trained MTP → extract_problem → Pruning matrices → prune → Pareto-optimal reduced MTPs`

The MTP trained in this tutorial can be used as the input for the pruning tutorial.

## 13. Troubleshooting

| Symptom | Cause / Fix |
|---|---|
| `Failed to modify UD QP to INIT on mlx5_0: Operation not permitted` | Ran on a login node. Use `debugjob` or `sbatch` instead. |
| `unknown link speed 0x80` | MPI probing network hardware on the login node. Use a compute node. |
| Segfault right after `mlp convert` | `mlp convert` needs both input and output filenames — see note below. |
| `cannot open ... for reading` | Check the input path and your working directory. |
| `Atomic number ... is not present in the MTP potential` | Template's `species_count` doesn't match the dataset (step 6). |
| `out/18_trained.almtp` missing | Check `train_<jobid>.out`; confirm the SLURM job completed. |
| `sbatch` submission error | Run `sbatch` from the login node, not from inside `debugjob`. |
| `Job output requested to be written to file ... which is read-only on the compute nodes` | Your working directory is under plain `~/scratch`, not `~/links/scratch` (real `$SCRATCH`). Move your working directory under `~/links/scratch/...` — see step 1. |
| Training slower than expected | Increase SLURM wall time and/or adjust `--iteration_limit`. |

**Note on `mlp convert`:** not needed if your data is already `.cfg`. It requires both filenames: `mlp convert [options] inputfilename outputfilename` (e.g. `mlp convert --input_format=outcar OUTCAR out/relax.cfg`). Running `mlp convert` alone is incomplete and will fail.

## 14. Final Directory Structure

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
