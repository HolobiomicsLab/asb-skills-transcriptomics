---
name: pearson-correlation-stability-analysis
description: Use when a model has a stochastic component (e.g. random GAN initialization)
  and you must test whether an internal loss reliably predicts external alignment
  quality across repeated runs, via per-run Pearson correlations between loss and
  metric trajectories.
when_to_use_negative:
- You have only a single training run with no repeated random initializations — there
  is no trajectory distribution to correlate across runs.
- Your goal is benchmarking scTopoGAN against UnionCom or MMD-MA rather than assessing
  initialization stability — use the UnionCom.py / manifold_align_mmd_pytorch.py /
  scTopoGAN_results.py scripts instead.
- Your input modalities share overlapping features or cells — the article motivates
  scTopoGAN for non-overlapping single-modality datasets; methods relying on overlaps
  are the established route there.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scTopoGAN
  role: run_scTopoGAN trains the topology-guided GAN whose 40 random-initialization
    runs supply the topological loss and matching-score trajectories to correlate
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder
  role: get_TopoAE_Embeddings produces the low-dimensional source and target manifolds
    that are the input to each GAN run
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Python
  role: implementation language for the 40-run loop, topological loss (Eq. 2) computation,
    label-matching scores, and Pearson correlation calculation
  repo: https://github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:a3409d1b1d21fd3789163f2c8ce51f814c4b477879dbaef859b356c67b1415ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - Python
  - Topological Autoencoder
  - scTopoGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pearson-correlation-stability-analysis

## Summary

Quantifies run-to-run stability of a stochastic manifold-alignment model by correlating, per run, the trajectory of the topological loss (Eq. 2) with Celltype and Subcelltype label-matching score trajectories across repeated random initializations. In the scTopoGAN study this showed GANs with low topological loss align the ATAC source manifold to the RNA target manifold better (Pearson r ≈ −0.73 and −0.67 across 40 runs).

## When to use

Use when a model has a stochastic component (e.g. random GAN initialization) and you must test whether an internal loss reliably predicts external alignment quality across repeated runs, via per-run Pearson correlations between loss and metric trajectories.

## When NOT to use

- You have only a single training run with no repeated random initializations — there is no trajectory distribution to correlate across runs.
- Your goal is benchmarking scTopoGAN against UnionCom or MMD-MA rather than assessing initialization stability — use the UnionCom.py / manifold_align_mmd_pytorch.py / scTopoGAN_results.py scripts instead.
- Your input modalities share overlapping features or cells — the article motivates scTopoGAN for non-overlapping single-modality datasets; methods relying on overlaps are the established route there.

## Inputs

- Pre-processed PBMC ATAC (source) and RNA (target) data from the Data folder
- topoAE latent manifolds of each modality obtained with get_TopoAE_Embeddings
- Per-run checkpoint series of topological loss (Eq. 2) values at epochs 500, 600, 700, 800, 900, 1000
- Per-run checkpoint Celltype and Subcelltype label-matching scores at the same epochs

## Outputs

- Per-run Pearson correlation coefficients between topological loss and Celltype matching trajectories (40 values, expected ≈ −0.73)
- Per-run Pearson correlation coefficients between topological loss and Subcelltype matching trajectories (40 values, expected ≈ −0.67)
- gan_stability_correlations.csv

## How to apply

Load the pre-processed PBMC ATAC (source) and RNA (target) topoAE latent manifolds via get_TopoAE_Embeddings, then run run_scTopoGAN 40 times with different random initializations, training the topology-guided GAN for 1000 epochs per run. From epoch 500 onward, at checkpoints 500–1000 every 100 epochs, compute the topological loss (Eq. 2) between the source manifold and its projected output, and at the same checkpoints compute Celltype and Subcelltype label-matching scores between the aligned source and target manifolds. For each run, compute the Pearson correlation between the topological loss trajectory and the Celltype matching trajectory, and separately with the Subcelltype trajectory — do not pool all checkpoints into one regression. Report the distribution of the 40 per-run correlations (expected ≈ −0.73 for Celltype, ≈ −0.67 for Subcelltype) and save them to gan_stability_correlations.csv. The rationale: a consistently negative correlation across initializations demonstrates that topology preservation, not run luck, drives alignment quality of the GAN.

## Related tools

- **scTopoGAN** (run_scTopoGAN trains the topology-guided GAN whose 40 random-initialization runs supply the topological loss and matching-score trajectories to correlate) — https://github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder** (get_TopoAE_Embeddings produces the low-dimensional source and target manifolds that are the input to each GAN run) — https://github.com/AkashCiel/scTopoGAN
- **Python** (implementation language for the 40-run loop, topological loss (Eq. 2) computation, label-matching scores, and Pearson correlation calculation) — https://github.com/AkashCiel/scTopoGAN

## Examples

```
from scipy.stats import pearsonr; r_celltype = pearsonr(loss_traj, celltype_traj)[0]; r_sub = pearsonr(loss_traj, subcelltype_traj)[0]
```

## Evaluation signals

- All 40 per-run Pearson correlations are negative; the mean is approximately −0.73 for Celltype and −0.67 for Subcelltype matching
- Checkpoints sampled exactly at epochs 500, 600, 700, 800, 900, 1000 for both the loss and the matching-score trajectories (paired epochs)
- Each correlation is computed within a run (loss trajectory vs matching trajectory over ≥6 checkpoints), not across pooled runs
- gan_stability_correlations.csv exists with 40 rows and separate columns for Celltype and Subcelltype correlations
- Low topological loss at a checkpoint coincides with high Celltype/Subcelltype matching at that same checkpoint

## Limitations

- Requires 40 independent GAN runs of 1000 epochs each, so the analysis is compute-intensive and results vary with each random initialization
- Correlations were established on the Full PBMC dataset and epochs ≥500; other datasets or earlier checkpoints may not reproduce r ≈ −0.73/−0.67
- The relationship is correlational (Pearson) between two trajectories per run — it does not by itself prove the topological loss causes better alignment

## Evidence

- [methods] run_scTopoGAN aligns the source manifold to the target manifold using the topology-guided GAN model: "2) ```run_scTopoGAN``` which aligns the source manifold to the target manifold using the topology-guided GAN model."
- [abstract] A topology-guided Generative Adversarial Network then aligns these latent representations into a common space.: "A topology-guided Generative Adversarial Network then aligns these latent representations into a common space."
- [readme] The scTopoGAN_results.py script reproduce the alignment results of scTopoGAN using different datasets.: "2) The ```scTopoGAN_results.py``` script reproduce the alignment results of scTopoGAN using different datasets."
