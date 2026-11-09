---
name: cell-type-biomarker-ranking-extraction
description: Use when a trained scRNA-seq cell-type classifier (e.g. scaLR DNN on
  anndata .h5ad data) must be converted into per-class ranked gene lists, or when
  top-100 biomarkers must be checked for recall against a curated reference like CellMarker2.0.
when_to_use_negative:
- The comparison tool does not expose feature importance — the paper notes features
  cannot be extracted from scBalance, scvi, devCellPy, and CellPLM models, so classwise
  ranking is impossible for those baselines
- The goal is differential expression between clinical conditions (e.g. COVID-19 vs
  normal) rather than class-specific markers — use the scaLR DgePseudoBulk or DgeLMEM
  modules instead
- Input is raw unnormalized counts or non-anndata formats — the pipeline expects .h5ad
  with adata.X containing normalized (log1p, 0-10) gene counts
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scaLR
  role: Provides scalr.feature.selector.classwise_abs / abs_mean to rank classwise
    feature importance from LinearScorer weights and the GeneRecallCurve downstream
    analysis module
  repo: https://github.com/infocusp/scaLR
- name: CellMarker2.0
  role: Curated reference database of cell-type-specific biomarkers from published
    scRNA-seq studies, used as the ground-truth set for recall computation
- name: CellTypist
  role: CellTypist-Logistic baseline whose top-100 biomarkers are extracted and compared
    against scaLR's in the recall analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:b8caced46e30ecb88e7a33bb08d874af1444bd7da3adf6f485e9786374175b94
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/infocusp/scaLR
  repo_url: https://github.com/infocusp/scaLR
  tools:
  - CellMarker2.0
  - CellTypist
  - Python
  - scaLR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cell-type-biomarker-ranking-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract per-cell-type ranked biomarker lists from trained scaLR classifier models using the scalr.feature.selector modules (classwise_abs / abs_mean) over LinearScorer weight matrices, then validate the top-100 markers against CellMarker2.0 reference sets via gene recall curves.

## When to use

Use when a trained scRNA-seq cell-type classifier (e.g. scaLR DNN on anndata .h5ad data) must be converted into per-class ranked gene lists, or when top-100 biomarkers must be checked for recall against a curated reference like CellMarker2.0.

## When NOT to use

- The comparison tool does not expose feature importance — the paper notes features cannot be extracted from scBalance, scvi, devCellPy, and CellPLM models, so classwise ranking is impossible for those baselines
- The goal is differential expression between clinical conditions (e.g. COVID-19 vs normal) rather than class-specific markers — use the scaLR DgePseudoBulk or DgeLMEM modules instead
- Input is raw unnormalized counts or non-anndata formats — the pipeline expects .h5ad with adata.X containing normalized (log1p, 0-10) gene counts

## Inputs

- Trained scaLR model checkpoint / LinearScorer weights matrix (exp_run output from scalr_experiments)
- AnnData .h5ad file with log1p-normalized expression in adata.X (range 0-10), cell_type target column in adata.obs
- CellMarker2.0 curated reference gene list (CSV, e.g. grc_reference_gene.csv) of literature-reported cell-type-specific biomarkers

## Outputs

- Per-cell-type ranked top-100 biomarker list (classwise_abs feature selector output)
- Gene recall curves (one per cell type, sweeping the ranked list against the reference set)
- Overlap/recall counts vs reference markers (e.g. T-cell 21 vs 19; megakaryocyte 10 vs 5)

## How to apply

Run the scaLR pipeline (python pipeline.py with a config_celltype.yaml) so the LinearScorer scoring_config produces a weights matrix from the trained model; the weights matrix is then used to evaluate each feature's contribution toward predicting specific classes. Apply scalr.feature.selector.classwise_abs (or abs_mean) to rank features per class and select the top-K, using k=100 for the per-cell-type biomarker list as in the scaLR analysis config. Feed the ranked classwise gene lists into the GeneRecallCurve downstream step with a reference_genes_path CSV (CellMarker2.0-derived, e.g. grc_reference_gene.csv) and top_K swept (e.g. 300) to generate recall curves per cell type. Judge success by overlap counts: the paper reproduced 21 vs 19 T-cell top-100 biomarkers (of 223 CellMarker2.0 references) and 10 vs 5 megakaryocyte markers (of 39) for scaLR vs CellTypist-Logistic, with gene recall curves preserving scaLR's relative ranking.

## Related tools

- **scaLR** (Provides scalr.feature.selector.classwise_abs / abs_mean to rank classwise feature importance from LinearScorer weights and the GeneRecallCurve downstream analysis module) — https://github.com/infocusp/scaLR
- **CellMarker2.0** (Curated reference database of cell-type-specific biomarkers from published scRNA-seq studies, used as the ground-truth set for recall computation)
- **CellTypist** (CellTypist-Logistic baseline whose top-100 biomarkers are extracted and compared against scaLR's in the recall analysis)

## Examples

```
python pipeline.py --config scaLR/tutorials/pipeline/config_celltype.yaml -l -m  # with GeneRecallCurve (features_selector: ClasswiseAbs) and reference_genes_path: grc_reference_gene.csv
```

## Evaluation signals

- Overlap counts reproduce reported values: 21/223 T-cell top-100 biomarkers for scaLR vs 19 for CellTypist-Logistic; 10/39 megakaryocyte markers vs 5
- Gene recall curves show the same relative ranking between scaLR and CellTypist-Logistic as reported for PBMCs-SLE and PBMCs-C19-F
- Each cell type yields exactly K=100 ranked markers from the classwise_abs selector applied to the LinearScorer weights matrix
- Reference gene CSV paths resolve (e.g. grc_reference_gene.csv) and recall curves are produced per cell type without missing-class errors

## Limitations

- scaLR software and comparison-tool version numbers are not stated, complicating exact reproduction
- Memory usage is inversely proportional to execution time; reducing memory consumption may increase processing time
- Benchmark result tables (Tables 2-4) and supplementary biomarker tables (S10-S14) were not present in the extracted text, so exact per-cell-type recall numbers beyond those cited must be verified against the published tables
- scaLR performance may vary based on the input data format, requiring users to ensure proper data preprocessing

## Evidence

- [readme] Various visualizations, such as ROC curve of class annotation, feature rank plots, heatmap of top genes per class, DGE analysis, and gene recall curves, are generated.: [evidence span withheld — anchor status: not_found]
