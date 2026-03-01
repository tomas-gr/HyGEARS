# Installation Guide

## Prerequisites

- [Miniconda or Anaconda](https://docs.conda.io/en/latest/miniconda.html)
- Git with submodule support (any modern version)
- CUDA-capable GPU recommended (CPU-only is supported but slow for training)

---

## 1. Clone the repository

```bash
git clone --recurse-submodules https://github.com/tomas-gr/HyGEARS.git
cd HyGEARS
```

If you already cloned without `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

---

## 2. Select the branch

| Branch | Purpose |
|---|---|
| `master` | Unmodified GEARS baseline — use for benchmarking the original |
| `hyperbolic` | HyGEARS variant with hyperbolic embeddings |

```bash
# To run the original GEARS:
git checkout master

# To run HyGEARS:
git checkout hyperbolic
```

---

## 3. Create the conda environment

> The `environment.yml` file resolves version conflicts between GEARS and hypeGRL.
> See the comments inside it for a summary of each resolved conflict.

```bash
conda env create -f environment.yml
conda activate hygears
```

If the environment already exists and you want to update it after a `git pull`:

```bash
conda env update -f environment.yml --prune
```

### GPU vs CPU

Open `environment.yml` and follow the comment inside the `dependencies:` section to
switch between the GPU (CUDA) and CPU-only PyTorch builds before running the command above.

---

## 4. Make hypeGRL importable

hypeGRL is not a pip-installable package, so Python needs to know where to find it.
The recommended way is to set `PYTHONPATH` before running any script:

```bash
export PYTHONPATH="$PWD/libs/hypeGRL:$PYTHONPATH"
```

To make this permanent for the conda environment (so it is set automatically on
`conda activate hygears`):

```bash
# Run once after creating the environment
conda activate hygears
mkdir -p $CONDA_PREFIX/etc/conda/activate.d
echo "export PYTHONPATH=\"$PWD/libs/hypeGRL:\$PYTHONPATH\"" \
    > $CONDA_PREFIX/etc/conda/activate.d/hygears_path.sh
```

After that, deactivate and reactivate to pick up the change:

```bash
conda deactivate && conda activate hygears
python -c "from hyperbolic_embeddings import HyperbolicEmbeddings; print('OK')"
```

---

## 5. (Optional) hypeGRL sub-dependencies

hypeGRL supports several embedding backends. Some of them require additional
compiled packages that are not installable via pip. You only need these if you
use those specific backends:

| Embedding type | Extra setup needed |
|---|---|
| `poincare_maps` | None — works out of the box |
| `lorentz` | None — works out of the box |
| `hypermap` | Run `make` inside `libs/hypeGRL` (clones and builds `hypermap`) |
| `d-mercator` | Run `make` inside `libs/hypeGRL` (clones and builds `d-mercator`) |

For the `make`-based setup:

```bash
cd libs/hypeGRL
make          # creates its own venv — you do NOT need to activate it
cd ../..
```

---

## 6. Verify the installation

```bash
conda activate hygears
python - <<'EOF'
import torch
import torch_geometric
import scanpy
import gears
from hyperbolic_embeddings import HyperbolicEmbeddings
print("torch          :", torch.__version__)
print("torch_geometric:", torch_geometric.__version__)
print("All imports OK")
EOF
```

---

## Dependency conflict notes

| Package | GEARS pin | hypeGRL pin | Used version | Reason |
|---|---|---|---|---|
| `scipy` | 1.14.1 | 1.13.1 | **1.14.1** | GEARS requires the newer version; 1.14.x is backward-compatible with hypeGRL |
| `pandas` | 2.2.2 | 2.3.3 | **2.3.3** | Newer minor version; backward-compatible |
| `scikit-learn` | 1.5.1 | 1.7.2 | **1.7.2** | Newer minor version; backward-compatible |
| `networkx` | 3.3 | 3.5 | **3.5** | Newer minor version; backward-compatible |
| `torch` | unpinned | 2.8.0 | **2.8.0** | hypeGRL pins it; GEARS works with any recent version |
| `tensorflow` | not used | 2.17.1 | **optional** | Only needed for specific hypeGRL embedding backends; ~500 MB |
