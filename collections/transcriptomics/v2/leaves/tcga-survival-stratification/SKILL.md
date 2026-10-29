---
name: tcga-survival-stratification
description: Use when you need to quantify whether gene signatures derived from batch-corrected
  scRNA-seq data (e.g., tumor-vs-normal DEGs) preserve clinically meaningful survival
  signal, as benchmarked across methods like scCobra, Seurat, Harmony, Scanorama,
  and scVI.
when_to_use_negative:
- Your scRNA-seq dataset has no matched bulk cohort with survival metadata (e.g.,
  only the COVID-19 GOBP healthy-vs-disease analysis is needed)
- You are benchmarking integration on metrics like over-correction scores derived
  from 2D UMAP embeddings rather than downstream clinical relevance
- The DEGs are not cancer-relevant (e.g., immune dataset over-correction simulation),
  since the TCGA survival stratification is specific to liver cancer tumor-vs-normal
  signatures
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_2640
tools:
- name: scCobra
  role: Batch correction method whose corrected DEGs are scored for survival signal;
    produced the best log-rank P (4.1e-21)
  repo: https://github.com/mcgilldinglab/scCobra
- name: Seurat
  role: Benchmark integration method (v3.2.3-style) whose DEGs are scored for comparison
- name: Harmony
  role: Benchmark integration method whose DEGs are scored for comparison
- name: Scanorama
  role: Benchmark integration method whose DEGs are scored for comparison
- name: scVI
  role: Benchmark integration method (scvi-tools v0.11, n_batch parameter adjusted)
    whose DEGs are scored for comparison
- name: Scanpy
  role: Preprocessing and HVG selection (2000 HVGs, vst) feeding the corrected embeddings
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:970232bd5d1c45d0c325d4308333015ab987b2647ec26466d1816966a0d88c46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - Harmony
  - Scanorama
  - Scanpy
  - Seurat
  - scCobra
  - scVI
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# tcga-survival-stratification

## Summary

Evaluates whether batch-corrected single-cell DEGs retain survival-relevant signal by scoring top-ranked genes against TCGA bulk transcriptomes with survival metadata and computing log-rank P values for high- vs low-risk patient groups.

## When to use

Use when you need to quantify whether gene signatures derived from batch-corrected scRNA-seq data (e.g., tumor-vs-normal DEGs) preserve clinically meaningful survival signal, as benchmarked across methods like scCobra, Seurat, Harmony, Scanorama, and scVI.

## When NOT to use

- Your scRNA-seq dataset has no matched bulk cohort with survival metadata (e.g., only the COVID-19 GOBP healthy-vs-disease analysis is needed)
- You are benchmarking integration on metrics like over-correction scores derived from 2D UMAP embeddings rather than downstream clinical relevance
- The DEGs are not cancer-relevant (e.g., immune dataset over-correction simulation), since the TCGA survival stratification is specific to liver cancer tumor-vs-normal signatures

## Inputs

- Top-ranked DEG lists (top 75, 50, 25 genes) per batch-correction method from tumor-vs-normal differential expression on corrected embeddings
- TCGA bulk transcriptomes with survival metadata (high-risk/low-risk labels)
- Batch-corrected scRNA-seq embeddings from scCobra, Seurat, Harmony, Scanorama, and scVI plus uncorrected control

## Outputs

- survival_pvalues.csv: log-rank survival P values per batch-correction method and gene-set size (25/50/75 genes)
- High-risk vs low-risk patient group assignments per signature gene set

## How to apply

First identify tumor-vs-normal differentially expressed genes on each batch-corrected embedding (e.g., with Omicverse) and extract the top 75, 50, and 25 ranked genes per method as signature gene sets. Apply Scissor-style scoring of each signature against TCGA bulk transcriptomes carrying survival metadata to split patients into high-risk and low-risk groups. Compute log-rank survival P values per method and gene-set size, assembling a table that replicates Fig. 3e (scCobra 4.1e-21, scVI 1.1e-19, Scanorama 4.4e-14, Seurat 9.6e-11, Harmony 2.4e-9, uncorrected 1.0e-8) and save as survival_pvalues.csv. The rationale is that a smaller survival-adjusted P value indicates the batch-correction method better preserved biologically meaningful gene signal rather than over-correcting it.

## Related tools

- **scCobra** (Batch correction method whose corrected DEGs are scored for survival signal; produced the best log-rank P (4.1e-21)) — https://github.com/mcgilldinglab/scCobra
- **Seurat** (Benchmark integration method (v3.2.3-style) whose DEGs are scored for comparison)
- **Harmony** (Benchmark integration method whose DEGs are scored for comparison)
- **Scanorama** (Benchmark integration method whose DEGs are scored for comparison)
- **scVI** (Benchmark integration method (scvi-tools v0.11, n_batch parameter adjusted) whose DEGs are scored for comparison)
- **Scanpy** (Preprocessing and HVG selection (2000 HVGs, vst) feeding the corrected embeddings)

## Examples

```
sc.tl.rank_genes_groups(adata, groupby='condition', groups=['tumor'], reference='normal', method='wilcoxon'); top75 = adata.uns['rank_genes_groups']['names']['tumor'][:75]
```

## Evaluation signals

- Replicated Fig. 3e ordering: scCobra 4.1e-21 < scVI 1.1e-19 < Scanorama 4.4e-14 < Seurat 9.6e-11 < Harmony 2.4e-9 < uncorrected 1.0e-8
- survival_pvalues.csv contains a log-rank P value for every combination of method (6 including uncorrected) and gene-set size (25, 50, 75)
- Kaplan-Meier curves for high- vs low-risk groups separate visibly for signatures that pass the log-rank test
- Signature gene sets contain exactly the requested top-N gene counts after Omicverse DEG ranking
- Corrected methods should generally yield more significant survival P values than the uncorrected baseline (1.0e-8) if batch correction preserved signal

## Limitations

- Benchmarking methods were run with default parameters, which may not reflect their tuned performance
- The evaluation depends on TCGA bulk survival data being available and matched to the tissue (liver cancer); it cannot be applied to datasets without survival-relevant signatures
- Scissor-style scoring splits patients into risk groups whose interpretation depends on the DEG ranking cutoff chosen (25/50/75 genes)

## Evidence

- [results] workflow step 5-6: [evidence span withheld — anchor status: not_found]
- [results] workflow step 6: [evidence span withheld — anchor status: not_found]
- [discussion] article discussion: "Using bulk data from TCGA, this method evaluates the significance of DEGs in distinguishing high-risk and low-risk patient groups"
- [readme] README install/run: "git clone https://github.com/mcgilldinglab/scCobra.git"
