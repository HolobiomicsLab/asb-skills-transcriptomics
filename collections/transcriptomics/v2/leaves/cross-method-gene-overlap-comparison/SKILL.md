---
name: cross-method-gene-overlap-comparison
description: Use when comparing whether a trajectory-based scRNA-seq DEG method identifies
  a higher fraction of literature-verified genes among its top-ranked DEGs than a
  competing method on the same preprocessed expression matrix.
when_to_use_negative:
- The two methods were not run on the same preprocessed expression matrix — overlap
  percentages are then not comparable.
- The comparison requires genome-wide DEG concordance rather than agreement among
  the highest-ranked (top-20) genes.
- No independent literature-verified gene set is available to define the overlap basis.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scTITANS
  role: Trajectory inference-based DEG method whose top-20 significant genes (from
    SigGenes.fdr0.01.txt, full mode, FDR 0.01) form one side of the overlap comparison
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Monocle3
  role: Constructs the trajectory and calculates per-cell pseudotime used by scTITANS
    before DEG identification
- name: scran
  role: Normalizes the filtered count matrix with the default pool-and-deconvolute
    algorithm so both methods are compared on identical normalized input
- name: edge
  role: Bulk RNA-seq baseline DEG method whose top-20 q-value-ranked genes form the
    comparison side of the overlap analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:f7e2325a190f567f219fd8405ed2edc34050657d17f07bdd74a839167f9d2d2d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scTITANS
  repo_url: https://github.com/ZJUFanLab/scTITANS
  tools:
  - Monocle3
  - edge
  - scTITANS
  - scran
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# cross-method-gene-overlap-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks two differential expression methods (e.g., scTITANS vs the bulk RNA-seq method 'edge') by ranking each method's DEGs by q-value, retaining the top-20 genes per method, and computing the percentage of overlapping literature-verified genes, assessed with a Fisher exact test.

## When to use

Use when comparing whether a trajectory-based scRNA-seq DEG method identifies a higher fraction of literature-verified genes among its top-ranked DEGs than a competing method on the same preprocessed expression matrix.

## When NOT to use

- The two methods were not run on the same preprocessed expression matrix — overlap percentages are then not comparable.
- The comparison requires genome-wide DEG concordance rather than agreement among the highest-ranked (top-20) genes.
- No independent literature-verified gene set is available to define the overlap basis.

## Inputs

- raw U count matrix for the AD dataset (GSE98969, mouse CD45+ immune cells from Alzheimer's disease brain, 8016 cells) from NCBI GEO
- scTITANS significant DEG list (SigGenes.fdr0.01.txt)
- edge baseline DEG list on the same preprocessed matrix
- list of literature-verified genes for the disease/condition

## Outputs

- gene_overlap_top20.csv containing overlap percentages for the top-20 DEG lists
- Fisher exact test p-value and statistical power for the scTITANS vs edge overlap comparison

## How to apply

Run both DEG methods on the identical preprocessed matrix — for the AD (GSE98969) dataset, the mouse CD45+ immune cell matrix (8016 cells) is filtered (genes detected with >=2 transcripts in at least 10 cells; cells outside the 5th-95th percentile gene counts or with >10% mitochondrial UMIs removed), scran-normalized, and analyzed by scTITANS_full with root_type 'none' and fdr_level 0.01, using Monocle3 to construct the trajectory and pseudotime. Collect scTITANS significant DEGs from SigGenes.fdr0.01.txt and run 'edge' on the same matrix. Rank each method's DEGs by q-value significance and retain the top-20 per method, then compute the percentage of overlapping literature-verified genes and evaluate the difference with a Fisher exact test, checking statistical power. Write overlap percentages to gene_overlap_top20.csv and compare against Table S3/S4.

## Related tools

- **scTITANS** (Trajectory inference-based DEG method whose top-20 significant genes (from SigGenes.fdr0.01.txt, full mode, FDR 0.01) form one side of the overlap comparison) — https://github.com/ZJUFanLab/scTITANS
- **Monocle3** (Constructs the trajectory and calculates per-cell pseudotime used by scTITANS before DEG identification)
- **scran** (Normalizes the filtered count matrix with the default pool-and-deconvolute algorithm so both methods are compared on identical normalized input)
- **edge** (Bulk RNA-seq baseline DEG method whose top-20 q-value-ranked genes form the comparison side of the overlap analysis)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- Computed gene-overlap percentages in gene_overlap_top20.csv must match the values reported in Table S3/S4 (output_matches_reference against the supplementary tables).
- The expected AD-dataset result is 80% literature-verified genes for scTITANS top-20 versus 40% for edge top-20, with Fisher exact p-value 0.30.
- Statistical power of the Fisher exact test should be reported (0.60 in the reference run) to judge whether a non-significant p-value is reliable.
- Both DEG lists must derive from the same scran-normalized, filtered matrix, and each top-20 list must be ranked by q-value significance.

## Limitations

- The reported scTITANS vs edge difference on AD was not statistically significant (p-value 0.30) and power was low (0.60), so the p-value may be unreliable.
- Software version numbers for Monocle3, scran, and scTITANS are not stated, complicating exact reproduction.
- If scTITANS cluster identification fails, an error is displayed and SigClusters output may be missing (README).

## Evidence
