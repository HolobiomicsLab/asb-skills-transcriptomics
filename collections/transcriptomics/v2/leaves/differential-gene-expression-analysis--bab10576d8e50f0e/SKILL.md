---
name: differential-gene-expression-analysis
description: Use when a normalized gene/ortholog expression matrix with sample metadata
  is available and the goal is a ranked table of DEGs at a defined FDR threshold (e.g.,
  FDR < 0.05 with |log2FC| > 1.5, or FDR < 0.1 for salamander-style analyses).
when_to_use_negative:
- Input is already a gene list or pathway membership set rather than an expression
  matrix — use the enrichment module instead of the DE module
- The count table has not been normalized and QC'd — run box plot/PCA QC and PerformExpressNormalization
  (variance filter 15, abundance filter 4) before DE
- Multiple datasets must be jointly analyzed for batch effects — switch to the meta-analysis
  branch (SetAnalType("metadata"), PerformDEAnalMeta, batch correction via PlotMetaPCA)
  instead of single-table limma
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ExpressAnalyst
  role: 'Web platform hosting the DE workflow: metadata selection, design matrix construction,
    limma DE analysis, FDR adjustment, and QC PCA plotting'
  repo: https://www.expressanalyst.ca
- name: ExpressAnalystR
  role: R package replicating the web server's DE pipeline locally via Init.Data,
    ReadTabExpressData, SetSelectedMetaInfo, SetupDesignMatrix, and PerformDEAnal
  repo: https://github.com/xia-lab/ExpressAnalystR
- name: limma
  role: Statistical engine for voom log2 CPM normalization, design matrix with blocking
    factors (e.g., species), custom contrasts, and FDR-adjusted p-values
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:4a919e735ee0ea4a53c749fa58a185b7c8d77c13fb3c27ab22be03eea30e98a9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - ExpressAnalystR
  - Seq2Fun
  - Seq2Fun 2.0
  - limma
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# differential-gene-expression-analysis

## Summary

Identify differentially expressed genes from a normalized RNA-seq or microarray count table using limma (with voom log2 CPM normalization for RNA-seq) as implemented in ExpressAnalyst/ExpressAnalystR, supporting blocking factors, custom contrasts, and FDR-adjusted thresholds matched to the study design.

## When to use

Use when a normalized gene/ortholog expression matrix with sample metadata is available and the goal is a ranked table of DEGs at a defined FDR threshold (e.g., FDR < 0.05 with |log2FC| > 1.5, or FDR < 0.1 for salamander-style analyses).

## When NOT to use

- Input is already a gene list or pathway membership set rather than an expression matrix — use the enrichment module instead of the DE module
- The count table has not been normalized and QC'd — run box plot/PCA QC and PerformExpressNormalization (variance filter 15, abundance filter 4) before DE
- Multiple datasets must be jointly analyzed for batch effects — switch to the meta-analysis branch (SetAnalType("metadata"), PerformDEAnalMeta, batch correction via PlotMetaPCA) instead of single-table limma

## Inputs

- Gene/ortholog expression count table (tab-delimited, e.g., estrogen.txt format) with samples as columns
- Sample metadata defining primary factor, secondary factor, and optional blocking factor (e.g., time and species)
- RNA-seq count matrices from Kallisto (v0.46.1) or Seq2Fun (v2.0.2) quantification of FASTQ files
- DEG threshold definition (FDR cutoff and optional log2FC cutoff)

## Outputs

- Differential expression results table per gene (logFC, p-value, FDR-adjusted p-value), e.g., salamander_de_results.csv
- DEG count per treatment contrast (e.g., 2780 DEGs at FDR < 0.1)
- Relative DEG counts per contrast for cross-quantification comparison (Kallisto vs Seq2Fun)
- qs dataset object retrievable via readDataset(fileName) containing dataSet$comp.res

## How to apply

