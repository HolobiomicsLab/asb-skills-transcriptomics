---
name: cpu-gpu-training-speedup-measurement
description: Use when you need to measure the device speed-up of GPU vs CPU training
  for a scETM model, e.g. timing 1000 training steps on the 1886-cell Mouse Pancreas
  AnnData object.
when_to_use_negative:
- When the goal is clustering/integration quality (ARI against cell type labels) rather
  than runtime — use the full scalability benchmark instead.
- When timing CPU-only methods such as Harmony, Scanorama, Seurat, or LIGER, which
  have no GPU training path to compare.
- When the dataset or mini-batch size is such that fewer than ~6k gradient updates
  are performed — the README warns small datasets may not converge, confounding timing
  with under-training.
tools:
- name: scETM
  role: Model being timed; trained via UnsupervisedTrainer on CPU and GPU to measure
    device speed-up
  repo: https://github.com/hui2000ji/scETM
- name: PyTorch
  role: GPU-enabled backend that must be installed before scETM to enable device speed-up
    comparison
  repo: https://pytorch.org/
- name: anndata
  role: Reads the MousePancreas.h5ad cells-by-genes matrix used as the timing benchmark
    input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:f37a84c412a214969d91bc5e74e082ba72a8fe1808af75320c02f1b9cbe61b1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hui2000ji/scETM
  tools:
  - Harmony
  - LIGER
  - PyTorch
  - Scanorama
  - Seurat
  - anndata
  - scETM
  - scVAE-GM
  - scVI
  - scVI-LD
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cpu-gpu-training-speedup-measurement

## Summary

Quantifies the wall-clock speed-up of GPU-accelerated training of scETM over CPU by timing identical training runs (1000 gradient steps) on the Mouse Pancreas dataset, validating the reported ~13x device speed-up.

## When to use

Use when you need to measure the device speed-up of GPU vs CPU training for a scETM model, e.g. timing 1000 training steps on the 1886-cell Mouse Pancreas AnnData object.

## When NOT to use

- When the goal is clustering/integration quality (ARI against cell type labels) rather than runtime — use the full scalability benchmark instead.
- When timing CPU-only methods such as Harmony, Scanorama, Seurat, or LIGER, which have no GPU training path to compare.
- When the dataset or mini-batch size is such that fewer than ~6k gradient updates are performed — the README warns small datasets may not converge, confounding timing with under-training.

## Inputs

- MousePancreas.h5ad AnnData cells-by-genes matrix (1886 cells) with batch_indices in adata.obs
- GPU-enabled PyTorch installation and a CUDA device
- scETM installed via pip (Python 3.7+)

## Outputs

- Wall-clock training time on CPU for a fixed number of training steps (e.g. 1000 steps)
- Wall-clock training time on GPU for the same steps
- CPU-vs-GPU speed-up ratio (expected ≈13x, ~650 s vs ~50 s)

## How to apply

Install PyTorch with GPU support BEFORE installing scETM, since GPU compute significantly boosts performance. Load the Mouse Pancreas data as an AnnData object (MousePancreas.h5ad, 1886 cells) and initialize scETM with n_vars from mp.n_vars, batch count from mp.obs.batch_indices.nunique(), and enable_batch_bias=True. Instantiate an UnsupervisedTrainer and time a fixed number of training steps (e.g. 1000 steps using the same batched gradient descent settings: n_epochs=12000, eval_every=1000, n_samplers=4) first on CPU, then on GPU. Record wall-clock time per run and compute the CPU/GPU ratio; the paper's reference point is approximately 650 s on CPU versus ~50 s on GPU, i.e. a ~13x speed-up. Confirm the GPU path is actually used (GPU-enabled PyTorch visible to the process) before attributing timing differences to the device.

## Related tools

- **scETM** (Model being timed; trained via UnsupervisedTrainer on CPU and GPU to measure device speed-up) — https://github.com/hui2000ji/scETM
- **PyTorch** (GPU-enabled backend that must be installed before scETM to enable device speed-up comparison) — https://pytorch.org/
- **anndata** (Reads the MousePancreas.h5ad cells-by-genes matrix used as the timing benchmark input)

## Examples

```
trainer = UnsupervisedTrainer(model, mp, train_instance_name="MP", ckpt_dir="../results"); trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)
```

## Evaluation signals

- GPU timing for 1000 training steps approximates ~50 s while CPU timing approximates ~650 s, giving a ~13x speed-up as reported
- Both CPU and GPU runs use identical settings (same seed via UnsupervisedTrainer, same mini-batch sampling with n_samplers=4), so only the device differs
- Verify the GPU run actually executes on the device (GPU-enabled PyTorch installed before scETM, per README)
- Wall-clock time is recorded per fixed step count (e.g. 1000 steps), not per epoch, because small datasets fit in one mini-batch so one epoch = one gradient step

## Limitations

- Speed-up depends on installing GPU-enabled PyTorch before scETM; without it the timing comparison is meaningless
- For small datasets (e.g. MousePancreas, 1886 cells) with large mini-batch size (2000), one epoch yields only one gradient update — the README recommends at least 6k gradient updates for convergence, so timing runs on tiny data measure step cost, not converged-model cost
- This measures training speed only; it does not capture memory usage or clustering accuracy (ARI), which belong to the broader scalability benchmark on merged AD+MDD datasets

## Evidence

- [readme] GPU support required before install: "To enable GPU computing (which significantly boosts the performance), please install [PyTorch](https://pytorch.org/) with GPU support **before** installing scETM."
- [methods] Training settings: "trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)"
- [methods] Timing benchmark dataset: "mp = anndata.read_h5ad("MousePancreas.h5ad")"
- [readme] Small-dataset convergence caveat: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model. Otherwise, the model may not have converged when training stops."
