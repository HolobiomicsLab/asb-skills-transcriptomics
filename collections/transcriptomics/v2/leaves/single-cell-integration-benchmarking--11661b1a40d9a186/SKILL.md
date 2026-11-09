---
name: single-cell-integration-benchmarking
description: Use when a reference-query single-cell label transfer or integration
  method must be ranked against alternatives on scIB metrics, e.g. over a multi-cancer
  AnnData split where some tools (Seurat-based, >85 batches) may be inapplicable.
when_to_use_negative:
- Methods requiring batch count >85 with Seurat-based implementations are inapplicable
  on this split — do not force-include them in the ranking table
- Do not use when only annotation accuracy (not integration quality/batch removal)
  is the goal — this skill evaluates via the 8-metric scIB benchmark, not accuracy
  alone
- Input datasets whose gene names/order cannot be reconciled between reference and
  query (TOSICA requires identical var_names)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: TOSICA
  role: Transformer-based reference-to-query cell type annotation and batch-insensitive
    integration tool being benchmarked; trained with human_reactome mask
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Loading and manipulating the reference/query AnnData objects and normalizing
    the attention matrix by library-size correction
- name: torch
  role: Deep-learning backend (version 1.7.1) used to train and run the TOSICA Transformer
    model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:9b5159c85859fc4a4f89be183487a6ce33af8cd23ea39dd81ed10233aea8b22f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# single-cell-integration-benchmarking

## Summary

Benchmark TOSICA label-transfer/integration against ~10 comparison methods on a pan-cancer tumor-infiltrating myeloid reference-query split (71,159 cells; KIDNEY/UCEC/ESCA reference vs six-cancer-type query) using 8 scIB metrics, combined as overall = 0.6 × biology conservation + 0.4 × batch removal. Produces a scib_ranking.csv of 11 applicable methods with TOSICA ranked second.

## When to use

Use when a reference-query single-cell label transfer or integration method must be ranked against alternatives on scIB metrics, e.g. over a multi-cancer AnnData split where some tools (Seurat-based, >85 batches) may be inapplicable.

## When NOT to use

- Methods requiring batch count >85 with Seurat-based implementations are inapplicable on this split — do not force-include them in the ranking table
- Do not use when only annotation accuracy (not integration quality/batch removal) is the goal — this skill evaluates via the 8-metric scIB benchmark, not accuracy alone
- Input datasets whose gene names/order cannot be reconciled between reference and query (TOSICA requires identical var_names)

## Inputs

- Reference AnnData: pan-cancer tumor-infiltrating myeloid cells, KIDNEY/UCEC/ESCA (part of 71,159-cell dataset)
- Query AnnData: six-cancer-type myeloid cells with genes reordered as query_adata[:, ref_adata.var_names]
- Pre-prepared REACTOME pathway mask: reactome.gmt (human_reactome)
- Reference-query cell-type label key in ref_adata.obs for TOSICA.train(label_name=...)

## Outputs

- Trained TOSICA model folder: mask.npy, pathway.csv, label_dictionary.csv, model-n.pth
- Query attention matrix (new_adata.X), predicted labels (new_adata.obs['Prediction']), and probabilities (new_adata.obs['Probability'])
- Comparable cell embeddings and annotations from each of the 10 comparison integration methods
- scib_ranking.csv: 8 scIB metric scores plus overall score (0.6 biology + 0.4 batch) for 11 applicable methods, ranked

## How to apply

Load the KIDNEY/UCEC/ESCA pan-cancer myeloid reference and six-cancer-type query AnnData with scanpy (scanpy 1.7.1) and force gene consistency with query_adata[:, ref_adata.var_names], since TOSICA requires identical var_names order. Train TOSICA via TOSICA.train on the reference using the pre-prepared human_reactome mask (reactome.gmt, torch 1.7.1), which saves mask.npy, pathway.csv, label_dictionary.csv, and model-n.pth under a named project folder; then run TOSICA.pre on the query to obtain the attention matrix (new_adata.X), predicted labels, and probabilities. Run all comparison label-transfer/integration methods on the same reference-query split to produce comparable embeddings and annotations, excluding inapplicable ones (e.g. Seurat-based methods for >85 batches). Normalize TOSICA's attention matrix by library-size correction (default size factor 10,000) before metric computation, then compute the 8 scIB benchmark metrics for all 11 applicable methods and rank by overall score = 0.6 × biology conservation + 0.4 × batch removal. Validation: scib_ranking.csv must contain exactly 11 method rows with TOSICA placed second.

## Related tools

- **TOSICA** (Transformer-based reference-to-query cell type annotation and batch-insensitive integration tool being benchmarked; trained with human_reactome mask) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Loading and manipulating the reference/query AnnData objects and normalizing the attention matrix by library-size correction)
- **torch** (Deep-learning backend (version 1.7.1) used to train and run the TOSICA Transformer model)

## Examples

```
TOSICA.train(ref_adata, 'human_reactome', project='reactome_panmyeloid', label_name='celltype'); new_adata = TOSICA.pre(query_adata[:, ref_adata.var_names], model_weight_path='./reactome_panmyeloid/model-n.pth', project='reactome_panmyeloid')
```

## Evaluation signals

- scib_ranking.csv contains exactly 11 method rows (inapplicable methods such as Seurat-based ones for >85 batches excluded)
- TOSICA is placed second in the overall scIB ranking (overall = 0.6 × biology conservation + 0.4 × batch removal)
- query_adata.var_names is identical to and ordered as ref_adata.var_names before prediction
- All 11 methods evaluated on the same reference-query split with the same 8 scIB metrics
- TOSICA outputs present: new_adata.X attention matrix, obs['Prediction'], obs['Probability'], var['pathway_index']

## Limitations

- Seurat-based integration methods are inapplicable when the number of batches exceeds 85, limiting the comparable method set
- Attention matrix must be normalized by library-size correction (default size factor 10,000) before scIB metric computation; raw attention values are not comparable
- TOSICA's accuracy and batch-insensitivity depend on tokens masked by biologically relevant pathways (e.g. reactome.gmt); an inappropriate mask degrades the benchmark

## Evidence

- [other] TOSICA ranked second out of all 11 applicable data integration methods: [evidence span withheld — anchor status: not_found]
- [other] overall score formula: [evidence span withheld — anchor status: not_found]
- [readme] gene ordering requirement: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [discussion] attention matrix normalization: "the matrix is normalized by library-size correction using default size factor 10,000"
- [discussion] batch-insensitive integration via pathway-masked attention: "The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons"
