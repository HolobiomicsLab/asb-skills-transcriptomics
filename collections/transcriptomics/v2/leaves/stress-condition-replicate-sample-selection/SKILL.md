---
name: stress-condition-replicate-sample-selection
description: Use when preparing a count matrix of tncRNA expression for DESeq2 stress
  contrasts and you must pick which control/treatment samples qualify, ensuring ≥3
  biological replicates per condition.
when_to_use_negative:
- A stress contrast lacks at least three biological replicates in either the control
  or treatment condition — DESeq2 dispersion estimation will be unreliable, so the
  contrast should be excluded rather than forced.
- The goal is tncRNA discovery/identification rather than differential expression
  testing — sample replicate balance is not required, and the tncRNA Toolkit pipeline
  should be run on all samples.
- Input data are already aggregated or tissue-level matrices without per-replicate
  sample annotation (e.g. tissues with ≥10 samples used for t-SNE matrix generation)
  — replicate structure needed for the Wald test cannot be recovered.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0780
tools:
- name: DESeq2
  role: Differential expression testing (Wald test, Benjamini-Hochberg correction)
    applied to the replicate-selected tncRNA count matrix
- name: tncRNA Toolkit
  role: Produces the per-sample tncRNA counts that the replicate selection feeds into
    the DESeq2 count matrix
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: NCBI SRA Toolkit (v2.10)
  role: Downloads the single-end Illumina sRNA-seq samples from SRA that are then
    screened for replicate availability
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:dcebff62e9f6ac4e7c1ce26fe34c321edf6a7077081a11b4b6aa2a61bfb75490
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
  - NCBI SRA Toolkit (v2.10)
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# stress-condition-replicate-sample-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects sRNA-seq sample sets with at least three biological replicates per condition (control and stress) from the tncRNA sample listings (Supplementary sheet 3.1) so that DESeq2 differential expression of tncRNAs under abiotic and biotic stress is statistically reliable.

## When to use

Use when preparing a count matrix of tncRNA expression for DESeq2 stress contrasts and you must pick which control/treatment samples qualify, ensuring ≥3 biological replicates per condition.

## When NOT to use

- A stress contrast lacks at least three biological replicates in either the control or treatment condition — DESeq2 dispersion estimation will be unreliable, so the contrast should be excluded rather than forced.
- The goal is tncRNA discovery/identification rather than differential expression testing — sample replicate balance is not required, and the tncRNA Toolkit pipeline should be run on all samples.
- Input data are already aggregated or tissue-level matrices without per-replicate sample annotation (e.g. tissues with ≥10 samples used for t-SNE matrix generation) — replicate structure needed for the Wald test cannot be recovered.

## Inputs

- Sample listing corresponding to Supplementary sheet 3.1 (sample-to-condition mapping for stress contrasts)
- tncRNA count matrix (or per-sample tncRNA quantifications) from the tncRNA Toolkit pipeline
- Processed single-end sRNA-seq FASTQ files (quality filtered and adapter trimmed) underlying the sample set

## Outputs

- Selected sample set (≥3 biological replicates for both control and treatment) for one or more stress contrasts
- Count matrix of tncRNA counts restricted to the selected control and stress samples, ready for DESeq2

## How to apply

Start from the sample listing corresponding to Supplementary sheet 3.1, which catalogs the 2,448 sRNA-seq samples from SRA across the six angiosperms (Arabidopsis, Medicago, tomato, chickpea, rice, maize). For each stress contrast (e.g. heat stress in maize tassel at 48 h), retain only sample sets where both control and treatment conditions have at least three biological replicates; drop contrasts with fewer replicates rather than relaxing the threshold. Use the tncRNA counts produced by the tncRNA Toolkit pipeline (from quality/adapter-trimmed reads processed with Trim Galore v0.6.6 and mapped to the artificial tRNA genome with bowtie) to build a count matrix restricted to the selected control and stress samples. This replicate threshold is a prerequisite for the DESeq2 Wald test with Benjamini-Hochberg correction; significance is subsequently called at p < 0.05 with up/down direction at |log2FC| > 1.

## Related tools

- **DESeq2** (Differential expression testing (Wald test, Benjamini-Hochberg correction) applied to the replicate-selected tncRNA count matrix)
- **tncRNA Toolkit** (Produces the per-sample tncRNA counts that the replicate selection feeds into the DESeq2 count matrix) — https://github.com/skbinfo/tncRNA-Toolkit
- **NCBI SRA Toolkit (v2.10)** (Downloads the single-end Illumina sRNA-seq samples from SRA that are then screened for replicate availability)

## Examples

```
dds <- DESeqDataSetFromMatrix(countData=count_mat, colData=meta, design=~condition); res <- results(DESeq(dds), contrast=c("condition","stress","control")); sig <- subset(res, pvalue<0.05)
```

## Evaluation signals

- Every contrast in the selected sample set has ≥3 biological replicates in both control and treatment conditions, as cross-checked against Supplementary sheet 3.1.
- The resulting count matrix contains only samples belonging to the selected contrasts, with correct condition labels for DESeq2 colData.
- Downstream DESeq2 output (differential_expression_results.csv) contains only tncRNAs with p < 0.05, with up-regulation assigned when log2FC > 1 and down-regulation when log2FC < -1.
- No samples from tissues/conditions with insufficient replicates leak into the DE analysis, and the number of DE tncRNAs is consistent with per-contrast reporting (e.g. 1748 DE tncRNAs in maize tassel after 48 h heat stress).

## Limitations

- Sample availability in SRA constrains which stress contrasts can be analyzed; many contrasts cannot meet the ≥3-replicate requirement.
- Conventional RNA-seq-based tncRNA quantification carries tRNA modification biases, which the article notes would require switching to tRNA-seq to overcome.
- DE tncRNAs and their putative targets identified from these sample sets still require further experimental validation.

## Evidence

- [methods] Select sample sets in which at least three biological replicates are present for both control and treatment conditions: [evidence span withheld — anchor status: not_found]
- [other] Run DESeq2 on the count matrix using the Wald test: [evidence span withheld — anchor status: not_found]
