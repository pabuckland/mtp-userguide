# MTP Pruning Tutorial

This tutorial walks through training and pruning a Moment Tensor Potential (MTP) using the `mlip-3-prune` fork, on a SLURM cluster (Trillium).

This tutorial builds on the [MTP Training Workflow](./mtp-training-workflow.md) — refer to that tutorial for the general explanation of the training command, job submission, and login-node vs. compute-node behavior. This page focuses on what's specific to pruning.

## Prerequisites

- Access to the shared MTP install: `/home/<user>/links/projects/def-belandl1/shared/mtp/bin/`
  - Binaries used: `mlp` (standard MLIP-3) and `mlp_prune` (pruning fork — has the `prune`/`extract_problem` commands `mlp` doesn't).
- A training dataset in `.cfg` format (MLIP configuration format).
- An MTP potential template. Blank/untrained templates for levels 6–28 live at:
  `/home/<user>/links/scratch/mlip-3-prune/MTP_templates/<level>.almtp`

> **Note:** all commands below must run on a **compute node**, not the login node (see the Training tutorial for why). Use `debugjob` for quick tests (60 min cap) and `sbatch` for real runs.

---

## Recommended Training Dataset

If you do not already have a training dataset, the following dataset is recommended for following this tutorial:

**Unified_training_set2_1159.cfg**

Provided by the MTPu project:

https://gitlab.com/Kazongogit/MTPu/-/blob/main/datasets/Unified_training_set2_1159.cfg?ref_type=headsis

An MLIP `.cfg` dataset containing 1159 structures.

### Download the dataset

From the Trillium login node:

```bash
mkdir -p ~/scratch/my-run/data
cd ~/scratch/my-run

wget -O data/training_set.cfg \
"https://gitlab.com/Kazongogit/MTPu/-/raw/main/datasets/Unified_training_set2_1159.cfg"
```

The rest of this tutorial refers to the dataset as `data/training_set.cfg`.

---

## 1. Set Up Your Working Folder

```bash
mkdir -p ~/scratch/my-run/data
cd ~/scratch/my-run

cp /home/<user>/links/scratch/mlip-3-prune/MTP_templates/18.almtp data/18_template.almtp
cp /path/to/your/training_set.cfg data/
```

## 2. Match the Template's Species Count

As in the Training tutorial, the template's `species_count` must match the number of distinct atom types in your `.cfg` file, or `extract_problem`/`prune` will fail with `Atomic number 1 is not present in the MTP potential!`.

```bash
grep -oE "^\s*[0-9]+\s+[0-9]+" data/training_set.cfg | awk '{print $2}' | sort -u
grep species_count data/18_template.almtp
sed -i 's/species_count = 1/species_count = 2/' data/18_template.almtp   # only if it needs changing
```

## 3. Train the Potential

A freshly edited template has no fitted parameters yet — it must be trained before it can be pruned. Use the same `mlp train` process as the [Training tutorial](./mtp-training-workflow.md), pointed at this working folder:

```bash
cat > train_job.sh << 'JOBEOF'
#!/bin/bash
#SBATCH --account=def-belandl1
#SBATCH --time=03:00:00
#SBATCH --cpus-per-task=1
#SBATCH --job-name=mtp_train
#SBATCH --output=train_%j.out

export OPENBLAS_NUM_THREADS=1
export OMP_NUM_THREADS=1
export MKL_NUM_THREADS=1

srun -n 1 /home/<user>/links/projects/def-belandl1/shared/mtp/bin/mlp train \
    data/18_template.almtp data/training_set.cfg \
    --save_to=out/18_trained.almtp \
    --iteration_limit=100 \
    --al_mode=nbh
JOBEOF
mkdir -p out
sbatch train_job.sh
```

> **Trillium-specific:** don't set `#SBATCH --mem=...` — this cluster schedules whole nodes and rejects memory requests.

Once `squeue -u <user>` shows the job is done, confirm the output exists: `ls -la out/18_trained.almtp`.

## 4. Extract the Pruning Problem

This step builds the matrices `prune` needs (`xtwx.bin`, `xtwy.bin`), and prints two values you'll copy into `config.json`. It's fast, so a `debugjob` is enough:

```bash
debugjob
```

```bash
srun -n 1 /home/<user>/links/projects/def-belandl1/shared/mtp/bin/mlp_prune extract_problem \
    out/18_trained.almtp data/training_set.cfg data/xtwx.bin data/xtwy.bin
```

Output looks like:
```
Sum of Squared Ground Truths (yTWy): 6483714564.847916603088379
Average number of neighbors = 30.616666
Extraction Complete!
```

Copy those two numbers — you need them for the next step.

## 5. Write `config.json`

```bash
cat > config.json << 'CFGEOF'
{
  "mtp_file": "out/18_trained.almtp",
  "pop_size": 1152,
  "n_gen": 30000,
  "time": 3000,
  "save_interval": 1000,
  "regularization": 1e-9,
  "neigh_count": 30.616666,

  "seed": 69,
  "max_fill": 1,

  "ytwy_train": 6483714564.847916603088379,
  "xtwx_train_file": "data/xtwx.bin",
  "xtwy_train_file": "data/xtwy.bin",

  "ytwy_val": 6483714564.847916603088379,
  "xtwx_val_file": "data/xtwx.bin",
  "xtwy_val_file": "data/xtwy.bin",

  "out_dir": "out",
  "restart_from": ""
}
CFGEOF
```

Field notes:
| Field | Meaning |
|---|---|
| `mtp_file` | your **trained** potential from Step 3 |
| `neigh_count`, `ytwy_train`/`ytwy_val` | values printed by `extract_problem` in Step 4 |
| `xtwx_*`/`xtwy_*` | files written by `extract_problem` in Step 4 |
| `pop_size` / `n_gen` / `time` | NSGA-II population size, generation cap, and wall-clock cap (seconds) — tune to taste |
| `restart_from` | leave `""` unless resuming a previous prune run |

## 6. Run Pruning

Longer than a debug allocation allows — submit as a batch job (from the login node — `exit` first if you're still in a `debugjob` session):

```bash
cat > prune_job.sh << 'PRUNEEOF'
#!/bin/bash
#SBATCH --account=def-belandl1
#SBATCH --time=01:30:00
#SBATCH --cpus-per-task=1
#SBATCH --job-name=mtp_prune
#SBATCH --output=prune_%j.out

export OPENBLAS_NUM_THREADS=1
export OMP_NUM_THREADS=1
export MKL_NUM_THREADS=1

srun -n 1 /home/<user>/links/projects/def-belandl1/shared/mtp/bin/mlp_prune prune config.json
PRUNEEOF
sbatch prune_job.sh
```

## 7. Read the Results

Once the job finishes, look for a new timestamped output folder:
```bash
ls -la
```
You'll see something like `out_20260805_212432/` containing:

| File | Contents |
|---|---|
| `pareto_final_objectives.csv` | The Pareto-optimal trade-off curve — two columns, one row per non-dominated solution. Row count matches `pop_size` at most, but is usually smaller since only non-dominated points survive. |
| `pareto_final_population.csv` | Full parameter/mask data for every retained individual (much larger). |

View the front:
```bash
cat out_<timestamp>/pareto_final_objectives.csv
```
Column 1 and 2 are your two optimization objectives (e.g. pruned model size/complexity vs. fit error ratio) — low column 1 + high column 2 means aggressive pruning at the cost of accuracy; the curve shows the full range of that trade-off.

## 8. Download Results to Your Local Machine

From your **local** terminal (not the cluster):
```bash
scp <user>@<login-host>:/path/to/my-run/out_<timestamp>/pareto_final_objectives.csv ~/Downloads/
```
Or use your editor's remote file browser (e.g. VS Code Remote-SSH → right-click file → Download).

---

## Troubleshooting

For general login-node/InfiniBand errors (`UD QP`, `unknown link speed`) and `sbatch`-from-compute-node issues, see the [Training tutorial's troubleshooting table](./mtp-training-workflow.md#13-troubleshooting) — the same fixes apply here. Pruning-specific issues:

| Symptom | Cause / Fix |
|---|---|
| `Error: command prune does not exist.` | You ran `mlp` instead of `mlp_prune`. Only the pruning fork has `prune`/`extract_problem`. |
| `Rank 0, Atomic number 1 is not present in the MTP potential!` | Potential's `species_count` doesn't match the training set — see Step 2. |
| `SBATCH ERROR: The --mem=... request is not allowed...` | Trillium schedules whole nodes only — remove any `#SBATCH --mem=` line. |
````
