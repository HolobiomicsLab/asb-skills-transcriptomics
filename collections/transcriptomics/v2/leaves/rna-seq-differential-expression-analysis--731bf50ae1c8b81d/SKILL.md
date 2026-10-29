---
name: rna-seq-differential-expression-analysis
description: Use when a raw RNA-seq counts matrix (.csv, Ensembl ID rows, sample ID
  columns) must be tested for differentially expressed genes between sample groups,
  or when edgeR/limma/DESeq2 runtimes must be benchmarked on identical input.
when_to_use_negative:
- Input is already a TPM matrix or GSVA score matrix — ARMT normalizes counts to log2(TPM+1)
  for GSVA, but DEG analysis needs the raw counts matrix
- Differential analysis of GSVA scores or gene mutation (.maf) data — ARMT routes
  these through separate workflows, not the counts-based DEG path
- A single quick DEG call is needed and runtime on a large TCGA cohort is critical
  — DESeq2 is significantly slower than edgeR and limma on the same input
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: edgeR
  role: Primary DEG detection package employed by ARMT 'for high efficiency', with
    TMM as one of four offered normalization options (TMM, TMMwsp, RLE, upperquartile);
    fastest of the three benchmarked tools
- name: limma
  role: Differential analysis tool run on the identical counts matrix for the runtime
    comparison; among the Bioconductor packages ARMT depends on
- name: DESeq2
  role: Differential analysis tool run on the identical counts matrix for the runtime
    comparison; benchmarked as significantly slower than edgeR and limma (Fig. 1B)
- name: ARMT
  role: Shiny-based GUI (run via ARMT::run_app()) that wraps the DEG analysis on counts
    matrices and provides filtering by p-value, logFC and FDR plus volcano plots
  repo: https://github.com/Dulab2020/ARMT
- name: R
  role: Runtime environment (>=4.0.2) in which edgeR, limma and DESeq2 differential
    analyses are executed and wall-clock times recorded
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:b4289ecc01697965eb85dea8b14414f2df61fa7adcd67db347d03755c3f92976
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Dulab2020/ARMT
  repo_url: https://github.com/Dulab2020/ARMT
  tools:
  - ARMT
  - DESeq2
  - R
  - edgeR
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# rna-seq-differential-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Differential expression analysis of RNA-seq count matrices (Ensembl gene IDs x samples, e.g. TCGA LUAD raw counts) using the R packages edgeR, limma and DESeq2 that the ARMT paper integrates, including side-by-side runtime comparison of the three tools on an identical counts matrix.

## When to use

Use when a raw RNA-seq counts matrix (.csv, Ensembl ID rows, sample ID columns) must be tested for differentially expressed genes between sample groups, or when edgeR/limma/DESeq2 runtimes must be benchmarked on identical input.

## When NOT to use

- Input is already a TPM matrix or GSVA score matrix — ARMT normalizes counts to log2(TPM+1) for GSVA, but DEG analysis needs the raw counts matrix
- Differential analysis of GSVA scores or gene mutation (.maf) data — ARMT routes these through separate workflows, not the counts-based DEG path
- A single quick DEG call is needed and runtime on a large TCGA cohort is critical — DESeq2 is significantly slower than edgeR and limma on the same input

## Inputs

- Raw counts matrix as .csv with gene Ensembl IDs as row names and sample IDs as column names (e.g. TCGA LUAD counts)
- Sample grouping factor (two values for experiment vs. control, or a numeric factor split high/low with low as control)
- Optional .gmt geneset or clinical/TPM data for downstream ARMT integration

## Outputs

- DEG result table per tool (gene, logFC, p-value, FDR/adj.p) filterable by p-value, logFC and FDR thresholds
- Volcano plot visualization of DEGs
- runtime_benchmark.csv with one row per tool (edgeR, limma, DESeq2) and a positive numeric runtime column

## How to apply

Load the counts matrix with gene Ensembl IDs as rows and samples as columns (the format ARMT requires for DEG analysis), define the group factor (e.g. TCGA LUAD tumor vs. normal or a high/low split of a continuous factor, with the low part as control), and run edgeR with default TMM normalization, limma, and DESeq2 on the same matrix under identical settings. Record wall-clock time per tool: the ARMT paper reports DESeq2's computation speed is significantly slower than edgeR and limma (Fig. 1B), so expect this ordering on a large cohort. Filter DEG results by p-value, logFC and FDR (adj.p) as ARMT allows, and visualize with volcano plots. Note that different statistical principles across the three tools may yield different DEG sets, so tool choice is a reported decision point, not a neutral substitution.

## Related tools

- **edgeR** (Primary DEG detection package employed by ARMT 'for high efficiency', with TMM as one of four offered normalization options (TMM, TMMwsp, RLE, upperquartile); fastest of the three benchmarked tools)
- **limma** (Differential analysis tool run on the identical counts matrix for the runtime comparison; among the Bioconductor packages ARMT depends on)
- **DESeq2** (Differential analysis tool run on the identical counts matrix for the runtime comparison; benchmarked as significantly slower than edgeR and limma (Fig. 1B))
- **ARMT** (Shiny-based GUI (run via ARMT::run_app()) that wraps the DEG analysis on counts matrices and provides filtering by p-value, logFC and FDR plus volcano plots) — https://github.com/Dulab2020/ARMT
- **R** (Runtime environment (>=4.0.2) in which edgeR, limma and DESeq2 differential analyses are executed and wall-clock times recorded)

## Examples

```
Rscript -e 'library(edgeR); counts<-read.csv("luad_counts.csv",row.names=1); y<-DGEList(counts=counts); y<-calcNormFactors(y,method="TMM"); t0<-proc.time(); fit<-glmQLFit(estimateDisp(y)); print(proc.time()-t0)'
```

## Evaluation signals

- runtime_benchmark.csv exists and contains exactly three rows (edgeR, limma, DESeq2) with positive numeric runtime values
- DESeq2 runtime is the largest of the three, reproducing the Fig. 1B ordering that DESeq2 is significantly slower than edgeR and limma
- All three tools ran on the identical counts matrix and identical sample grouping (same LUAD matrix, same normalization settings where applicable)
- DEG output table exposes p-value, logFC and FDR (adj.p) columns that can be screened as ARMT allows
- Counts matrix used as input has gene Ensembl IDs as row names and sample IDs as column names, per ARMT's input format

## Limitations

- No software version numbers for edgeR, limma or DESeq2 are stated in the article or README, so exact reproducibility of runtimes across versions is not guaranteed
- Based on different statistical principles, different tools may lead to different DEG results, so runtimes are comparable but DEG sets are not expected to be identical
- Runtime ordering was demonstrated on TCGA LUAD data; behavior on very small cohorts or other organisms may differ

## Evidence
