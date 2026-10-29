---
name: spatial-expression-matrix-scaling
description: Use when a sparse APA usage (RUD) matrix from spatially barcoded ST data
  must be imputed using spot–spot distances derived from a scaled gene-spot expression
  profile (e.g., count.csv + APA.RDA inputs).
when_to_use_negative:
- The APA usage matrix is already dense/imputed — re-imputation with KNN would overwrite
  valid RUD values
- Data are not spatially barcoded ST (e.g., MERFISH/seqFISH), since APA sites can
  only be identified from spatially barcoded ST data with existing tools like scAPAtrap
  or Sierra
- No gene expression profile is available alongside the poly(A) site matrix, because
  the spot–spot distance ranking depends entirely on expression similarity
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: stAPAminer
  role: R package providing computeAPAIndex to compute raw RUD values, and imputeAPAIndex
    (k=10, init=TRUE) to scale-informed KNN-impute the sparse APA usage matrix
  repo: https://github.com/BMILAB/stAPAminer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:1daf155956c8e2356c1602170abec937a6bdbbbd0eaa6ec09f867458baf2123f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - stAPAminer
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spatial-expression-matrix-scaling

## Summary

Scales the gene-spot expression count matrix of spatially barcoded ST data to derive a spot–spot distance ranking matrix, which stAPAminer then uses to drive KNN-based imputation of the sparse APA usage (RUD) matrix.

## When to use

Use when a sparse APA usage (RUD) matrix from spatially barcoded ST data must be imputed using spot–spot distances derived from a scaled gene-spot expression profile (e.g., count.csv + APA.RDA inputs).

## When NOT to use

- The APA usage matrix is already dense/imputed — re-imputation with KNN would overwrite valid RUD values
- Data are not spatially barcoded ST (e.g., MERFISH/seqFISH), since APA sites can only be identified from spatially barcoded ST data with existing tools like scAPAtrap or Sierra
- No gene expression profile is available alongside the poly(A) site matrix, because the spot–spot distance ranking depends entirely on expression similarity

## Inputs

- Gene-spot expression count matrix (count.csv)
- APA-spot poly(A) site count matrix as a movAPA PACdataset (APA.RDA)
- Spot barcodes with coordinates (position.txt)

## Outputs

- Imputed APA usage (RUD) matrix (e.g., RUD_imputed.csv)
- Spot–spot distance ranking matrix derived from the scaled expression profile
- Raw RUD matrix prior to imputation (RUD_RAW from computeAPAIndex)

## How to apply

Load the APA-spot poly(A) site matrix (APA.RDA as a movAPA PACdataset), the gene-spot count matrix (count.csv), and spot coordinates (position.txt) shipped with stAPAminer, then obtain the raw RUD matrix via computeAPAIndex(APA, rownames(position)). Scale the gene-spot expression count matrix and compute a spot–spot distance ranking matrix from expression similarity, since the KNN model borrows APA usage information from neighboring spots defined by this expression-based ranking. Run iterative imputation with imputeAPAIndex(RUD_RAW, count, k=10, init=TRUE), which repeatedly fills missing RUD entries by averaging non-zero RUD values of the k=10 nearest spots until no new entries can be filled. Any RUD entries still missing after the full iteration are set to 0, matching the package convention, and the completed imputed RUD matrix is written out (e.g., RUD_imputed.csv).

## Related tools

- **stAPAminer** (R package providing computeAPAIndex to compute raw RUD values, and imputeAPAIndex (k=10, init=TRUE) to scale-informed KNN-impute the sparse APA usage matrix) — https://github.com/BMILAB/stAPAminer

## Examples

```
RUD <- imputeAPAIndex(RUD_RAW, count, k=10, init = TRUE)
```

## Evaluation signals

- The raw RUD matrix from computeAPAIndex(APA, rownames(position)) has the same spots (rows of position.txt) before and after scaling/imputation
- After imputeAPAIndex with k=10 and init=TRUE, no missing RUD entries remain; any residual missing entries are explicitly set to 0 per package convention
- Downstream clustering (makeStCluster) on the imputed RUD matrix reproduces the expected MOB morphological layers (GCL, GL, ONL, OPL, MCL) via findLabels
- Imputation correctness is verifiable by the package's correlation checks (computePearsonIndex/computeIndex and volcano plots comparing e.g. GCL vs GL)
- The spot–spot neighbor ranking must be computed from the scaled expression count matrix, not the raw counts, for KNN borrowing to be consistent with the article's method

## Limitations

- If there are still missing entries after the entire iteration process, their values are directly set to 0 — an arbitrary convention flagged as a gap in the article
- APA sites can only be identified from spatially barcoded ST data using existing scRNA-seq tools such as scAPAtrap or Sierra, limiting applicability to image-based technologies like MERFISH/seqFISH
- Benchmarking of different methods for obtaining the gene expression and RUD matrices is still needed, per the article's discussion

## Evidence

- [methods] imputeAPAIndex call: "RUD <-imputeAPAIndex(RUD_RAW,count,k=10,init = TRUE)"
- [methods] raw RUD computation: "RUD_RAW<-computeAPAIndex(APA,rownames(position))"
- [intro] KNN model rationale: "an imputation model based on k-nearest neighbors (KNN) was designed to recover APA signals obtained from the ST data by borrowing information from the spatial gene expression profile"
- [discussion] residual missing entries convention: "If there are still missing entries after the entire iteration process, we can directly set their values to 0."
- [readme] README example workflow: "## calculating APA ratios
RUD <- computeAPAIndex(APA,rownames(position))

## imputation
RUD <- imputeAPAIndex(RUD_RAW,count,k=10)"
