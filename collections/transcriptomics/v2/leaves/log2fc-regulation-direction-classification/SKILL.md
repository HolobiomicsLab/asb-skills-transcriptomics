---
name: log2fc-regulation-direction-classification
description: Use when a DESeq2 differential expression table of tncRNA (or gene) counts
  across stress vs. control conditions must be filtered to significant hits and each
  hit labeled as up- or down-regulated by log2 fold-change direction.
when_to_use_negative:
- The input is already a filtered/significant hits table with direction labels — re-running
  the classification would only duplicate the up/down assignment step
- Count data lack biological replicates (fewer than three per condition), which the
  DESeq2 dispersion estimation in this workflow requires
- The goal is tncRNA identification or quantification from raw sRNA-seq FASTQ rather
  than differential expression testing of an existing count matrix — use the tncRNA
  Toolkit pipeline instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DESeq2
  role: Runs the Wald test on the tncRNA count matrix and produces log2 fold changes
    and Benjamini-Hochberg-corrected p-values that feed the direction classification
- name: tncRNA Toolkit
  role: Upstream workflow that detects and quantifies tRF-5s, tRF-3s, tRF-1s, and
    tRNA halves from genome FASTA and adapter-trimmed small RNA reads, supplying the
    counts classified here
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Trim Galore
  role: Quality- and adapter-trims the single-end small RNA FASTQ reads before tncRNA
    counting, ensuring clean count-matrix input
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:56accf6d769020eeeadc78d1c9801df25aab696d4ccb8d7771aa2cfa97540e1c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - DESeq2
  - Trim Galore
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# log2fc-regulation-direction-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies stress-responsive plant tncRNAs as up- or down-regulated by filtering DESeq2 Wald-test results to padj < 0.05 and assigning direction via a |log2FC| > 1 cutoff, so that only tncRNAs with both statistical significance and a magnitude threshold are cataloged as differentially expressed.

## When to use

Use when a DESeq2 differential expression table of tncRNA (or gene) counts across stress vs. control conditions must be filtered to significant hits and each hit labeled as up- or down-regulated by log2 fold-change direction.

## When NOT to use

- The input is already a filtered/significant hits table with direction labels — re-running the classification would only duplicate the up/down assignment step
- Count data lack biological replicates (fewer than three per condition), which the DESeq2 dispersion estimation in this workflow requires
- The goal is tncRNA identification or quantification from raw sRNA-seq FASTQ rather than differential expression testing of an existing count matrix — use the tncRNA Toolkit pipeline instead

## Inputs

- tncRNA count matrix for selected control and stress samples (constructed per Supplementary sheet 3.1 sample listing)
- Sample metadata grouping replicates into control vs. stress conditions (>= 3 biological replicates each)

## Outputs

- differential_expression_results.csv containing significant (p < 0.05) tncRNAs with log2 fold changes, p-values, and up/down direction labels

## How to apply

Start from a tncRNA count matrix built from sample sets with at least three biological replicates per condition (control and stress). Run DESeq2 on the count matrix using the Wald test, letting p-values be corrected by the default Benjamini-Hochberg method, then retain only tncRNAs with p < 0.05. Apply the direction rule to each significant tncRNA: label it up-regulated when log2FC > 1 and down-regulated when log2FC < -1. Write the surviving tncRNAs with their log2 fold changes and p-values to differential_expression_results.csv. The rationale is that the dual threshold (significance plus effect size) guards against cataloging trivially small expression shifts as stress responses, which matters when screening thousands of tncRNAs across abiotic and biotic stress contrasts in six angiosperms.

## Related tools

- **DESeq2** (Runs the Wald test on the tncRNA count matrix and produces log2 fold changes and Benjamini-Hochberg-corrected p-values that feed the direction classification)
- **tncRNA Toolkit** (Upstream workflow that detects and quantifies tRF-5s, tRF-3s, tRF-1s, and tRNA halves from genome FASTA and adapter-trimmed small RNA reads, supplying the counts classified here) — https://github.com/skbinfo/tncRNA-Toolkit
- **Trim Galore** (Quality- and adapter-trims the single-end small RNA FASTQ reads before tncRNA counting, ensuring clean count-matrix input) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
Rscript -e 'dds <- DESeqDataSetFromMatrix(cnts, coldata, ~condition); res <- results(DESeq(dds, test="Wald")); sig <- subset(res, padj<0.05); dir <- ifelse(sig$log2FoldChange>1,"up",ifelse(sig$log2FoldChange<-1,"down",NA)); write.csv(cbind(sig,dir),"differential_expression_results.csv")'
```

## Evaluation signals

- Output table contains only tncRNAs passing the p < 0.05 (Benjamini-Hochberg adjusted) significance threshold — no rows with padj >= 0.05
- Every row carries a direction label consistent with the rule: 'up' only when log2FC > 1, 'down' only when log2FC < -1; no direction assigned outside these bounds
- Each reported tncRNA has both a numeric log2 fold change and a p-value written to differential_expression_results.csv
- Every sample group in the input count matrix had at least three biological replicates for both control and treatment before DESeq2 was run
- Row counts are plausible against the paper's scale — e.g. the heat-stress contrast in maize tassel yielded 1748 differentially expressed tncRNAs, the largest stress response observed

## Limitations

- tncRNAs with |log2FC| between 0 and 1 but padj < 0.05 are significant yet receive no up/down label, so they are excluded from the direction-cataloged output
- DE tncRNAs and their putative targets still require experimental validation, as the authors note this is needed to understand the underlying stress-response mechanisms
- Existing detection tools (e.g. tDRmapper, MINTmap, tRF2Cancer) were trained and tested on human datasets, so plant-specific count generation depends on the tncRNA Toolkit pipeline rather than off-the-shelf mammalian tools

## Evidence

- [methods] Classify significant tncRNAs as up-regulated when log2FC > 1 and down-regulated when log2FC < -1.: [evidence span withheld — anchor status: not_found]
- [methods] Run DESeq2 on the count matrix using the Wald test, with p-values corrected for multiple testing via the Benjamini and Hochberg method (default).: [evidence span withheld — anchor status: not_found]
- [results] The largest differential expression response was to heat stress: 1748 tncRNAs were differentially expressed in maize tassel (reproductive stage) after 48 h of heat stress.: [evidence span withheld — anchor status: not_found]
