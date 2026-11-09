---
name: limma-voom-count-normalization
description: Use when the input is a raw RNA-seq gene/ortholog count matrix (e.g.,
  salamander samples SRR7499348–SRR7499365) and the goal is DE analysis with a primary
  factor plus a blocking factor at a defined FDR threshold.
when_to_use_negative:
- Input is a gene list or already-normalized expression values rather than raw counts
  — the README notes normalization may be skipped when PCA shows data are already
  normalized, and limma for microarray data requires 'mean' or 'median' summary instead
  of 'sum'
- Multi-dataset meta-analysis context — use the meta-analysis branch (SetAnalType("metadata"),
  PerformDEAnalMeta) instead of the one-data limma pipeline
- Raw FASTQ processing/annotation for non-model species — that is the Seq2Fun read-mapping
  stage, not the count-table statistics stage
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: ExpressAnalystR
  role: R package providing the sequential pipeline functions (Init.Data, ReadTabExpressData,
    PerformExpressNormalization, SetSelectedMetaInfo, SetupDesignMatrix, PerformDEAnal)
    that wrap limma voom normalization and DE testing
  repo: github.com/xia-lab/ExpressAnalystR
- name: limma
  role: 'Statistical engine invoked by SetupDesignMatrix and PerformDEAnal: voom log2
    CPM normalization, design matrix with blocking factor, empirical-Bayes moderated
    testing, and FDR adjustment'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:78c62fae00f5ef02a22811e784236ac547d1bacfd91ba2f46da845939fe6b608
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - ExpressAnalystR
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# limma-voom-count-normalization

## Summary

Normalization of raw RNA-seq count tables via limma voom (log2 CPM transformation) followed by linear-model differential expression in ExpressAnalystR, supporting blocking factors (e.g., species) and FDR-adjusted p-value DEG calls.

## When to use

Use when the input is a raw RNA-seq gene/ortholog count matrix (e.g., salamander samples SRR7499348–SRR7499365) and the goal is DE analysis with a primary factor plus a blocking factor at a defined FDR threshold.

## When NOT to use

- Input is a gene list or already-normalized expression values rather than raw counts — the README notes normalization may be skipped when PCA shows data are already normalized, and limma for microarray data requires 'mean' or 'median' summary instead of 'sum'
- Multi-dataset meta-analysis context — use the meta-analysis branch (SetAnalType("metadata"), PerformDEAnalMeta) instead of the one-data limma pipeline
- Raw FASTQ processing/annotation for non-model species — that is the Seq2Fun read-mapping stage, not the count-table statistics stage

## Inputs

- Tab-delimited raw RNA-seq count matrix (rows = genes/orthologs, columns = samples, e.g. SRR7499348–SRR7499365)
- Sample metadata table with primary factor (e.g., time) and blocking factor (e.g., species)
- ExpressAnalystR organism/annotation parameters (organism code, data type = RNA-seq, ID type, gene-level summary = sum)

## Outputs

- Normalized log2 CPM expression matrix (voom-transformed)
- Differential expression results table per gene: logFC, raw p-value, FDR-adjusted p-value (e.g., salamander_de_results.csv)
- DEG count at the chosen FDR threshold (2780 DEGs at FDR < 0.1 in the salamander case study)
- QC boxplot and PCA diagnostics (qc_boxplot_dpi72.png, qc_pca_dpi72.png)

## How to apply

Initialize ExpressAnalystR in local mode with Init.Data(FALSE) and SetAnalType("onedata"), then load the tab-delimited count table with ReadTabExpressData. Normalize the counts with limma voom — a log2 CPM transformation appropriate for RNA-seq count data — combining variance filtering (set to 15) and abundance filtering (set to 4, lower percentile removed) via PerformExpressNormalization; for RNA-seq, gene-level summarization must use sum rather than mean or median. Select the metadata of interest with SetSelectedMetaInfo (primary factor plus a blocking factor, e.g., species in the salamander case), then compute the linear-model design matrix with SetupDesignMatrix(..., "limma"). Run the contrast (e.g., time0 vs time24) with PerformDEAnal using a custom contrast, with log2-transformed fold changes and Benjamini–Hochberg FDR adjustment of p-values. Define DEGs at the study's chosen FDR cut-off (salamander publication: FDR-adjusted p < 0.1, no log2FC cut-off) and write the per-gene results table (logFC, p-value, FDR) to CSV.

## Related tools

- **ExpressAnalystR** (R package providing the sequential pipeline functions (Init.Data, ReadTabExpressData, PerformExpressNormalization, SetSelectedMetaInfo, SetupDesignMatrix, PerformDEAnal) that wrap limma voom normalization and DE testing) — github.com/xia-lab/ExpressAnalystR
- **limma** (Statistical engine invoked by SetupDesignMatrix and PerformDEAnal: voom log2 CPM normalization, design matrix with blocking factor, empirical-Bayes moderated testing, and FDR adjustment)

## Examples

```
dataSets <- SetupDesignMatrix("estrogen.txt", "limma"); dataSets <- PerformDEAnal("estrogen.txt", "custom", "time0 vs. time24", "NA", "intonly")
```

## Evaluation signals

- DEG count at the stated FDR threshold matches the published result: 2780 DEGs at FDR-adjusted p < 0.1 for the salamander analysis (18 samples, SRR7499350 excluded as QA/QC outlier)
- Results table contains all three required columns per gene: logFC (log2-transformed), raw p-value, and FDR-adjusted p-value
- QC PCA and boxplot diagnostics show reasonable sample separation by the primary factor and no residual batch/outlier structure
- Design matrix correctly encodes both the primary factor (time) and the blocking factor (species) before the custom contrast (time0 vs time24) is tested
- Functions were run in the documented sequence without skips — the README warns that skipping commands causes downstream errors

## Limitations

- ExpressAnalystR is under development and full functionality is not guaranteed; it is primarily useful for statistics (DE and meta-analysis), not visual analytics, and returned values are often less useful locally than the saved qs files
- Excluding a flagged outlier sample (e.g., SRR7499350) depends on prior QA/QC information that may not be present in the original manuscript; other sample details (e.g., sex) were also not reported in the original manuscript
- For taxonomically diverse cross-species analyses, reads from the same gene may map to similar but different orthologs across species, requiring care in interpretation

## Evidence

- [readme] RNA-seq needs sum for gene-level summarization: "For gene-level summary, microarray can use mean or median while RNA-seq needs to be sum."
- [discussion] variance and abundance filter settings: "setting the variance filter to 15 and the abundance filter to 4"
- [discussion] salamander DEG threshold: "genes were considered differentially expressed if their FDR-adjusted p-values were less than 0.1."
- [readme] fold change and FDR conventions in DE output: "Fold change is log2 transformed. Adjusted P-value using False Discovery Rate (FDR) method."
- [readme] normalization call with filter parameters: "dataSets <- PerformExpressNormalization("estrogen.txt", "none", 15, 4, 0);"
