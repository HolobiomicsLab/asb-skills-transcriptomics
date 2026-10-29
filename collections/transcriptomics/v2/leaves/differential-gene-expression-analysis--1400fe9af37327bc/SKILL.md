---
name: differential-gene-expression-analysis
description: Use when evaluating a batch-correction method on scRNA-seq data (e.g.
  two healthy samples from the same condition) and you need DEG counts, survival-relevant
  DEG signatures, or a comparison of over-correction vs. batch-noise removal across
  methods.
when_to_use_negative:
- Input data has not been batch-corrected and the goal is integration quality rather
  than biological DEG discovery — DEG counts between same-condition samples will be
  dominated by uncorrected batch effects.
- Reference and query datasets originate from different tissue types or contain dissimilar
  cell types — label transfer assumptions underlying the workflow do not hold.
- The comparison requires methods evaluated with tuned (not default) parameters —
  benchmarking here used default parameters for Seurat, Harmony, Scanorama, scVI,
  scDML, and scDREAMER, so results may not reflect tuned performance.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scanpy
  role: Preprocessing (filtering, normalization to 10,000 counts, log1p) and differential
    expression testing on raw and corrected matrices
- name: scCobra
  role: Batch correction of the input samples prior to DEG-based evaluation
  repo: github.com/mcgilldinglab/scCobra
- name: Harmony
  role: Benchmark batch-correction method whose corrected output is evaluated by DEG
    counts and survival P values
- name: Scanorama
  role: Benchmark batch-correction method evaluated in the same DEG/survival framework
- name: scVI
  role: Benchmark VAE-based batch correction (scvi-tools, n_batch parameter adjusted)
    evaluated in the same framework
- name: Seurat
  role: Benchmark integration method (v3.2.3-style, 2000 HVGs via vst) whose corrected
    output feeds the survival-adjusted DEG analysis
- name: Omicverse
  role: Identification of tumor-vs-normal differentially expressed genes from each
    corrected embedding
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:168fc4878f3fd490d16913c77258010131633eb2f6436c07d15d31bfac5fffe0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - Omicverse
  - Scanorama
  - Seurat
  - scCobra
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# differential-gene-expression-analysis

## Summary

Quantify residual batch noise and preserved biological signal after single-cell batch correction by running differential expression between samples and calling DEGs with defined thresholds (e.g. log2FC > 0.6, adjusted P < 0.05), or by testing DEGs against TCGA survival stratification.

## When to use

Use when evaluating a batch-correction method on scRNA-seq data (e.g. two healthy samples from the same condition) and you need DEG counts, survival-relevant DEG signatures, or a comparison of over-correction vs. batch-noise removal across methods.

## When NOT to use

- Input data has not been batch-corrected and the goal is integration quality rather than biological DEG discovery — DEG counts between same-condition samples will be dominated by uncorrected batch effects.
- Reference and query datasets originate from different tissue types or contain dissimilar cell types — label transfer assumptions underlying the workflow do not hold.
- The comparison requires methods evaluated with tuned (not default) parameters — benchmarking here used default parameters for Seurat, Harmony, Scanorama, scVI, scDML, and scDREAMER, so results may not reflect tuned performance.

## Inputs

- Raw scRNA-seq count matrices for samples to compare (e.g. GSE149614 HCC03N and HCC04N as .h5ad)
- Batch-corrected expression matrices from each method (scCobra, Harmony, Scanorama, scVI, Seurat) as .h5ad
- TCGA bulk transcriptomes with survival metadata (for survival-adjusted DEG evaluation)

## Outputs

- Table of DEG counts per condition/raw/corrected method (e.g. deg_counts_S3.csv)
- Survival log-rank P values per method and gene-set size (e.g. survival_pvalues.csv, replicating Fig. 3e)
- Lists of DEGs per comparison with log2FC > 0.6 and adjusted P < 0.05

## How to apply

After preprocessing with scanpy (filter cells <200 genes, genes <3 cells, mitochondrial fraction >5%; normalize to 10,000 counts with log1p), run the batch-correction methods (scCobra, Harmony, Scanorama, scVI, Seurat) and save each corrected matrix as .h5ad. For batch-noise quantification, run differential expression between two same-condition samples (e.g. HCC03N vs HCC04N from GSE149614) on raw and each corrected matrix, calling DEGs with log2FC > 0.6 and adjusted P < 0.05 — fewer DEGs means better batch-noise removal (scCobra reduced 470 raw DEGs to 30 vs 40 for Harmony and 66 for scVI). For biological-signal preservation, identify tumor-vs-normal DEGs with Omicverse from each corrected embedding, extract top 75/50/25 ranked genes as signature sets, apply Scissor-style scoring against TCGA bulk transcriptomes with survival metadata to split patients into high-/low-risk groups, and compute log-rank P values (scCobra achieved 4.1e-21). Tabulate DEG counts per method (e.g. deg_counts_S3.csv) and survival P values per gene-set size (survival_pvalues.csv) for comparison against expected patterns.

## Related tools

- **scanpy** (Preprocessing (filtering, normalization to 10,000 counts, log1p) and differential expression testing on raw and corrected matrices)
- **scCobra** (Batch correction of the input samples prior to DEG-based evaluation) — github.com/mcgilldinglab/scCobra
- **Harmony** (Benchmark batch-correction method whose corrected output is evaluated by DEG counts and survival P values)
- **Scanorama** (Benchmark batch-correction method evaluated in the same DEG/survival framework)
- **scVI** (Benchmark VAE-based batch correction (scvi-tools, n_batch parameter adjusted) evaluated in the same framework)
- **Seurat** (Benchmark integration method (v3.2.3-style, 2000 HVGs via vst) whose corrected output feeds the survival-adjusted DEG analysis)
- **Omicverse** (Identification of tumor-vs-normal differentially expressed genes from each corrected embedding)

## Examples

```
python run_deg_counts.py --raw raw.h5ad --corrected sccobra.h5ad,harmony.h5ad,scvi.h5ad --thresholds log2fc=0.6,padj=0.05 --out deg_counts_S3.csv
```

## Evaluation signals

- DEG counts between two same-condition samples should drop after effective batch correction (raw 470 → scCobra 30, Harmony 40, scVI 66 in the reference liver cancer experiment)
- Corrected matrices should be saved as .h5ad and DEG calls use consistent thresholds (log2FC > 0.6, adjusted P < 0.05) across all methods
- Survival log-rank P values per method should be comparable to published order (scCobra 4.1e-21 < scVI 1.1e-19 < Scanorama 4.4e-14 < Seurat 9.6e-11 < Harmony 2.4e-9 < uncorrected 1.0e-8)
- deg_counts_S3.csv should reproduce the Supplementary Fig. S3 pattern of DEG counts per condition
- High-risk/low-risk TCGA patient splits from Scissor-style scoring should yield significant separation for biologically preserved signatures

## Limitations

- Over-correction scores derived from two-dimensional UMAP embeddings introduce dependence on UMAP geometry
- Benchmarking methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, which may not reflect their tuned performance
- Label transfer requires reference and query datasets to originate from the same tissue type or contain similar cell types for optimal performance

## Evidence

- [results] DEG thresholds: [evidence span withheld — anchor status: not_found]
- [results] survival ranking: [evidence span withheld — anchor status: not_found]
- [abstract] scCobra batch-noise removal: [evidence span withheld — anchor status: not_found]
- [discussion] default-parameter caveat: "we used Seurat, Harmony, Scanorama, scVI, scDML, and scDREAMER as benchmarking methods with default parameters"
- [discussion] preprocessing thresholds: "we first filtered out cells that expressed fewer than 200 genes"
