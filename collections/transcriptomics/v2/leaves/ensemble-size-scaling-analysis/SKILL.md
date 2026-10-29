---
name: ensemble-size-scaling-analysis
description: Use when evaluating snapshot-ensemble embedding generation on multimodal
  single-cell omics (e.g., CITE-seq RNA+ADT) and you need to determine which ensemble
  size and epoch budget reproduces conventional VAE-ensemble clustering performance
  at minimal cost.
when_to_use_negative:
- Input data are not multimodal single-cell omics with matched modalities across the
  same cells (SnapCCESS expects a list of modalities measured across the same cells,
  e.g., RNA and ADT from CITE-seq)
- No ground-truth cell-type labels are available, since ARI/NMI concordance against
  reference labels is the core evaluation metric of this skill
- You are seeking optimal hyperparameters from the package itself — the tutorial explicitly
  does not recommend best parameters for your datasets; the paper must be consulted
  for the published settings
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: snapccess
  role: Python package used to create snapshots of multimodality embeddings with variational
    autoencoders across epoch and ensemble-size settings
  repo: https://github.com/PYangLab/SnapCCESS
- name: SnapCCESS
  role: Framework under evaluation; R wrapper installed via remotes and coupled with
    clustering algorithms to generate consensus clustering
  repo: https://github.com/PYangLab/SnapCCESS
- name: k-means (scikit-learn)
  role: Clustering algorithm coupled with SnapCCESS embeddings to generate consensus
    clusterings scored by ARI/NMI at each ensemble size
- name: CiteFuse spectral clustering
  role: Clustering algorithm coupled with SnapCCESS embeddings for the ensemble-size
    scaling comparison
- name: SIMLR
  role: Clustering algorithm coupled with SnapCCESS embeddings for the ensemble-size
    scaling comparison
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:ea547d6b1b6e5b890d97e8f6ae0c491e45e10eafed34db05e30c7e156bfbf0cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - CiteFuse spectral clustering
  - Python
  - R
  - SIMLR
  - SnapCCESS
  - k-means (scikit-learn)
  - remotes
  - snapccess
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# ensemble-size-scaling-analysis

## Summary

Benchmarking technique for unsupervised ensemble deep learning (SnapCCESS snapshots of multimodal variational autoencoder embeddings) in which clustering concordance (ARI/NMI) is measured as ensemble size is scaled (up to 100) and per-model training budget is varied (1, 2, 10 epochs), to find the smallest configuration that matches conventional VAE ensembles.

## When to use

Use when evaluating snapshot-ensemble embedding generation on multimodal single-cell omics (e.g., CITE-seq RNA+ADT) and you need to determine which ensemble size and epoch budget reproduces conventional VAE-ensemble clustering performance at minimal cost.

## When NOT to use

- Input data are not multimodal single-cell omics with matched modalities across the same cells (SnapCCESS expects a list of modalities measured across the same cells, e.g., RNA and ADT from CITE-seq)
- No ground-truth cell-type labels are available, since ARI/NMI concordance against reference labels is the core evaluation metric of this skill
- You are seeking optimal hyperparameters from the package itself — the tutorial explicitly does not recommend best parameters for your datasets; the paper must be consulted for the published settings

## Inputs

- Six public multimodal single-cell omics datasets (e.g., CITE-seq RNA + ADT matrices, including the Ramaswamy CITE-seq data) with ground-truth cell-type labels
- SnapCCESS snapshot embeddings per modality, generated at ensemble sizes up to 100 and epochs 1/2/10 (snapccess Python package)
- Conventional VAE-ensemble embeddings (one full model per ensemble member) as the baseline

## Outputs

- ari_nmi_ensemble_benchmark.csv: ARI/NMI versus epochs and ensemble size for SnapCCESS vs conventional VAE ensemble
- algorithm_comparison_ari_nmi.csv: ARI/NMI of k-means, spectral clustering, and SIMLR at each ensemble size
- Performance plateau characterization (expected near ensemble size 50), matching Figure 2a / Supplementary Fig. S1 and Figure 4b / Supplementary Fig. S5

## How to apply

Generate SnapCCESS snapshots of multimodality embeddings with variational autoencoders, sweeping per-model training budgets of 1, 2, and 10 epochs and ensemble sizes up to 100 members, using the snapccess Python package and the repository tutorial notebook. For every configuration, couple the embedding set with clustering algorithms (k-means via scikit-learn, CiteFuse spectral clustering, and SIMLR) to generate consensus clusterings, and compute Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) against ground-truth cell-type labels. Run the conventional VAE-ensemble baseline (one full model per member) on the same six multimodal single-cell datasets for the efficiency comparison. Tabulate ARI/NMI versus epochs and ensemble size, looking for the performance plateau around an ensemble size of ~50 that the paper reports (Figure 2a / Supplementary Fig. S1; Figure 4b / Supplementary Fig. S5). Rationale: snapshots are cheaper than training full VAEs per member, so a plateau at small sizes justifies using SnapCCESS at epoch=1, size=50 rather than conventional ensembles that need many epochs.

## Related tools

- **snapccess** (Python package used to create snapshots of multimodality embeddings with variational autoencoders across epoch and ensemble-size settings) — https://github.com/PYangLab/SnapCCESS
- **SnapCCESS** (Framework under evaluation; R wrapper installed via remotes and coupled with clustering algorithms to generate consensus clustering) — https://github.com/PYangLab/SnapCCESS
- **k-means (scikit-learn)** (Clustering algorithm coupled with SnapCCESS embeddings to generate consensus clusterings scored by ARI/NMI at each ensemble size)
- **CiteFuse spectral clustering** (Clustering algorithm coupled with SnapCCESS embeddings for the ensemble-size scaling comparison)
- **SIMLR** (Clustering algorithm coupled with SnapCCESS embeddings for the ensemble-size scaling comparison)

## Examples

```
pip install snapccess --index-url https://pypi.org/simple
```

## Evaluation signals

- SnapCCESS at a single training epoch with ensemble sizes 50 and 100 should reach ARI/NMI comparable to the conventional VAE ensemble, which requires many epochs for comparable results
- ARI/NMI curves should show a clear increase with increasing ensemble size, regardless of clustering algorithm (k-means, spectral, SIMLR), with improvement peaking around ensemble size 50
- SnapCCESS (epoch=1, ensemble size 50) should achieve higher ARI than base ensemble size 1 in both the major and minor partitions of cell types across all six datasets
- Benchmark tables should be written to ari_nmi_ensemble_benchmark.csv and algorithm_comparison_ari_nmi.csv, with values consistent with Figure 2a / Supplementary Fig. S1 and Figure 4b / Supplementary Fig. S5

## Limitations

- The SnapCCESS tutorial only explains how to use the package and does not recommend best parameters for your datasets; parameters for the published datasets are listed in the paper
- The Python package depends on PyTorch, and GPU acceleration is optional and depends on the local PyTorch/CUDA or PyTorch/ROCm installation
- The R wrapper requires a reticulate-managed Python environment (e.g., SnapCCESS::install_SnapCCESS(envname='SnapCCESS', method='conda')) and must be configured before loading

## Evidence

- [intro] workflow/snapshots: "By creating snapshots of embeddings of multimodality using variational autoencoders, SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells."
- [results] ensemble-size scaling: [evidence span withheld — anchor status: not_found]
- [readme] input data format: [evidence span withheld — anchor status: external_doc]
- [readme] hyperparameter guidance: [evidence span withheld — anchor status: external_doc]
- [methods] install command: "pip install snapccess  --index-url https://pypi.org/simple"
