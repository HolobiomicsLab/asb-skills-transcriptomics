---
name: harmony-batch-corrected-embedding-evaluation
description: Use when comparing how well alternative single-cell embeddings (TCR-only,
  RNA-only, Harmony-corrected RNA, mvTCR joint) preserve pMHC antigen specificity,
  quantified by NMI of Leiden clusters and per-cluster antigen purity.
when_to_use_negative:
- The dataset lacks paired TCR sequences and pMHC specificity annotations, so NMI
  against specificity labels cannot be computed
- The goal is general cell-type clustering or batch correction of RNA alone rather
  than evaluating antigen-specificity preservation across embedding variants
- Working with B-cell or unpaired data — the article notes mvTCR was applied mainly
  to paired data with gene expression plus α- and β-CDR3, and no large-scale B-cell
  specificity benchmark exists
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: mvTCR
  role: Generates the multimodal joint embedding (and unimodal TCR/RNA baselines)
    whose Leiden-cluster NMI against pMHC specificity is evaluated
  repo: https://github.com/SchubertLab/mvTCR
- name: Scanpy
  role: Builds k-nearest-neighbor graphs over each embedding and runs Leiden clustering
    with resolution sweep
- name: PyTorch
  role: Backend for loading and running the pretrained mvTCR models to derive shared
    embeddings
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:d0b7f4bddc56bd94846942888989fef444471f6ddf8f0977f752c1adf11c8cf9
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
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# harmony-batch-corrected-embedding-evaluation

## Summary

Evaluates whether unimodal (TCR-only, RNA-only), batch-corrected (Harmony-corrected RNA), and multimodal (mvTCR) cell embeddings cluster T cells by pMHC antigen specificity, using Leiden clustering with resolution swept to maximize NMI against specificity labels on the 10x dataset (>60k cells).

## When to use

Use when comparing how well alternative single-cell embeddings (TCR-only, RNA-only, Harmony-corrected RNA, mvTCR joint) preserve pMHC antigen specificity, quantified by NMI of Leiden clusters and per-cluster antigen purity.

## When NOT to use

- The dataset lacks paired TCR sequences and pMHC specificity annotations, so NMI against specificity labels cannot be computed
- The goal is general cell-type clustering or batch correction of RNA alone rather than evaluating antigen-specificity preservation across embedding variants
- Working with B-cell or unpaired data — the article notes mvTCR was applied mainly to paired data with gene expression plus α- and β-CDR3, and no large-scale B-cell specificity benchmark exists

## Inputs

- Pretrained TCR-only cell embeddings for the 10x dataset
- Pretrained RNA-only cell embeddings
- Harmony-corrected RNA embeddings
- Pretrained mvTCR joint embeddings
- pMHC specificity annotations (e.g. GILGFVFTL-specific vs bystander) per cell

## Outputs

- NMI values per embedding variant (expected: TCR 0.408, RNA 0.456, Harmony-corrected RNA 0.212, mvTCR 0.535)
- Per-cluster antigen purity metrics (e.g. GILGFVFTL cluster at 99.2% purity containing 96.5% of GILGFVFTL-specific cells)
- nmi_clustering_results.csv with all computed metrics

## How to apply

Load pretrained cell embeddings for TCR-only, RNA-only, Harmony-corrected RNA, and mvTCR variants from the SchubertLab/mvTCR_reproducibility repository for the preprocessed 10x dataset (>60k cells). Build a k-nearest-neighbor graph over each embedding in Scanpy and run Leiden clustering, sweeping the resolution parameter to select the granularity that best separates specificity. Compute normalized mutual information (NMI) between Leiden cluster assignments and pMHC specificity annotations for each embedding variant, and compute per-cluster antigen purity (fraction of cells carrying the dominant specificity, e.g. GILGFVFTL). Benchmark against the reported values (TCR 0.408, RNA 0.456, Harmony-corrected RNA 0.212, mvTCR 0.535) and verify that mvTCR contains a GILGFVFTL-dominated cluster (96.5% of GILGFVFTL-specific cells in one cluster of 99.2% purity) versus three RNA-only clusters. The lower NMI for Harmony-corrected RNA demonstrates that batch correction without TCR information can degrade specificity structure.

## Related tools

- **mvTCR** (Generates the multimodal joint embedding (and unimodal TCR/RNA baselines) whose Leiden-cluster NMI against pMHC specificity is evaluated) — https://github.com/SchubertLab/mvTCR
- **Scanpy** (Builds k-nearest-neighbor graphs over each embedding and runs Leiden clustering with resolution sweep)
- **PyTorch** (Backend for loading and running the pretrained mvTCR models to derive shared embeddings)

## Examples

```
conda activate mvTCR && pip install mvtcr==0.1.3; # then in Scanpy: sc.pp.neighbors(adata, use_rep='mvtcr'); sc.tl.leiden(adata, resolution=res); sklearn.metrics.normalized_mutual_info_score(adata.obs['leiden'], adata.obs['specificity'])
```

## Evaluation signals

- Computed NMI ranking reproduces the report: mvTCR highest (0.535), RNA (0.456), TCR (0.408), Harmony-corrected RNA lowest (0.212)
- mvTCR Leiden clustering contains a GILGFVFTL-dominated cluster: 96.5% of GILGFVFTL-specific cells in one cluster at 99.2% purity
- RNA-only embedding splits GILGFVFTL-specific cells across three clusters, unlike the single compact mvTCR cluster
- All metrics are written to nmi_clustering_results.csv with one row per embedding variant

## Limitations

- Pretrained models may diverge from publication models due to differing compute clusters, hardware, and CUDA versions; download reference models from Zenodo (https://doi.org/10.5281/zenodo.7215447) for exact reproduction
- UMAP visualisation underlying the figures may not be fully recreated due to reproducibility issues in the underlying library
- Adjusting the contribution of each modality requires repeated retraining of mvTCR, ideally with an additional hyperparameter search

## Evidence

- [other] Leiden NMI sweep with resolution optimized to maximize NMI: [evidence span withheld — anchor status: not_found]
- [other] GILGFVFTL cluster purity in mvTCR vs RNA embeddings: [evidence span withheld — anchor status: not_found]
- [abstract] Multimodal beats unimodal for antigen specificity: "Our analysis demonstrates the added value of multimodal over unimodal approaches to capture antigen specificity"
- [discussion] Scanpy workflow integration: "mvTCR’s representation seamlessly integrates into standard single-cell analysis workflows over the Scanpy-format51"
- [readme] Trained models provided on Zenodo: [evidence span withheld — anchor status: external_doc]
