---
name: k-means-clustering-of-cell-embeddings
description: Use when cells measured across multiple omics modalities (e.g. CITE-seq
  RNA+ADT or RNA+ADT+ATAC) must be grouped into cell types, and per-modality clustering
  quality must be compared against an integrated-multimodality baseline using ARI/NMI.
when_to_use_negative:
- Input is not multimodal (e.g. only one modality is available) — then integration
  cannot be evaluated against unimodal baselines
- The goal is learning embeddings themselves rather than clustering — hyperparameter
  guidance must come from the paper, not the tutorial
- Cells differ across modalities so modalities cannot be arranged as a matched list
  across the same cells
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: snapccess
  role: Python package used to train the variational-autoencoder backbone and generate
    snapshots of multimodal embeddings that k-means is run on
  repo: https://github.com/PYangLab/SnapCCESS
- name: SnapCCESS (R wrapper)
  role: R wrapper that calls the Python package through reticulate for embedding generation
    before clustering
  repo: https://github.com/yulijia/SnapCCESS
- name: Python
  role: Environment in which SnapCCESS is installed via pip and the tutorial workflow
    (k-means clustering, ARI/NMI computation) is executed
- name: remotes
  role: R package used to install the SnapCCESS R wrapper from GitHub
  repo: https://github.com/yulijia/SnapCCESS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:d734b0c93925fecf9e4bdc1a0afc9e565254fad0efe2c023f8828af2003ff307
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - Python
  - R
  - SnapCCESS (R wrapper)
  - remotes
  - snapccess
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# k-means-clustering-of-cell-embeddings

## Summary

Clusters single-cell multimodal omics data by running k-means on SnapCCESS ensemble embeddings (snapshots of variational-autoencoder embeddings of RNA/ADT/ATAC modalities), then scoring clusters with ARI and NMI against known cell types.

## When to use

Use when cells measured across multiple omics modalities (e.g. CITE-seq RNA+ADT or RNA+ADT+ATAC) must be grouped into cell types, and per-modality clustering quality must be compared against an integrated-multimodality baseline using ARI/NMI.

## When NOT to use

- Input is not multimodal (e.g. only one modality is available) — then integration cannot be evaluated against unimodal baselines
- The goal is learning embeddings themselves rather than clustering — hyperparameter guidance must come from the paper, not the tutorial
- Cells differ across modalities so modalities cannot be arranged as a matched list across the same cells

## Inputs

- Multimodal single-cell omics matrices (RNA, ADT, ATAC) measured across the same cells, arranged features-by-cells before preprocessing in the R wrapper
- Trained SnapCCESS ensemble embedding of the full multimodality (epoch=1)
- Unimodality SnapCCESS embeddings (RNA only, ADT only, ATAC only)
- Cell type labels for computing ARI and NMI

## Outputs

- Consensus cell clusters from k-means on ensemble embeddings
- ARI and NMI per clustering run
- Comparison table ari_nmi_modality_comparison.csv with ARI/NMI for multimodal vs. RNA-only vs. ADT-only vs. ATAC-only clusterings

## How to apply

Install the snapccess Python package (pip install snapccess --index-url https://pypi.org/simple) and follow the tutorial notebook an_example_of_generate_embedding_using_SnapCCESS_python_version.ipynb. Load the multimodal dataset (RNA, ADT, ATAC) and train SnapCCESS to create snapshots of embeddings using its variational autoencoder backbone for 1 epoch — the paper's parameters, not the tutorial defaults, apply since the tutorial does not recommend best parameters. Run k-means on the ensemble embedding and record ARI and NMI. Repeat training, k-means, and ARI/NMI computation independently for each unimodality (RNA only, ADT only, ATAC only), then assemble the four results into a comparison table (ari_nmi_modality_comparison.csv). The rationale is that integrated multimodality embeddings generally yield significantly better clustering than any single modality, as reported in Fig. 4a and Supplementary Fig. S4.

## Related tools

- **snapccess** (Python package used to train the variational-autoencoder backbone and generate snapshots of multimodal embeddings that k-means is run on) — https://github.com/PYangLab/SnapCCESS
- **SnapCCESS (R wrapper)** (R wrapper that calls the Python package through reticulate for embedding generation before clustering) — https://github.com/yulijia/SnapCCESS
- **Python** (Environment in which SnapCCESS is installed via pip and the tutorial workflow (k-means clustering, ARI/NMI computation) is executed)
- **remotes** (R package used to install the SnapCCESS R wrapper from GitHub) — https://github.com/yulijia/SnapCCESS

## Examples

```
pip install snapccess --index-url https://pypi.org/simple
```

## Evaluation signals

- Multimodal SnapCCESS embedding (epoch=1) attains higher ARI/NMI than every single-modality embedding, consistent with Fig. 4a / Supplementary Fig. S4
- ari_nmi_modality_comparison.csv contains ARI and NMI for all four conditions (multimodal, RNA, ADT, ATAC)
- Embedding generation succeeds with the paper's parameters (epoch=1) rather than tutorial defaults, since the tutorial does not prescribe best parameters
- Clustering is reproduced independently per modality and results are assembled into the comparison table before claiming integration benefit

## Limitations

- The tutorial does not recommend the best parameters for datasets; for the datasets used in the paper the parameters are listed in the paper (e.g. epoch=1)
- No discussion section content was available in the extracted document; no changelog or body text

## Evidence

- [results] Clustering cells using integrated embeddings of multimodality generally leads to significantly better results than clustering from any unimodality alone, as reported in Fig. 4a and Supplementary Fig. S4.: [evidence span withheld — anchor status: not_found]
- [methods] pip install snapccess  --index-url https://pypi.org/simple: "pip install snapccess  --index-url https://pypi.org/simple"
- [readme] SnapCCESS expects a list of modalities measured across the same cells, for example RNA and ADT matrices from CITE-seq.: [evidence span withheld — anchor status: external_doc]
- [other] This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets.: "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets."
