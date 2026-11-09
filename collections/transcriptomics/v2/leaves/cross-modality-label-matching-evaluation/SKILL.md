---
name: cross-modality-label-matching-evaluation
description: Use when you have aligned source/target single-cell latent manifolds
  (e.g. ATAC projected onto RNA space by scTopoGAN) plus cell type labels, and need
  a metric of alignment quality or a stability check across random GAN initializations.
when_to_use_negative:
- Datasets with overlapping features or cells — standard integration methods that
  exploit overlap are sufficient, and this unsupervised alignment evaluation is unnecessary
- Input is raw ATAC/RNA data rather than low-dimensional topoAE latent manifolds —
  the scores must be computed on aligned embeddings produced by run_scTopoGAN, not
  unembedded matrices
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTopoGAN
  role: Aligns the source manifold to the target manifold via a topology-guided GAN,
    producing the aligned embeddings whose label-matching scores are evaluated
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder
  role: Produces the per-modality latent manifolds (get_TopoAE_Embeddings) on which
    alignment quality and topological loss are computed
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Python
  role: Implementation language; the matching-score and Pearson-correlation computations
    are run via the Python scripts in the repository
  repo: https://github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:2406375e1420d9a3d3b6509070225929c6b5f8afb3368283ea899f1d3adec94d
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-modality-label-matching-evaluation

## Summary

Quantifies the quality of an unsupervised cross-modality single-cell alignment by computing Celltype and Subcelltype label-matching scores between the aligned source manifold and the target manifold. It also supports correlating those scores with the topology-guided GAN's topological loss (Eq. 2) to verify that low topological loss tracks better alignment.

## When to use

Use when you have aligned source/target single-cell latent manifolds (e.g. ATAC projected onto RNA space by scTopoGAN) plus cell type labels, and need a metric of alignment quality or a stability check across random GAN initializations.

## When NOT to use

- Datasets with overlapping features or cells — standard integration methods that exploit overlap are sufficient, and this unsupervised alignment evaluation is unnecessary
- Input is raw ATAC/RNA data rather than low-dimensional topoAE latent manifolds — the scores must be computed on aligned embeddings produced by run_scTopoGAN, not unembedded matrices

## Inputs

- Pre-processed PBMC ATAC (source) and RNA (target) topoAE latent manifolds from get_TopoAE_Embeddings
- Celltype and Subcelltype label annotations for source and target cells
- scTopoGAN-aligned source manifold (source data projected into target space) at checkpoint epochs 500-1000

## Outputs

- Celltype matching score trajectory per GAN run
- Subcelltype matching score trajectory per GAN run
- Pearson correlations between topological loss and each matching score, saved to gan_stability_correlations.csv (expected ~ -0.73 Celltype, ~ -0.67 Subcelltype)

## How to apply

Load the pre-processed topoAE latent manifolds for the source (e.g. ATAC) and target (e.g. RNA) modalities from the Data folder, then align them with run_scTopoGAN from scTopoGAN_Functions.py. At checkpoints from epoch 500 onward (500, 600, 700, 800, 900, 1000), compute the topological loss (Eq. 2) between the input source manifold and its output projected onto the target space, and in parallel compute Celltype and Subcelltype label-matching scores between the aligned source and target manifolds. For each run, compute the Pearson correlation between the topological loss trajectory and each label-matching trajectory; repeating this across 40 random initializations yields expected correlations near r = -0.73 (Celltype) and r = -0.67 (Subcelltype). The negative correlation is the key rationale: low topological loss between the source manifold and its projection indicates better label-consistent alignment, so matching scores serve as an independent validation signal for the topology-guided GAN.

## Related tools

- **scTopoGAN** (Aligns the source manifold to the target manifold via a topology-guided GAN, producing the aligned embeddings whose label-matching scores are evaluated) — https://github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder** (Produces the per-modality latent manifolds (get_TopoAE_Embeddings) on which alignment quality and topological loss are computed) — https://github.com/AkashCiel/scTopoGAN
- **Python** (Implementation language; the matching-score and Pearson-correlation computations are run via the Python scripts in the repository) — https://github.com/AkashCiel/scTopoGAN

## Examples

```
python Run_scTopoGAN.py
```

## Evaluation signals

- Pearson correlation between topological loss and Celltype matching across 40 GAN initializations falls near r = -0.73
- Pearson correlation between topological loss and Subcelltype matching falls near r = -0.67
- Matching scores are recorded at all six checkpoint epochs (500, 600, 700, 800, 900, 1000) for every run
- Correlations are written to gan_stability_correlations.csv with one row per run and both Celltype and Subcelltype values
- All 40 runs use identical architecture and the Full PBMC dataset, differing only in random initialization

## Limitations

- Reported correlations (-0.73 / -0.67) are specific to the Full PBMC dataset with 1000-epoch training and 40 initializations; other datasets or training budgets may shift the relationship
- Label matching requires reliable Celltype/Subcelltype annotations, which unsupervised settings may not provide for new datasets
- The relationship is a trend across runs, not a per-run guarantee — individual GAN initializations may deviate from the negative correlation

## Evidence

- [methods] run_scTopoGAN aligns source manifold: "```run_scTopoGAN``` which aligns the source manifold to the target manifold using the topology-guided GAN model"
- [abstract] topoAE latent representations: "We use topological autoencoders (topoAE) to obtain latent representations of each modality separately. A topology-guided Generative Adversarial Network then aligns these latent representations"
- [readme] README main functions: "The main two functions are 1) ```get_TopoAE_Embeddings``` which obtains the low-dimensional manifold of each dataset separately using Topological Autoencoder"
- [readme] README reproduction script: "The ```Run_scTopoGAN.py``` script provides an example for running scTopoGAN on a new data."
