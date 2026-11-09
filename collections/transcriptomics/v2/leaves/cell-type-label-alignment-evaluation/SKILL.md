---
name: cell-type-label-alignment-evaluation
description: Use when you have per-cell embeddings (e.g., tGPT transformer features)
  plus ground-truth cell type annotations and must quantify clustering alignment via
  NMI, ARI, and FMI.
when_to_use_negative:
- No ground-truth cell type annotations are available — external metrics (NMI/ARI/FMI)
  cannot be computed; use internal clustering quality measures instead
- The goal is batch-effect sensitivity assessment or diffusion pseudo-time/trajectory
  inference rather than cluster-vs-label agreement quantification
- Input is already a cluster-assignment table with no per-cell embeddings or gene
  rankings — re-running tGPT feature extraction adds no information
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tGPT
  role: Pretrained transformer (checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64')
    whose embedding-layer and 8 transformer-layer representations are the cell embeddings
    being clustered and evaluated
  repo: https://github.com/deeplearningplus/tGPT
- name: Scanpy
  role: AnnData construction, KNN neighbor graph (sc.pp.neighbors), and Leiden clustering
    (sc.tl.leiden) over the tGPT features
  repo: https://github.com/theislab/scanpy
- name: scikit-learn
  role: Computes the external clustering metrics NMI, ARI, and FMI between the best
    Leiden assignments and ground-truth cell labels
- name: transformers
  role: Loads PreTrainedTokenizerFast and GPT2LMHeadModel (output_hidden_states=True)
    from the Hugging Face checkpoint for feature extraction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:657dcd8e3d690f194074ae66739ff4c2dee315ed5de36bf98ec74065276618af
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deeplearningplus/tGPT
  repo_url: https://github.com/deeplearningplus/tGPT
  tools:
  - NumPy
  - Scanpy
  - pandas
  - scikit-learn
  - tGPT
  - transformers
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cell-type-label-alignment-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how well data-driven clusters align with ground-truth cell type labels by grid-searching Leiden clustering over tGPT-derived cell embeddings and scoring with the external clustering metrics NMI, ARI, and FMI. This reproduces the paper's quantitative clustering table across its six single-cell and bulk evaluation datasets.

## When to use

Use when you have per-cell embeddings (e.g., tGPT transformer features) plus ground-truth cell type annotations and must quantify clustering alignment via NMI, ARI, and FMI.

## When NOT to use

- No ground-truth cell type annotations are available — external metrics (NMI/ARI/FMI) cannot be computed; use internal clustering quality measures instead
- The goal is batch-effect sensitivity assessment or diffusion pseudo-time/trajectory inference rather than cluster-vs-label agreement quantification
- Input is already a cluster-assignment table with no per-cell embeddings or gene rankings — re-running tGPT feature extraction adds no information

## Inputs

- Gene-ranking text files (e.g., Muris_gene_rankings.txt.gz) listing top-expressing gene symbols per cell in descending expression order
- Ground-truth cell type annotation files (e.g., Muris_cell_labels.txt.gz)
- Pretrained tGPT tokenizer and checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' loaded via Hugging Face transformers

## Outputs

- Per-cell mean-pooled tGPT feature matrix (NumPy array)
- Best Leiden cluster assignments per cell after the n_neighbors/resolution grid search
- clustering_metrics.csv — per-dataset table of NMI, ARI, and FMI across the six evaluation datasets

## How to apply

Extract one embedding per cell by mean-pooling tGPT hidden states over non-special tokens (positions 1 through the attention-mask EOS index), using the checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' with output_hidden_states=True, max_length=64, and batch_size=64; the paper extracted representations from the embedding layer and 8 transformer layers. Build a Scanpy AnnData object from the pooled features and compute the KNN neighbor graph with sc.pp.neighbors. Grid-search Leiden parameters over n_neighbors 5-100 (step 5) and resolution 0.1-2 (step 0.2), selecting the combination per dataset that maximizes agreement with the ground-truth labels. Score the best Leiden assignments against the ground-truth cell labels with scikit-learn's external clustering metrics NMI, ARI, and FMI. Aggregate per-dataset results for all six evaluation datasets into clustering_metrics.csv and verify the values fall within the reported ranges (NMI 0.75 on HCA to 0.90 on GTEx; ARI 0.53 on HCL to 0.84 on Tabula Muris; FMI 0.55 on HCL to 0.85 on Tabula Muris).

## Related tools

- **tGPT** (Pretrained transformer (checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64') whose embedding-layer and 8 transformer-layer representations are the cell embeddings being clustered and evaluated) — https://github.com/deeplearningplus/tGPT
- **Scanpy** (AnnData construction, KNN neighbor graph (sc.pp.neighbors), and Leiden clustering (sc.tl.leiden) over the tGPT features) — https://github.com/theislab/scanpy
- **scikit-learn** (Computes the external clustering metrics NMI, ARI, and FMI between the best Leiden assignments and ground-truth cell labels)
- **transformers** (Loads PreTrainedTokenizerFast and GPT2LMHeadModel (output_hidden_states=True) from the Hugging Face checkpoint for feature extraction)

## Examples

```
sc.pp.neighbors(adata,n_neighbors=20); sc.tl.leiden(adata,resolution=0.6)
```

## Evaluation signals

- Per-dataset NMI falls within the reported span 0.75 (HCA) to 0.90 (GTEx)
- ARI falls within 0.53 (HCL) to 0.84 (Tabula Muris) and FMI within 0.55 (HCL) to 0.85 (Tabula Muris)
- Grid search covers n_neighbors 5-100 with step 5 and resolution 0.1-2 with step 0.2, with one best combination selected per dataset
- Feature matrix has exactly one row per cell, each mean-pooled over non-special tokens (positions 1 to the attention-mask EOS index)
- clustering_metrics.csv contains NMI, ARI, and FMI for each of the six evaluation datasets

## Limitations

- tGPT uses only the top-expressing genes, so information specifically represented within the low-expressing genes may be missed
- Rank-based input neglects fold changes among genes, which can affect biological interpretation of the resulting clusters
- The generative pretraining objective (predicting gene rankings) is not directly related to cell clustering, so the last-layer representation is not guaranteed to give the best clustering — compare the embedding layer and all 8 transformer layers rather than assuming the final layer wins

## Evidence
