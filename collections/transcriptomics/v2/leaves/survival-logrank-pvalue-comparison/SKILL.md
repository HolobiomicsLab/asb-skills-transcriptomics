---
name: survival-logrank-pvalue-comparison
description: Use when you need to judge which batch correction method preserves biologically
  meaningful (survival-relevant) gene signals, by comparing log-rank survival P values
  of method-specific DEG signatures against TCGA bulk transcriptomes.
when_to_use_negative:
- You have no matched bulk cohort with survival metadata (e.g., TCGA) — the Scissor-style
  scoring and log-rank test cannot be performed
- You only need integration quality metrics like ARI/NMI or batch-effect removal scores
  (e.g., scIB Leiden clustering across resolutions 0.1–2.0) rather than preservation
  of disease-relevant gene signal
- The scRNA-seq batches do not correspond to biological conditions (e.g., tumor vs
  normal), since survival-relevant DEGs require genuine disease-associated differences
  to detect over-correction
tools:
- name: scCobra
  role: Contrastive learning with domain adaptation batch correction whose DEG signatures
    are survival-scored; benchmarked against raw data and other methods
  repo: github.com/mcgilldinglab/scCobra
- name: Seurat
  role: v3.2.3-style integration baseline producing a corrected embedding for DEG
    extraction and survival comparison
- name: Harmony
  role: Batch correction baseline producing a corrected embedding for DEG extraction
    and survival comparison
- name: Scanorama
  role: Batch correction baseline producing a corrected embedding for DEG extraction
    and survival comparison
- name: scVI
  role: scvi-tools VAE-based batch correction baseline (n_batch parameter adjusted)
    producing a corrected embedding for survival comparison
- name: Scanpy
  role: Preprocessing pipeline (QC filtering, normalization to 10,000 counts, log1p,
    2000 HVG selection with vst) feeding all correction methods
- name: Omicverse
  role: Used to identify tumor-vs-normal differentially expressed genes from each
    corrected embedding
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:f5ab58c80dadaba662179f2549c200aa6dbd40514934f69075417a4f0afb9e56
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - Omicverse
  - Scanorama
  - Scanpy
  - Seurat
  - scCobra
  - scVI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# survival-logrank-pvalue-comparison

## Summary

