---
name: nmi-clustering-benchmark-evaluation
description: Use when comparing how compactly and purely different single-cell embeddings
  (unimodal vs multimodal) group cells by a categorical label such as pMHC antigen
  specificity, using clustering quality metrics.
when_to_use_negative:
- The goal is antigen-specificity prediction on query data (F1-score atlas-query mapping)
  rather than unsupervised clustering quality evaluation
- The embedding is not yet trained — this skill evaluates embeddings and cannot compensate
  for a poorly trained model (mvTCR results depend on models trained with mvTCR v0.1.3
  + PyTorch v1.8.0)
- Data lack categorical annotations (e.g. pMHC specificity labels) to compute NMI
  against
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: mvTCR
  role: Source of the multimodal TCR+transcriptome embedding being evaluated; installed
    as mvtcr v0.1.3 in a conda environment
  repo: https://github.com/SchubertLab/mvTCR
- name: Scanpy
  role: Builds k-nearest-neighbor graphs over each embedding and runs Leiden clustering
    with a swept resolution parameter
- name: SchubertLab/mvTCR_reproducibility
  role: Repository supplying the pretrained embeddings, preprocessing notebooks, and
    evaluation notebooks used for the NMI clustering comparison
  repo: https://github.com/SchubertLab/mvTCR_reproducibility
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:5bfc83de7e5c6dd29f89e31cc44bed85eb6fa456cc6eaf26b0ba3a328e792c52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - Scanpy
  - SchubertLab/mvTCR_reproducibility
  - mvTCR
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# nmi-clustering-benchmark-evaluation

## Summary

Benchmarks cell embeddings (e.g. TCR-only, RNA-only, Harmony-corrected RNA, mvTCR joint) by Leiden-clustering them over kNN graphs, sweeping resolution to maximize normalized mutual information (NMI) against pMHC antigen-specificity annotations, and validating with per-cluster antigen purity.

## When to use

Use when comparing how compactly and purely different single-cell embeddings (unimodal vs multimodal) group cells by a categorical label such as pMHC antigen specificity, using clustering quality metrics.

## When NOT to use

- The goal is antigen-specificity prediction on query data (F1-score atlas-query mapping) rather than unsupervised clustering quality evaluation
- The embedding is not yet trained — this skill evaluates embeddings and cannot compensate for a poorly trained model (mvTCR results depend on models trained with mvTCR v0.1.3 + PyTorch v1.8.0)
- Data lack categorical annotations (e.g. pMHC specificity labels) to compute NMI against

## Inputs

- Pretrained cell embeddings: TCR-only, RNA-only, Harmony-corrected RNA, and mvTCR joint embeddings for the preprocessed 10x dataset (>60k cells)
- Per-cell pMHC antigen specificity annotations (e.g. GILGFVFTL-specific vs other)

## Outputs

- Per-embedding NMI values between Leiden clusters and specificity annotations
- Per-cluster antigen purity fractions (e.g. GILGFVFTL purity per Leiden cluster)
- nmi_clustering_results.csv with all metrics

## How to apply

Load pretrained cell embeddings (TCR-only, RNA-only, Harmony-corrected RNA, mvTCR) for the preprocessed 10x dataset (>60k cells) from the SchubertLab/mvTCR_reproducibility repository. Build k-nearest-neighbor graphs over each embedding in Scanpy and run Leiden clustering, sweeping the resolution parameter to select the granularity that best separates specificity. Compute NMI between Leiden cluster assignments and pMHC specificity annotations for each embedding variant, then compute per-cluster antigen purity (fraction of cells carrying the dominant specificity, e.g. GILGFVFTL). Compare against reported benchmarks (TCR 0.408, RNA 0.456, Harmony-corrected RNA 0.212, mvTCR 0.535) and write all metrics to nmi_clustering_results.csv. The rationale is that a joint TCR+transcriptome embedding should cluster antigen-specific cells (e.g. 96.5% of GILGFVFTL-specific cells in one mvTCR cluster of 99.2% purity) more compactly than unimodal embeddings.

## Related tools

- **mvTCR** (Source of the multimodal TCR+transcriptome embedding being evaluated; installed as mvtcr v0.1.3 in a conda environment) — https://github.com/SchubertLab/mvTCR
- **Scanpy** (Builds k-nearest-neighbor graphs over each embedding and runs Leiden clustering with a swept resolution parameter)
- **SchubertLab/mvTCR_reproducibility** (Repository supplying the pretrained embeddings, preprocessing notebooks, and evaluation notebooks used for the NMI clustering comparison) — https://github.com/SchubertLab/mvTCR_reproducibility

## Examples

```
conda create --name mvTCR python=3.8.8 -y && conda activate mvTCR && pip install mvtcr==0.1.3
```

## Evaluation signals

- Computed NMI ranking reproduces the reported order: mvTCR highest (0.535), then RNA (0.456), TCR (0.408), and Harmony-corrected RNA lowest (0.212)
- The mvTCR Leiden clustering contains a GILGFVFTL-dominated cluster at 96.5% purity within a 99.2%-purity cluster, versus three RNA clusters for GILGFVFTL-specific cells
- Resolution sweep converges: the selected Leiden resolution is the granularity that best separates specificity (maximizes NMI) per embedding
- All metrics are written to nmi_clustering_results.csv with one NMI value per embedding variant

## Limitations

- Due to reproducibility issues in the underlying library, UMAP visualisation might not be fully recreated; retrained models may diverge from publication models depending on hardware and CUDA version (trained models are provided on Zenodo, doi 10.5281/zenodo.7215447)
- Adjusting the contribution of each modality requires repeated training of mvTCR, ideally with an additional hyperparameter search
- mvTCR was mainly applied to fully paired data (gene expression with α- and β-CDR3); handling of partially missing data remains insufficiently addressed

## Evidence

- [methods] Reported NMI benchmarks: [evidence span withheld — anchor status: not_found]
- [methods] Leiden clustering method: [evidence span withheld — anchor status: not_found]
- [results] mvTCR clustering purity finding: [evidence span withheld — anchor status: not_found]
- [discussion] Integration into Scanpy workflows: "mvTCR’s representation seamlessly integrates into standard single-cell analysis workflows over the Scanpy-format51"
- [readme] Reproducibility setup: [evidence span withheld — anchor status: external_doc]
