---
name: differential-expression-testing
description: Use when DC3 subpopulation-specific FPKM profiles are available and transcription
  factors must be tested for differential expression between subpopulations to build
  a key-regulator candidate list.
when_to_use_negative:
- Single cells have not yet been clustered/assigned to DC3 subpopulations, so no subpopulation-specific
  FPKM profiles exist to test
- Only bulk-level profiles are available without per-subpopulation values, leaving
  no replicate groups for the t-test
- No scATAC-seq data are available, so the motif enrichment score > 2 filter and motif
  component of the importance score cannot be computed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DC3
  role: Produces the coupled clustering and subpopulation-specific expression profiles
    that the t-test is applied to
  repo: github.com/SUwonglab/DC3
- name: scipy
  role: Two-sided t-test for differential expression between subpopulations with adjusted
    p-value < 0.01
- name: MACS2
  role: Calls subpopulation-specific open peaks on merged scATAC-seq reads as input
    to motif enrichment filtering
- name: Homer
  role: Computes motif enrichment scores (geometric mean of -log10(p-value) and fold
    change) used to filter candidate TFs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:493cb3f9fa6e20478cfbd55374324d9a77e79663baa6093969c0f5d21979de1a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  - Homer
  - MACS2
  - scipy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# differential-expression-testing

## Summary

Statistical testing of DC3-derived subpopulation-specific expression profiles using a two-sided t-test (scipy) with adjusted p-value < 0.01, to select transcription factors significantly differentially expressed between RA-day 4 subpopulations before key-regulator ranking.

## When to use

Use when DC3 subpopulation-specific FPKM profiles are available and transcription factors must be tested for differential expression between subpopulations to build a key-regulator candidate list.

## When NOT to use

- Single cells have not yet been clustered/assigned to DC3 subpopulations, so no subpopulation-specific FPKM profiles exist to test
- Only bulk-level profiles are available without per-subpopulation values, leaving no replicate groups for the t-test
- No scATAC-seq data are available, so the motif enrichment score > 2 filter and motif component of the importance score cannot be computed

## Inputs

- DC3 subpopulation assignments for scRNA-seq cells (cluster-specific expression matrices, one column per subpopulation)
- Subpopulation-level FPKM gene expression profiles merged per DC3 cluster
- Subpopulation-specific scATAC-seq peak calls (MACS2) with Homer motif enrichment scores for TF filtering

## Outputs

- Per-subpopulation list of key regulators (TFs) passing FPKM > 10, motif enrichment score > 2, and adjusted p-value < 0.01 (58, 42, 71 TFs for subpopulations 1-3)
- Top-30 ranked key TFs per subpopulation by importance score (log2(FPKM) x specificity fold change x motif enrichment score)

## How to apply

First run DC3 to assign scRNA-seq and scATAC-seq cells to subpopulations (e.g., k=3 for the RA-day 4 mixture), merge reads per subpopulation, and compute subpopulation-specific FPKM. For each candidate transcription factor passing FPKM > 10 and motif enrichment score > 2 (geometric mean of -log10(p-value) and fold change from Homer on MACS2-called subpopulation peaks), run a scipy t-test of its FPKM values against each other subpopulation and apply multiple-testing correction, keeping TFs with adjusted p-value < 0.01. Rank surviving key regulators by the importance score = log2(FPKM) x expression specificity (maximum fold change versus the other two subpopulations) x motif enrichment score, and report the Top 30 per subpopulation. In the published application this yielded 58, 42, and 71 key regulators for subpopulations 1, 2, and 3, respectively.

## Related tools

- **DC3** (Produces the coupled clustering and subpopulation-specific expression profiles that the t-test is applied to) — github.com/SUwonglab/DC3
- **scipy** (Two-sided t-test for differential expression between subpopulations with adjusted p-value < 0.01)
- **MACS2** (Calls subpopulation-specific open peaks on merged scATAC-seq reads as input to motif enrichment filtering)
- **Homer** (Computes motif enrichment scores (geometric mean of -log10(p-value) and fold change) used to filter candidate TFs)

## Examples

```
from scipy.stats import ttest_ind; t, p = ttest_ind(subpop1_fpkm, subpop2_fpkm, axis=1)
```

## Evaluation signals

- Key regulator counts per subpopulation equal the reported 58, 42, and 71 for subpopulations 1, 2, and 3 of the RA-day 4 mixture
- High-ranking specific TFs reproduce the paper's results: Lhx1 and Neurod1 in subpopulation 1; Gata4, Sox17, and Foxa2 in subpopulation 2; Rfx4 and Sox3 in subpopulation 3
- Every reported key regulator satisfies all three filters: FPKM > 10, motif enrichment score > 2, and adjusted p-value < 0.01 from the scipy t-test
- Top-30 ranked lists per subpopulation are ordered by the importance score = log2(FPKM) x maximum fold-change specificity x motif enrichment score

## Limitations

- The t-test is defined on DC3-deconvolved/merged subpopulation profiles, not on raw single cells; results depend on the coupled clustering quality and the chosen number of clusters k
- Requires both scRNA-seq and scATAC-seq from the same heterogeneous population, since the FPKM and motif-enrichment filters must be computed on the same subpopulation assignments
- DC3's discussion notes emerging single-cell data types (e.g., jointly profiled sci-CAR data) are not yet incorporated into the inference

## Evidence

- [methods] adjusted p-value threshold for differential expression: [evidence span withheld — anchor status: not_found]
- [methods] importance-score ranking definition: [evidence span withheld — anchor status: not_found]
- [abstract] DC3 subpopulation profiles are the substrate for subpopulation-specific testing: "DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data"
- [readme] README requirement of scipy and expression matrix format: "For preprocessing the scRNA-seq data, please following the standard processing pipline to get the expression matrix, where each row represents a gene, each column represents a cell."
