---
name: benjamini-hochberg-multiple-testing-correction
description: Use when a tncRNA count matrix with ≥3 biological replicates per condition
  is being tested for differential expression across many tncRNA loci and per-locus
  p-values must be corrected for multiple testing before significance calling.
when_to_use_negative:
- Input is already a table of adjusted p-values or a single-hypothesis test (only
  one locus tested), where correction would be meaningless
- Sample sets lack at least three biological replicates for both control and treatment,
  so DESeq2 dispersion estimation is unreliable
- The goal is tncRNA discovery/quantification from raw sRNA-seq FASTQ rather than
  differential expression testing of an existing count matrix
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0659
tools:
- name: DESeq2
  role: Runs the Wald test on the tncRNA count matrix with default Benjamini-Hochberg
    multiple-testing correction over p-values
- name: tncRNA Toolkit
  role: Upstream pipeline that identifies, classifies, and quantifies tncRNAs (tRF-5s,
    tRF-3s, tRF-1, tRNA halves) from sRNA-seq FASTQ to produce the count matrix input
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Trim Galore
  role: Quality and adapter trimming of single-end small RNA reads prior to tncRNA
    prediction, ensuring accurate counts entering the DE test
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:cf892dc06ea33626bcd99f75266aa3e62c2cd3ef000f78a34e2277cb911d41e6
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
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# benjamini-hochberg-multiple-testing-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply Benjamini-Hochberg false-discovery-rate correction to Wald-test p-values when calling differentially expressed tncRNAs with DESeq2, so that thousands of tRNA-derived non-coding RNAs (tRF-5s, tRF-3s, tRF-1s, tRNA halves) can be tested simultaneously without inflating false positives.

## When to use

Use when a tncRNA count matrix with ≥3 biological replicates per condition is being tested for differential expression across many tncRNA loci and per-locus p-values must be corrected for multiple testing before significance calling.

## When NOT to use

- Input is already a table of adjusted p-values or a single-hypothesis test (only one locus tested), where correction would be meaningless
- Sample sets lack at least three biological replicates for both control and treatment, so DESeq2 dispersion estimation is unreliable
- The goal is tncRNA discovery/quantification from raw sRNA-seq FASTQ rather than differential expression testing of an existing count matrix

## Inputs

- Count matrix of tncRNA counts for selected control and stress samples (≥3 biological replicates per condition)
- Sample listing corresponding to Supplementary sheet 3.1 mapping samples to control/treatment conditions

## Outputs

- differential_expression_results.csv containing significant (p < 0.05) DE tncRNAs with log2 fold changes and (adjusted) p-values
- Per-tncRNA BH-adjusted p-values and direction calls (up if log2FC > 1, down if log2FC < -1)

## How to apply

Build a count matrix of tncRNA counts (identified via the tncRNA Toolkit from adapter-trimmed, quality-filtered sRNA-seq FASTQ) using only sample sets with at least three biological replicates for both control and stress conditions. Run DESeq2 with the Wald test, leaving the multiple-testing correction at its default Benjamini and Hochberg procedure so raw p-values are converted to BH-adjusted p-values across all tested tncRNAs. Filter results to significant tncRNAs with p < 0.05, then assign direction: up-regulated when log2FC > 1 and down-regulated when log2FC < -1. The BH correction is essential here because thousands of tncRNA loci are tested in parallel (e.g., 1748 DE tncRNAs in maize tassel after 48 h heat stress), and uncorrected p-values would yield a large number of spurious calls.

## Related tools

- **DESeq2** (Runs the Wald test on the tncRNA count matrix with default Benjamini-Hochberg multiple-testing correction over p-values)
- **tncRNA Toolkit** (Upstream pipeline that identifies, classifies, and quantifies tncRNAs (tRF-5s, tRF-3s, tRF-1, tRNA halves) from sRNA-seq FASTQ to produce the count matrix input) — https://github.com/skbinfo/tncRNA-Toolkit
- **Trim Galore** (Quality and adapter trimming of single-end small RNA reads prior to tncRNA prediction, ensuring accurate counts entering the DE test) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
Rscript -e 'library(DESeq2); dds <- DESeqDataSetFromMatrix(countData=cnt, colData=meta, design=~condition); dds <- DESeq(dds, test="Wald"); res <- results(dds, alpha=0.05); sig <- subset(res, pvalue<0.05); write.csv(sig, "differential_expression_results.csv")'
```

## Evaluation signals

- The output table (differential_expression_results.csv) contains only tncRNAs passing the p < 0.05 significance threshold
- Every retained tncRNA has an up/down direction consistent with the |log2FC| > 1 criterion (up if log2FC > 1, down if log2FC < -1)
- All samples entering the test had ≥3 biological replicates in both control and treatment conditions per Supplementary sheet 3.1
- Adjusted p-values are ≤ corresponding raw p-values, confirming BH correction was applied (DESeq2 default)
- Sanity benchmark: heat stress in maize tassel (reproductive stage, 48 h) should yield on the order of the reported 1748 DE tncRNAs when the same samples and thresholds are used

## Limitations

- DE tncRNAs and their putative targets require further experimental validation; statistical significance alone does not establish the regulatory mechanism
- Only sample sets with at least three biological replicates per condition are eligible, limiting usable contrasts
- Conventional RNA-seq/sRNA-seq carries tRNA modification biases that may affect tncRNA identification and hence the counts being tested; tRNA-seq would be needed to overcome these biases

## Evidence

- [other] Run DESeq2 on the count matrix using the Wald test, with p-values corrected for multiple testing via the Benjamini and Hochberg method (default).: [evidence span withheld — anchor status: not_found]
- [other] Filter results to keep only significant tncRNAs with a p value less than 0.05.: [evidence span withheld — anchor status: not_found]
- [other] Select sample sets in which at least three biological replicates are present for both control and treatment conditions: [evidence span withheld — anchor status: not_found]
