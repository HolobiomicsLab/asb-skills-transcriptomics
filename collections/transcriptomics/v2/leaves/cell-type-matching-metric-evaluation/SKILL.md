---
name: cell-type-matching-metric-evaluation
description: Use when comparing cross-species cell type matching pipelines (e.g.,
  human vs mouse vs marmoset primary motor cortex scRNA-seq) and you need reproducible
  ADS/recall metrics, including gene-matching ablations that test whether ProtBERT
  gene matches outperform BLAST matches.
when_to_use_negative:
- The analysis is within-species only — the skill's ADS/recall evaluation is defined
  for cross-species cell type matching comparisons (human–mouse–marmoset motor cortex).
- A final metrics table (e.g., gene_matching_ablation_metrics.csv) already exists
  and the goal is re-analysis rather than recomputation.
- The goal is protein/gene sequence similarity computation itself (BLAST or embed_proteins/calc_dist)
  rather than evaluating downstream cell type matching quality.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: TACTiCS
  role: Cross-species cell type matching model whose matched/aligned cell types are
    scored with ADS and recall under each gene-matching condition
  repo: https://github.com/kbiharie/TACTiCS
- name: SAMap
  role: State-of-the-art baseline method re-run with ProtBERT-derived gene matches
    to provide comparison ADS/recall values
- name: BLAST
  role: Source of the many-to-many and one-to-one gene similarity distance matrices
    used in the ablation conditions
- name: PyTorch
  role: Framework required to train and evaluate the TACTiCS class for each condition
- name: transformers
  role: Package used to embed protein sequences with ProtBERT, generating the gene
    matches supplied to SAMap
- name: Seurat
  role: Additional baseline that TACTiCS is evaluated against in performance comparisons
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:2a86bc14301bb0199b24854fd5b8d113fe7d29b5a4073bbe6b2f927109681f93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - BLAST
  - PyTorch
  - SAMap
  - Seurat
  - TACTiCS
  - transformers
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cell-type-matching-metric-evaluation

## Summary

Quantify how well cell types are matched and aligned across species by computing average dimensional score (ADS) and recall for cross-species comparison methods (e.g., TACTiCS vs SAMap), optionally under ablated gene-matching conditions (ProtBERT-derived vs BLAST-derived gene matches).

## When to use

Use when comparing cross-species cell type matching pipelines (e.g., human vs mouse vs marmoset primary motor cortex scRNA-seq) and you need reproducible ADS/recall metrics, including gene-matching ablations that test whether ProtBERT gene matches outperform BLAST matches.

## When NOT to use

- The analysis is within-species only — the skill's ADS/recall evaluation is defined for cross-species cell type matching comparisons (human–mouse–marmoset motor cortex).
- A final metrics table (e.g., gene_matching_ablation_metrics.csv) already exists and the goal is re-analysis rather than recomputation.
- The goal is protein/gene sequence similarity computation itself (BLAST or embed_proteins/calc_dist) rather than evaluating downstream cell type matching quality.

## Inputs

- Two scRNA-seq count matrices (e.g., 50k-cell subsets of human and mouse primary motor cortex raw count data from Zenodo 10.5281/zenodo.7582460)
- Gene similarity distance matrix per condition (ProtBERT-embedding-derived via calc_dist, or BLAST many-to-many / BLAST one-to-one)
- Trained TACTiCS model and cell embeddings (saved via TACTiCS.save)
- SAMap gene matches derived from ProtBERT protein-sequence embeddings

## Outputs

- ADS and recall values per method and per gene-matching condition (gene_matching_ablation_metrics.csv)
- Saved preprocessed data, trained models, and cell embeddings per condition

## How to apply

Train and evaluate each method on two scRNA-seq datasets plus a gene similarity distance matrix — using the `TACTiCS` class in `tactics.py` (PyTorch) for the transfer-learning model, and SAMap with ProtBERT-derived gene matches from the transformers package for the state-of-the-art baseline. For the gene-matching ablation, construct BLAST many-to-many and BLAST one-to-one distance matrices as TACTiCS inputs in place of the ProtBERT embedding-based matrix. For each condition (TACTiCS BLAST many-to-many, TACTiCS BLAST one-to-one, SAMap ProtBERT), compute average dimensional score (ADS) and recall on the cross-species comparison (e.g., human–mouse) and write all per-condition values to a single metrics table such as gene_matching_ablation_metrics.csv. Judge correctness by reproduction of the reported effects: TACTiCS should outperform both Seurat and SAMap, ProtBERT gene matches should beat BLAST within TACTiCS (substantial ADS/recall decrease under BLAST), and SAMap performance should be near baseline except a recall drop from 0.8 to 0.73 with ProtBERT matches.

## Related tools

- **TACTiCS** (Cross-species cell type matching model whose matched/aligned cell types are scored with ADS and recall under each gene-matching condition) — https://github.com/kbiharie/TACTiCS
- **SAMap** (State-of-the-art baseline method re-run with ProtBERT-derived gene matches to provide comparison ADS/recall values)
- **BLAST** (Source of the many-to-many and one-to-one gene similarity distance matrices used in the ablation conditions)
- **PyTorch** (Framework required to train and evaluate the TACTiCS class for each condition)
- **transformers** (Package used to embed protein sequences with ProtBERT, generating the gene matches supplied to SAMap)
- **Seurat** (Additional baseline that TACTiCS is evaluated against in performance comparisons)

## Examples

```
import pandas as pd; metrics = pd.read_csv('gene_matching_ablation_metrics.csv')
```

## Evaluation signals

- TACTiCS ADS and recall decrease substantially under BLAST many-to-many gene matches relative to ProtBERT matches at the cross-species resolution across all comparisons
- SAMap ADS is similar to baseline, with recall dropping from 0.8 to 0.73 when ProtBERT matches are used
- TACTiCS outperforms both Seurat and SAMap on the matched human/mouse/marmoset motor cortex datasets
- Mixed one-to-one BLAST effects are observed across species pairs (not uniformly negative like many-to-many)
- All per-condition ADS and recall values are recorded in gene_matching_ablation_metrics.csv, and models/cell embeddings were persisted via TACTiCS.save

## Limitations

- Using BLAST gene matches instead of ProtBERT matches decreases TACTiCS's ADS and recall substantially at the cross-species resolution, so BLAST-based ablation results underestimate the method's achievable performance
- Metrics were reported for 50k-cell subsets of the motor cortex data; full unprocessed datasets (Zenodo 10.5281/zenodo.6786357) may give different values
- The transformers package is only needed for embedding protein sequences, so it can be omitted only for BLAST-condition evaluations

## Evidence

- [abstract] outperforms Seurat and SAMap: "our model outperforms Seurat and the state-of-the-art method SAMap"
- [abstract] ProtBERT gene matching beats BLAST: "our gene matching method results in better cell type matches than BLAST in our model"
- [methods] TACTiCS inputs and class: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [methods] save artifacts: "The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`."
- [readme] data availability: "the protein sequences, protein embeddings, raw count data and trained models for 50k subsets of the data can be found on [Zenodo]"
