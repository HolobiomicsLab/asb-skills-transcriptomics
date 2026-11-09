---
name: pmhc-specificity-cluster-purity-analysis
description: Use when comparing embeddings (TCR-only, RNA-only, Harmony-corrected,
  multimodal mvTCR) of paired scRNA/TCR-seq data by how compactly and purely they
  cluster cells sharing a pMHC specificity annotation.
when_to_use_negative:
- Data lacks pMHC specificity annotations — Leiden clusters cannot be scored against
  specificity, so NMI and purity are undefined
- Embeddings have not been derived yet — this skill operates on pretrained/evaluation
  embeddings; run mvTCR training and evaluation first (tutorials/02_training_analysis.ipynb
  or 03_training_prediction.ipynb)
- Goal is predicting specificity for new unannotated cells rather than quantifying
  clustering quality of annotated data — use the mvTCR prediction task instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0091
tools:
- name: mvTCR
  role: Provides the multiview VAE cell-level embedding that fuses TCR and transcriptome;
    pretrained models are loaded to obtain the multimodal embedding being scored
  repo: https://github.com/SchubertLab/mvTCR
- name: Scanpy
  role: Builds k-nearest-neighbor graphs over each embedding and runs Leiden clustering
    with a swept resolution parameter
- name: PyTorch
  role: Deep learning backend required to run mvTCR models (experiments used mvTCR
    v0.1.3 with PyTorch v1.8.0)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:a31bf674ad31547741eba37e504761a7c16346a9834e7d308e44872888381eae
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
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# pmhc-specificity-cluster-purity-analysis

## Summary

Evaluates how well TCR-only, RNA-only, Harmony-corrected RNA, and mvTCR joint embeddings cluster T cells by pMHC antigen specificity, using Leiden clustering with a resolution sweep scored by NMI, plus per-cluster antigen purity (e.g. GILGFVFTL). Benchmarks clustering quality against the published NMI ranking (TCR 0.408, RNA 0.456, Harmony 0.212, mvTCR 0.535).

## When to use

Use when comparing embeddings (TCR-only, RNA-only, Harmony-corrected, multimodal mvTCR) of paired scRNA/TCR-seq data by how compactly and purely they cluster cells sharing a pMHC specificity annotation.

## When NOT to use

- Data lacks pMHC specificity annotations — Leiden clusters cannot be scored against specificity, so NMI and purity are undefined
- Embeddings have not been derived yet — this skill operates on pretrained/evaluation embeddings; run mvTCR training and evaluation first (tutorials/02_training_analysis.ipynb or 03_training_prediction.ipynb)
- Goal is predicting specificity for new unannotated cells rather than quantifying clustering quality of annotated data — use the mvTCR prediction task instead

## Inputs

- Pretrained cell embeddings (TCR-only, RNA-only, Harmony-corrected RNA, mvTCR) for the preprocessed 10x dataset (>60k cells)
- pMHC specificity annotations per cell (e.g. GILGFVFTL-specific vs other)
- Trained mvTCR models from Zenodo (https://doi.org/10.5281/zenodo.7215447) and processed single-cell data from the mvTCR_reproducibility repo

## Outputs

- NMI values between Leiden cluster assignments and pMHC specificity per embedding, written to nmi_clustering_results.csv
- Per-cluster antigen purity metrics for the mvTCR embedding (e.g. GILGFVFTL-dominated cluster at 96.5% specificity within a 99.2%-purity cluster)
- Resolution sweep curves and selected Leiden resolution per embedding

## How to apply

Load pretrained cell embeddings (TCR-only, RNA-only, Harmony-corrected RNA, mvTCR) for the preprocessed 10x dataset (>60k cells) from the SchubertLab/mvTCR_reproducibility repository, which provides evaluation notebooks for deriving the shared embeddings. In Scanpy, build a k-nearest-neighbor graph over each embedding and run Leiden clustering, sweeping the resolution parameter and selecting the granularity that maximizes separation of pMHC specificity. Compute normalized mutual information (NMI) between Leiden cluster assignments and specificity annotations per embedding, and compute per-cluster antigen purity (fraction of cells carrying the dominant specificity, e.g. GILGFVFTL) for the mvTCR embedding. Rationale: mvTCR fuses transcriptome and TCR into a shared cell-level representation, so its embedding should cluster antigen-specific cells more compactly than unimodal alternatives. Write all metrics to nmi_clustering_results.csv and check the ranking against reported benchmarks: mvTCR highest (0.535), RNA 0.456, TCR 0.408, Harmony-corrected RNA lowest (0.212).

## Related tools

- **mvTCR** (Provides the multiview VAE cell-level embedding that fuses TCR and transcriptome; pretrained models are loaded to obtain the multimodal embedding being scored) — https://github.com/SchubertLab/mvTCR
- **Scanpy** (Builds k-nearest-neighbor graphs over each embedding and runs Leiden clustering with a swept resolution parameter)
- **PyTorch** (Deep learning backend required to run mvTCR models (experiments used mvTCR v0.1.3 with PyTorch v1.8.0))

## Examples

```
import scanpy as sc; from sklearn.metrics import normalized_mutual_info_score; sc.pp.neighbors(adata, use_rep='mvTCR'); sc.tl.leiden(adata, resolution=res, key_added='leiden'); nmi = normalized_mutual_info_score(adata.obs['leiden'], adata.obs['specificity'])
```

## Evaluation signals

- Computed NMI values reproduce the reported ranking with mvTCR highest (0.535), RNA 0.456, TCR 0.408, and Harmony-corrected RNA lowest (0.212)
- The mvTCR Leiden clustering contains a GILGFVFTL-dominated cluster where 96.5% of GILGFVFTL-specific cells fall into one cluster at 99.2% purity, versus three RNA-only clusters
- nmi_clustering_results.csv exists and contains one NMI value per embedding variant (TCR-only, RNA-only, Harmony-corrected RNA, mvTCR)
- The Leiden resolution sweep selects the granularity that maximizes NMI against specificity annotations for every embedding, not a fixed default resolution

## Limitations

- Due to differing runtimes, hardware, and CUDA versions, retrained models may diverge from the published ones — use the Zenodo-released trained models (https://doi.org/10.5281/zenodo.7215447) for exact reproduction
- UMAP visualizations may not be fully recreated due to reproducibility issues in the underlying library
- Leiden resolution is optimized to maximize NMI on specificity, which is a supervised granularity choice and may overfit cluster granularity to the specificity label

## Evidence

- [results] NMI benchmark ranking: [evidence span withheld — anchor status: not_found]
- [results] GILGFVFTL purity finding: [evidence span withheld — anchor status: not_found]
- [abstract] Multimodal value over unimodal: "Our analysis demonstrates the added value of multimodal over unimodal approaches to capture antigen specificity"
- [discussion] Integration into Scanpy workflows: "mvTCR’s representation seamlessly integrates into standard single-cell analysis workflows over the Scanpy-format"
- [readme] Reproducibility models and versions: [evidence span withheld — anchor status: external_doc]