Benchmarks single-cell batch correction methods by whether their batch-corrected DEG signatures stratify patient survival: extract top-ranked tumor-vs-normal DEGs per method, score TCGA bulk transcriptomes with Scissor-style signatures to split patients into high-risk and low-risk groups, and rank methods by log-rank survival P values (replicating the paper's Fig. 3e table).

## When to use

Use when you need to judge which batch correction method best preserves biologically meaningful (survival-relevant) gene signals, by comparing log-rank survival P values of method-specific DEG signatures against TCGA bulk transcriptomes.

## When NOT to use

- You have no matched bulk cohort with survival metadata (e.g., TCGA) — the Scissor-style scoring and log-rank test cannot be performed
- You only need integration quality metrics like ARI/NMI or batch-effect removal scores (e.g., scIB Leiden clustering across resolutions 0.1–2.0) rather than preservation of disease-relevant gene signal
- The scRNA-seq batches do not correspond to biological conditions (e.g., tumor vs normal), since survival-relevant DEGs require genuine disease-associated differences to detect over-correction

## Inputs

- Batch-corrected single-cell embeddings (e.g., .h5ad objects) from each compared method (scCobra, Seurat, Harmony, Scanorama, scVI)
- Preprocessed scRNA-seq data: GSE149614 liver cancer samples QC'd by filtering cells with <200 expressed genes, genes in <3 cells, and mitochondrial fraction >5%, normalized to 10,000 counts with log1p, 2000 highly variable genes (vst)
- TCGA bulk transcriptomes with survival metadata for the same cancer type
- Tumor-vs-normal cell labels for DEG identification

## Outputs

- Per-method DEG rankings from tumor-vs-normal differential expression (Omicverse)
- Signature gene sets of top 75, 50, and 25 ranked genes per correction method
- High-risk vs low-risk patient assignments per signature from Scissor-style TCGA scoring
- survival_pvalues.csv: log-rank survival P values per method and gene-set size (paper's reference values: scCobra 4.1e-21, scVI 1.1e-19, Scanorama 4.4e-14, Seurat 9.6e-11, Harmony 2.4e-9, uncorrected 1.0e-8)

## How to apply

After running each batch-correction method (scCobra, Seurat v3.2.3-style integration, Harmony, Scanorama, scVI via scvi-tools with the n_batch parameter adjusted) on the tumor/normal scRNA-seq samples, identify tumor-vs-normal differentially expressed genes for each corrected embedding with Omicverse. Extract the top 75, 50, and 25 ranked genes per method as signature gene sets. Apply Scissor-style scoring of each signature against TCGA bulk transcriptomes with survival metadata to split patients into high-risk and low-risk groups, then compute log-rank survival P values per method and per gene-set size and assemble them into a comparison table (saved as survival_pvalues.csv). The rationale is that a correction method that over-corrects batch effects erases genuine disease-associated expression differences, which manifests as weaker survival stratification (higher log-rank P values) in the TCGA validation.

## Related tools

- **scCobra** (Contrastive learning with domain adaptation batch correction whose DEG signatures are survival-scored; benchmarked against raw data and other methods) — github.com/mcgilldinglab/scCobra
- **Seurat** (v3.2.3-style integration baseline producing a corrected embedding for DEG extraction and survival comparison)
- **Harmony** (Batch correction baseline producing a corrected embedding for DEG extraction and survival comparison)
- **Scanorama** (Batch correction baseline producing a corrected embedding for DEG extraction and survival comparison)
- **scVI** (scvi-tools VAE-based batch correction baseline (n_batch parameter adjusted) producing a corrected embedding for survival comparison)
- **Scanpy** (Preprocessing pipeline (QC filtering, normalization to 10,000 counts, log1p, 2000 HVG selection with vst) feeding all correction methods)
- **Omicverse** (Used to identify tumor-vs-normal differentially expressed genes from each corrected embedding)

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git; pip install scanpy scib
```

## Evaluation signals

- survival_pvalues.csv contains one log-rank P value per (method, gene-set size) combination, with gene-set sizes exactly 75, 50, and 25
- P value ordering matches the paper's report: scCobra (4.1e-21) < scVI (1.1e-19) < Scanorama (4.4e-14) < Seurat (9.6e-11) < Harmony (2.4e-9) < uncorrected (1.0e-8)
- Each signature splits TCGA patients into two non-degenerate high-risk/low-risk groups before log-rank testing
- The uncorrected-data row is present as a baseline so over-correction (loss of survival signal) is detectable relative to raw data
- Reproducibility of the QC thresholds upstream: cells <200 genes, genes <3 cells, mitochondrial fraction >5%, 2000 HVGs (vst)

## Limitations

- Benchmarking methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, which may not reflect their tuned performance
- The Scissor-style validation depends on TCGA bulk transcriptome quality and the availability of survival metadata for the matched cancer type
- scCobra's label transfer requires that reference and query datasets originate from the same tissue type or contain similar cell types to ensure optimal performance

## Evidence

- [discussion] TCGA-based DEG validation rationale: "Using bulk data from TCGA, this method evaluates the significance of DEGs in distinguishing high-risk and low-risk patient groups"
- [readme] Over-correction rationale: "scCobra mitigates batch effects and minimizes over-correction without gene expression distribution assumptions"
- [discussion] HVG selection upstream of DEG extraction: "we identified 2000 highly variable genes for downstream analysis"
- [discussion] Benchmark methods run with defaults: "we used Seurat, Harmony, Scanorama, scVI, scDML, and scDREAMER as benchmarking methods with default parameters"
