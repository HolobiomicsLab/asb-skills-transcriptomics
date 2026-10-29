---
name: leiden-clustering-of-cell-embeddings
description: Use when you have precomputed cell-level embeddings plus per-cell specificity/clonotype
  annotations and need to quantify how compactly and purely the embedding groups cells
  by specificity, or when comparing embedding variants on a common clustering metric.
when_to_use_negative:
- You lack per-cell specificity (pMHC binding) annotations — NMI and purity cannot
  be computed, so the skill's evaluation step is undefined
- You need a train/test-evaluated specificity classifier rather than an unsupervised
  compactness comparison — use mvTCR's supervised prediction workflow (tutorials/03_training_prediction.ipynb)
  instead
- Input is raw counts rather than precomputed embeddings — first run preprocessing
  and mvTCR model training/evaluation to derive the shared embedding
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mvTCR
  role: Produces the multimodal joint cell embedding whose Leiden clustering is compared
    against unimodal baselines (reported NMI 0.535)
  repo: https://github.com/SchubertLab/mvTCR
- name: Scanpy
  role: Builds the k-nearest-neighbor graphs over each embedding and runs Leiden clustering
    with resolution sweeping; NMI/purity computed on Scanpy-format data
- name: PyTorch
  role: Deep-learning backend required to train/evaluate the mvTCR model that yields
    the embeddings
  repo: https://github.com/SchubertLab/mvTCR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:f40570b19f61e36138cc4bcf9428c3443e6292c0cea453fdfd2fc0796a0275b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - Scanpy
  - mvTCR
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Leiden clustering of cell embeddings with NMI and purity evaluation

## Summary

Cluster precomputed cell embeddings (TCR-only, RNA-only, Harmony-corrected RNA, and mvTCR joint embeddings) with the Leiden algorithm in Scanpy, sweeping the resolution parameter to maximize normalized mutual information (NMI) against pMHC specificity annotations, then assess per-cluster antigen purity. This is the method the mvTCR paper used to demonstrate that multimodal embeddings (mvTCR, NMI 0.535) separate antigen-specific T cells better than unimodal ones (RNA 0.456, TCR 0.408, Harmony-corrected RNA 0.212).

## When to use

Use when you have precomputed cell-level embeddings plus per-cell specificity/clonotype annotations and need to quantify how compactly and purely the embedding groups cells by specificity, or when comparing embedding variants on a common clustering metric.

## When NOT to use

- You lack per-cell specificity (pMHC binding) annotations — NMI and purity cannot be computed, so the skill's evaluation step is undefined
- You need a train/test-evaluated specificity classifier rather than an unsupervised compactness comparison — use mvTCR's supervised prediction workflow (tutorials/03_training_prediction.ipynb) instead
- Input is raw counts rather than precomputed embeddings — first run preprocessing and mvTCR model training/evaluation to derive the shared embedding

## Inputs

- Precomputed cell-level embeddings (latent representations) for each variant: TCR-only, RNA-only, Harmony-corrected RNA, and mvTCR joint embedding (Scanpy-compatible AnnData)
- Per-cell pMHC antigen specificity annotations (e.g. GILGFVFTL-specific vs other)
- Preprocessed 10x dataset (>60k cells) with TCR sequences

## Outputs

- Leiden cluster assignments per cell per embedding at the swept resolution values
- NMI values between cluster assignments and specificity annotations per embedding (nmi_clustering_results.csv)
- Per-cluster antigen purity values (e.g. GILGFVFTL fraction per Leiden cluster)

## How to apply

Load the pretrained embeddings (here: TCR-only, RNA-only, Harmony-corrected RNA, and mvTCR joint embeddings for the preprocessed 10x dataset of >60k cells from SchubertLab/mvTCR_reproducibility). For each embedding, build a k-nearest-neighbor graph in Scanpy and run Leiden clustering, sweeping the Leiden resolution parameter and selecting the granularity that best separates specificity annotations. Compute NMI between Leiden cluster assignments and pMHC specificity labels for each embedding, and compute per-cluster antigen purity (fraction of cells carrying the dominant specificity, e.g. GILGFVFTL). Benchmark NMI values against the reported values (TCR 0.408, RNA 0.456, Harmony-corrected RNA 0.212, mvTCR 0.535) and write results to a metrics file such as nmi_clustering_results.csv; mvTCR's embedding integrates with Scanpy-format workflows so standard neighborhood/clustering calls suffice.

## Related tools

- **mvTCR** (Produces the multimodal joint cell embedding whose Leiden clustering is compared against unimodal baselines (reported NMI 0.535)) — https://github.com/SchubertLab/mvTCR
- **Scanpy** (Builds the k-nearest-neighbor graphs over each embedding and runs Leiden clustering with resolution sweeping; NMI/purity computed on Scanpy-format data)
- **PyTorch** (Deep-learning backend required to train/evaluate the mvTCR model that yields the embeddings) — https://github.com/SchubertLab/mvTCR

## Examples

```
conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr
```

## Evaluation signals

- Computed NMI values reproduce the reported ranking: mvTCR highest (0.535) > RNA (0.456) > TCR (0.408) > Harmony-corrected RNA lowest (0.212)
- The mvTCR Leiden clustering contains a GILGFVFTL-dominated cluster: 96.5% of GILGFVFTL-specific cells in one cluster at 99.2% purity, versus three RNA clusters
- The same resolution sweep and kNN-graph settings are applied identically across all embedding variants so the NMI comparison is fair
- nmi_clustering_results.csv is produced with one NMI value per embedding variant

## Limitations

- Leiden resolution is chosen post hoc to maximize NMI on specificity labels, so the comparison reflects best-case granularity per embedding
- Embeddings depend on the trained models; per the README, different GPUs/CUDA versions can make resulting models diverge from the published ones, so pretrained models from Zenodo (https://doi.org/10.5281/zenodo.7215447) are recommended for exact reproduction
- UMAP visualization of the clustering may not be fully recreated due to reproducibility issues in the underlying library

## Evidence

- [discussion] mvTCR's representation seamlessly integrates into standard single-cell analysis workflows over the Scanpy-format51: "mvTCR’s representation seamlessly integrates into standard single-cell analysis workflows over the Scanpy-format51"
- [methods] 8GB RAM for 10x dataset: "8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)"
- [readme] README evaluation note: [evidence span withheld — anchor status: external_doc]
- [readme] README install command: "conda create --name mvTCR python=3.8.8 -y"
- [readme] Trained models on Zenodo: [evidence span withheld — anchor status: external_doc]
