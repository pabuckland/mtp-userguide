# MTP Pruning Workflow (Trillium / Alliance Canada)

This tutorial walks through training and pruning a Moment Tensor Potential (MTP) using the `mlip-3-prune` fork, on a SLURM cluster (Trillium).

## Prerequisites

- Access to the shared MTP install: `/home/<user>/links/projects/def-belandl1/shared/mtp/bin/`
  - Binaries used: `mlp` (standard MLIP-3) and `mlp_prune` (pruning fork — has the `prune`/`extract_problem` commands `mlp` doesn't).
- A training dataset in `.cfg` format (MLIP configuration format).
- An MTP potential template. Blank/untrained templates for levels 6–28 live at:
  `/home/<user>/links/scratch/mlip-3-prune/MTP_templates/<level>.almtp`

> **Note:** all commands below must run on a **compute node**, not the login node — the `mlp`/`mlp_prune` binaries use MPI/InfiniBand and are blocked on login nodes. Use `debugjob` for quick tests (60 min cap) and `sbatch` for real runs (training and pruning both take longer than that).

---

## 1. Set up your working folder

```bash
mkdir -p ~/scratch/my-run/data
cd ~/scratch/my-run
```

Copy in your potential template and training set:

```bash
cp /home/<user>/links/scratch/mlip-3-prune/MTP_templates/18.almtp data/18_template.almtp
cp /path/to/your/training_set.cfg data/
```

## 2. Match the template's species count to your dataset

Templates ship with a placeholder `species_count` — you must edit it to match the number of distinct atom types in your `.cfg` file.

Check how many species are in your training set:
```bash
grep -oE "^\s*[0-9]+\s+[0-9]+" data/training_set.cfg | awk '{print $2}' | sort -u
```

Check the template's current value:
```bash
grep species_count data/18_template.almtp
```

If they don't match, edit it (example: 1 → 2 species):
```bash
sed -i 's/species_count = 1/species_count = 2/' data/18_template.almtp
```

> Running `extract_problem` or `prune` with a mismatched species count fails with:
> `Rank 0, Atomic number 1 is not present in the MTP potential!`

## 3. Train the potential

A freshly edited template has no fitted parameters yet — it must be trained on your `.cfg` data before it can be pruned. This takes over an hour for a real dataset, so submit it as a batch job.

Create `train_job.sh`:
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

srun -n 1 /home/<user>/links/projects/def-belandl1/shared/mtp/bin/mlp train \
    data/18_template.almtp data/training_set.cfg \
    --save_to=out/18_trained.almtp \
    --iteration_limit=100 \
    --al_mode=nbh
EOF
mkdir -p out
sbatch train_job.sh
```

> **Trillium-specific:** don't set `#SBATCH --mem=...` — this cluster schedules whole nodes and rejects memory requests. Also, `sbatch` must be run **from the login node**, not from inside a `debugjob`/compute-node session.

Check progress:
```bash
squeue -u <user>
```
When the job disappears from the queue, it's done. Confirm the output exists:
```bash
ls -la out/18_trained.almtp
```

## 4. Extract the pruning problem (`xtwx.bin`, `xtwy.bin`, `yTWy`, neighbor count)

This step builds the matrices `prune` needs, and prints two values you'll copy into `config.json`.

Get a quick debug allocation (this step is fast, no need for `sbatch`):
```bash
debugjob
```

Run:
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
cat > config.json << 'EOF'
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
EOF
```

Field notes:
| Field | Meaning |
|---|---|
| `mtp_file` | your **trained** potential from Step 3 |
| `neigh_count`, `ytwy_train`/`ytwy_val` | values printed by `extract_problem` in Step 4 |
| `xtwx_*`/`xtwy_*` | files written by `extract_problem` in Step 4 |
| `pop_size` / `n_gen` / `time` | NSGA-II population size, generation cap, and wall-clock cap (seconds) — tune to taste |
| `restart_from` | leave `""` unless resuming a previous prune run |

## 6. Run pruning

Longer than a debug allocation allows — submit as a batch job.

Create `prune_job.sh`:
```bash
cat > prune_job.sh << 'EOF'
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
EOF
sbatch prune_job.sh
```

> Run `sbatch` from the **login node**. If you're inside a `debugjob` session from a previous step, `exit` first.

Check progress:
```bash
squeue -u <user>
```

## 7. Read the results

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

## 8. Download results to your local machine

From your **local** terminal (not the cluster):
```bash
scp <user>@<login-host>:/path/to/my-run/out_<timestamp>/pareto_final_objectives.csv ~/Downloads/
```
Or use your editor's remote file browser (e.g. VS Code Remote-SSH → right-click file → Download).

---

## Troubleshooting

| Symptom | Cause / Fix |
|---|---|
| `Failed to modify UD QP to INIT on mlx5_0: Operation not permitted` | You're on the login node — MPI/InfiniBand is blocked there. Use `debugjob` or `sbatch`. |
| `unknown link speed 0x80`, `OpenFabrics device` warnings | Harmless noise from Open MPI probing network hardware. Safe to ignore. |
| `Error: command prune does not exist.` | You ran `mlp` instead of `mlp_prune`. Only the pruning fork has `prune`/`extract_problem`. |
| `Rank 0, Atomic number 1 is not present in the MTP potential!` | Potential's `species_count` doesn't match the training set — see Step 2. |
| `Job submission for the partition compute cannot be done from this node...` | You tried `sbatch` from inside a compute-node session. `exit` back to the login node first. |
| `SBATCH ERROR: The --mem=... request is not allowed...` | Trillium schedules whole nodes only — remove any `#SBATCH --mem=` line. |
