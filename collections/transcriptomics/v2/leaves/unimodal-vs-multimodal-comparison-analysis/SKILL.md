---
name: unimodal-vs-multimodal-comparison-analysis
description: Use when multimodal single-cell omics data (e.g. CITE-seq RNA+ADT, or
  RNA+ADT+ATAC from the same cells) is available and you must decide whether integrated
  multimodal clustering beats per-modality unimodal clustering.
when_to_use_negative:
- Only one modality is available (e.g. RNA-only scRNA-seq) — no unimodal-vs-multimodal
  comparison can be constructed
- Modalities are measured on different cells rather than the same cells — SnapCCESS
  expects a list of modalities measured across the same cells
- You need dataset-specific optimal hyperparameters from scratch — the tutorial explicitly
  does not recommend best parameters; those must come from the paper
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: snapccess
  role: Python package that trains the variational autoencoder snapshot embeddings
    (epoch=1) on each modality combination before k-means clustering
  repo: https://github.com/PYangLab/SnapCCESS
- name: Python
  role: Execution environment; the tutorial workflow notebook is the Python version
  repo: https://github.com/yulijia/SnapCCESS/tree/main/snapccess-py
- name: remotes
  role: R package used to install the SnapCCESS R wrapper as an alternative interface
    to the same Python backend
  repo: https://github.com/yulijia/SnapCCESS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:784bd7a8abc63a88eddcfebabd4c26b282d6f58cef09fa5200b0de23ffc6833e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - Python
  - R
  - remotes
  - snapccess
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# unimodal-vs-multimodal-comparison-analysis

## Summary

Benchmark whether SnapCCESS (variational-autoencoder snapshot embeddings, epoch=1) trained on all available single-cell modalities (RNA, ADT, ATAC) yields better cell clustering than embeddings learned from each unimodality alone, quantified by ARI and NMI.

## When to use

Use when multimodal single-cell omics data (e.g. CITE-seq RNA+ADT, or RNA+ADT+ATAC from the same cells) is available and you must decide whether integrated multimodal clustering beats per-modality unimodal clustering.

## When NOT to use

- Only one modality is available (e.g. RNA-only scRNA-seq) — no unimodal-vs-multimodal comparison can be constructed
- Modalities are measured on different cells rather than the same cells — SnapCCESS expects a list of modalities measured across the same cells
- You need dataset-specific optimal hyperparameters from scratch — the tutorial explicitly does not recommend best parameters; those must come from the paper

## Inputs

- Multimodal single-cell omics dataset with RNA, ADT, and ATAC modalities measured across the same cells (features-by-cells matrices)
- Tutorial data in tutorials/in/ (small CITE-seq RNA, ADT, and cell type files) as an example input
- SnapCCESS Python tutorial notebook an_example_of_generate_embedding_using_SnapCCESS_python_version.ipynb as reference workflow

## Outputs

- ari_nmi_modality_comparison.csv containing ARI and NMI for the multimodal (all-modality) embedding and each unimodality (RNA-only, ADT-only, ATAC-only) embedding
- SnapCCESS ensemble embedding from the full multimodality (epoch=1) used for k-means consensus clustering

## How to apply

Install snapccess via pip and load the tutorial notebook 'an_example_of_generate_embedding_using_SnapCCESS_python_version.ipynb' as the reference workflow; the tutorial does not prescribe hyperparameters, so paper parameters apply (epoch=1). Load the RNA, ADT, and ATAC matrices measured across the same cells, train SnapCCESS on the full multimodality to create snapshot embeddings with its variational autoencoder backbone for 1 epoch, then run k-means on the ensemble embedding and record ARI and NMI. Independently repeat the epoch=1 SnapCCESS training, k-means clustering, and ARI/NMI computation for each unimodality (RNA only, ADT only, ATAC only). Assemble the four ARI/NMI results into a comparison table (ari_nmi_modality_comparison.csv); correct application is judged by the multimodal embedding attaining higher ARI/NMI than every single-modality embedding, matching the Figure 4a / Supplementary Fig. S4 modality-integration result.

## Related tools

- **snapccess** (Python package that trains the variational autoencoder snapshot embeddings (epoch=1) on each modality combination before k-means clustering) — https://github.com/PYangLab/SnapCCESS
- **Python** (Execution environment; the tutorial workflow notebook is the Python version) — https://github.com/yulijia/SnapCCESS/tree/main/snapccess-py
- **remotes** (R package used to install the SnapCCESS R wrapper as an alternative interface to the same Python backend) — https://github.com/yulijia/SnapCCESS

## Examples

```
pip install snapccess --index-url https://pypi.org/simple
```

## Evaluation signals

- The multimodal SnapCCESS embedding (epoch=1) attains higher ARI and NMI than every single-modality (RNA-only, ADT-only, ATAC-only) embedding
- Comparison table ari_nmi_modality_comparison.csv contains exactly four rows (multimodal, RNA, ADT, ATAC), each with ARI and NMI
- Each SnapCCESS run uses epoch=1, consistent with the published paper's parameters for its datasets
- ARI/NMI results are reproducible across independent repeats of the SnapCCESS snapshot-embedding + k-means procedure, consistent with the ensemble design
- Direction of the result is consistent with Figure 4a and Supplementary Fig. S4 of the paper

## Limitations

- The tutorial only explains how to use the package and does not recommend the best parameters for your datasets; the paper lists the parameters for its datasets (e.g. epoch=1), so hyperparameters must be sourced from the paper
- Tutorials demonstrate package usage; best hyperparameters can vary by dataset
- The README available in this context mentions CITE-seq RNA/ADT tutorial data; ATAC-specific handling should be confirmed against the paper before extending beyond CITE-seq inputs

## Evidence

- [methods] This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets.: "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets."
- [readme] SnapCCESS expects modalities across same cells: [evidence span withheld — anchor status: external_doc]
- [full_text] Workflow finding supporting multimodal advantage: [evidence span withheld — anchor status: not_found]
- [readme] Installation command: "pip install snapccess --index-url https://pypi.org/simple"
- [intro] VAE snapshot approach: "By creating snapshots of embeddings of multimodality using variational autoencoders"