Initialize ExpressAnalystR in local mode with Init.Data(FALSE) and SetAnalType("onedata"), read the count table with ReadTabExpressData, then normalize RNA-seq counts with limma voom (log2 CPM), applying a variance filter of 15 and an abundance filter of 4 as used in the paper. Select the primary metadata factor (e.g., time) and, where the design confounds conditions, set a blocking factor (e.g., species) with SetSelectedMetaInfo, then build the limma design matrix with SetupDesignMatrix("limma"). Run PerformDEAnal with a custom contrast (e.g., time0 vs time24 or absent vs. present); fold changes are log2 transformed and p-values are FDR (BH) adjusted. Define DEGs by the study-appropriate cutoff — FDR-adjusted p < 0.05 plus |log2FC| > 1.5 for the zebrafish/lobster equivalence analysis, or FDR-adjusted p < 0.1 with no log2FC cut-off to reproduce the salamander analysis that yielded 2780 DEGs from 17 of 18 samples (excluding outlier SRR7499350). For paired-end comparison workflows, the same DE table feeds relative DEG-count comparisons across Kallisto vs. Seq2Fun 2.0.2 quantification (mismatches=2, min matching length=19, min BLOSUM62 score=80).

## Related tools

- **ExpressAnalyst** (Web platform hosting the DE workflow: metadata selection, design matrix construction, limma DE analysis, FDR adjustment, and QC PCA plotting) — https://www.expressanalyst.ca
- **ExpressAnalystR** (R package replicating the web server's DE pipeline locally via Init.Data, ReadTabExpressData, SetSelectedMetaInfo, SetupDesignMatrix, and PerformDEAnal) — https://github.com/xia-lab/ExpressAnalystR
- **limma** (Statistical engine for voom log2 CPM normalization, design matrix with blocking factors (e.g., species), custom contrasts, and FDR-adjusted p-values)

## Examples

```
dataSets <- SetSelectedMetaInfo("estrogen.txt","ER", "NA", F); dataSets <- SetupDesignMatrix("estrogen.txt", "limma"); dataSets <- PerformDEAnal("estrogen.txt", "custom", "absent vs. present", "NA", "intonly")
```

## Evaluation signals

- Salamander benchmark: DEG count at FDR < 0.1 (no log2FC cut-off) matches the 2780 reported when 17 of 18 samples (excluding SRR7499350) are analyzed with species as blocking factor
- Zebrafish/lobster benchmark: relative DEG counts per treatment contrast at FDR < 0.05 and |log2FC| > 1.5 are almost identical between Kallisto- and Seq2Fun-derived count tables, reproducing Supplementary Figs. 2–3 and Tables 1–2
- Every DE result row carries all three columns (logFC, p-value, FDR-adjusted p-value); logFC is log2 transformed
- QC PCA plot of normalized data shows samples separating by the metadata of interest before DE is run (PlotDataPCA / PlotMetaPCA)
- Per-function sequence integrity: no ExpressAnalystR commands skipped between Init.Data and PerformDEAnal, since skipping causes downstream errors

## Limitations

- Genes were defined as DE only when FDR-adjusted p-values were less than 0.05 and absolute log fold changes were greater than 1.5 in the equivalence analysis; using different thresholds will not reproduce published counts
- Sample SRR7499350 must be excluded as an outlier identified by QA/QC in the original salamander publication; keeping it changes the DEG count
- Details on the sex of salamander samples were not included in the original manuscript, a reproducibility gap for the DE design
- ExpressAnalystR is still under development — full functionality is not guaranteed; results are returned as qs files whose values are often web-server-internal

## Evidence

- [discussion] Genes were defined as differentially expressed if their false discovery rate (FDR) adjusted p-values were less than 0.05 and the absolute log fold changes greater than 1.5.: "Genes were defined as differentially expressed if their false discovery rate (FDR) adjusted p-values were less than 0.05 and the absolute log fold changes greater than 1.5."
- [discussion] Salamander DEG threshold and outlier exclusion: "genes were considered differentially expressed if their FDR-adjusted p-values were less than 0.1."
- [discussion] Filtering thresholds used before DE: "setting the variance filter to 15 and the abundance filter to 4"
- [readme] README DE workflow with limma design matrix: "dataSets <- SetupDesignMatrix("estrogen.txt", "limma");"
- [readme] README fold change and FDR adjustment semantics: "Fold change is log2 transformed. Adjusted P-value using False Discovery Rate (FDR) method."
