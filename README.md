<h1 align="center">Tx2Mol</h1>

<p align="center">
  <strong>Phenotype-driven de novo molecular design from gene expression signatures.</strong>
</p>

<p align="center">
  <a href="environment.yml"><img src="https://img.shields.io/badge/Python-3.10-397C91?style=flat-square&amp;logo=python&amp;logoColor=white" alt="Python 3.10" /></a>
  <a href="scripts/install.sh"><img src="https://img.shields.io/badge/PyTorch-2.1.2-D87862?style=flat-square&amp;logo=pytorch&amp;logoColor=white" alt="PyTorch 2.1.2" /></a>
  <a href="environment.yml"><img src="https://img.shields.io/badge/CUDA-11.8-54866B?style=flat-square&amp;logo=nvidia&amp;logoColor=white" alt="CUDA 11.8" /></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/Code-MIT-597080?style=flat-square" alt="Code license: MIT" /></a>
</p>

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark) and (max-width: 720px)" srcset="assets/readme/overview-mobile-dark.svg" />
    <source media="(max-width: 720px)" srcset="assets/readme/overview-mobile-light.svg" />
    <source media="(prefers-color-scheme: dark)" srcset="assets/readme/overview-dark.svg" />
    <img src="assets/readme/overview-light.svg" width="1000" alt="Tx2Mol pipeline: GeneVAE pretraining, Tx2Mol post-training, and phenotype-guided SMILES generation." />
  </picture>
</p>

<p align="center">
  <a href="#2-quick-start-use-the-released-checkpoint">🚀&nbsp;<strong>Quick&nbsp;start</strong></a> &nbsp; · &nbsp;
  <a href="docs/TUTORIAL.md">📖&nbsp;<strong>Tutorial</strong></a> &nbsp; · &nbsp;
  <a href="data/README.md">🧬&nbsp;<strong>Datasets</strong></a> &nbsp; · &nbsp;
  <a href="https://github.com/Yaxin-Xu/Tx2Mol/releases/tag/v1.0">📦&nbsp;<strong>Checkpoints</strong></a> &nbsp; · &nbsp;
  <a href="docs/METHODS.md">🔬&nbsp;<strong>Methods</strong></a>
</p>

## 1. Install the environment

Requires Linux, Conda, and an NVIDIA Ampere-or-newer GPU. The pipeline was tested on one RTX 4090 (24 GB); the environment pins Python 3.10 and CUDA 11.8.

```bash
git clone https://github.com/Yaxin-Xu/Tx2Mol.git
cd Tx2Mol
conda env create -f environment.yml
conda activate tx2mol
export LD_LIBRARY_PATH="$CONDA_PREFIX/lib:${LD_LIBRARY_PATH:-}"
bash scripts/install.sh
python scripts/validate_data.py
```

Run subsequent commands from the repository root with the `tx2mol` environment active. Use a new output directory for each experiment.

## 2. Quick start: use the released checkpoint

Download the Tx2Mol checkpoint and its matching GeneVAE, then generate a small ten-target example:

```bash
python scripts/prepare_assets.py \
  --github_repo Yaxin-Xu/Tx2Mol --tag v1.0 --reference

python -m tx2mol.generate --config configs/generate_reference.json \
  --num_runs 1 --num_samples 10 --batch_size 10 \
  --output_dir outputs/reference_demo
```

This makes **100 generation attempts** across ten targets. Weights are downloaded from [Releases](https://github.com/Yaxin-Xu/Tx2Mol/releases/tag/v1.0) and verified automatically. An [archived execution example](examples/reference_demo/) is included.

For the full experiment (**10 targets**), this single command generates and evaluates for each target:

```bash
python -m tx2mol.generate --config configs/generate_reference.json
```

## 3. Read the results

Results are saved together in `outputs/reference_targets/`:

- `best_max_tanimoto.csv`: **each target's highest maximum Tanimoto**, scoring all valid molecules.
- `best_run_attempts.csv`: all 100 attempts in each winning group.
- `evaluation_summary.json`: the mean of the ten selected target maxima, settings, and checksums.

## 4. Train the three-stage pipeline

Install the starting backbone, then run GeneVAE pretraining, Tx2Mol post-training, and generation in order:

```bash
python scripts/prepare_assets.py \
  --github_repo Yaxin-Xu/Tx2Mol --tag v1.0 --base

python -m tx2mol.pretrain --config configs/pretrain.json
python -m tx2mol.finetune --config configs/finetune.json
python -m tx2mol.generate --config configs/generate.json
```

The defaults connect all three stages; generation automatically evaluates and selects the best groups under `outputs/targets/`. Keep each Tx2Mol checkpoint paired with the GeneVAE used during its training. See the [training tutorial](docs/TUTORIAL.md#4-train-the-three-stage-pipeline) for hyperparameters and implementation details.

## 5. Validate the pipeline

```bash
python -m unittest discover -s tests -v
python scripts/validate_data.py
```

Completed checks and their scope are recorded in [VALIDATION.md](docs/VALIDATION.md).

## 6. Datasets and documentation

Shared training data are under `data/`. The `targets/`, `sciplex3/`, and `patient/` subdirectories each contain `test/` and `known_ligands/`; patient data cover the 12 diseases in the updated heatmap. The executable example covers ten targets; SciPlex3 and patient data require their respective protocols.

- [Full tutorial](docs/TUTORIAL.md): installation, generation, training, and custom inputs.
- [Data guide](data/README.md): file formats, disease list, and ligand collections.
- [Methods](docs/METHODS.md) and [provenance](docs/PROVENANCE.md): architecture, metrics, seeds, and implementation limitations.
- [Configurations](configs/), [data manifest](assets/data_manifest.json), and [weight manifest](assets/release_manifest.json): reproducibility settings and checksums.

## 7. Troubleshooting

<details>
<summary>🔧 Common issues and fixes</summary>

| Problem | Solution |
| --- | --- |
| Environment or FlashAttention errors | Use the pinned environment and rerun `bash scripts/install.sh`. |
| CUDA out of memory | Reduce `--batch_size`; adjust `--grad_accum` during post-training. |
| Checkpoint loading errors | Check model paths and use the GeneVAE paired with the Tx2Mol checkpoint. |

</details>
