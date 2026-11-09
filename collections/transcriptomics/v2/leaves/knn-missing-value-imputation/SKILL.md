---
name: knn-missing-value-imputation
description: Use when a spot-by-poly(A)-site RUD (relative usage difference) matrix
  derived from spatially barcoded ST data is too sparse for downstream clustering
  or spatial-pattern tests, and a companion gene-spot count matrix is available to
  define spot neighborhoods.
when_to_use_negative:
- 'Input is image-based ST data (MERFISH/seqFISH): APA sites can currently only be
  identified from spatially barcoded ST data using scRNA-seq tools like scAPAtrap
  or Sierra'
- There is no companion gene expression count matrix, since spot–spot distances are
  computed from the expression profile
- The RUD matrix is already dense, or zero vs. truly missing entries cannot be distinguished
  — the authors note there is no method to determine whether an entry is truly missing
  before imputation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stAPAminer
  role: R package providing computeAPAIndex, imputeAPAIndex (KNN imputation, k=10,
    init=TRUE) and optimalKvalue for recovering sparse APA usage signals
  repo: https://github.com/BMILAB/stAPAminer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:d94ed3a3145fe0ab793aaad512a2345ea619b3c3b757b1efa3ce8be1d5ddd0bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - stAPAminer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# knn-missing-value-imputation

## Summary

KNN-based imputation of sparse APA usage (RUD) matrices in stAPAminer: spot–spot distances are inferred from the spatial gene expression profile, and missing RUD entries are filled iteratively by borrowing APA usage values from the k nearest spots (default k=10).

## When to use

Use when a spot-by-poly(A)-site RUD (relative usage difference) matrix derived from spatially barcoded ST data is too sparse for downstream clustering or spatial-pattern tests, and a companion gene-spot count matrix is available to define spot neighborhoods.

## When NOT to use

- Input is image-based ST data (MERFISH/seqFISH): APA sites can currently only be identified from spatially barcoded ST data using scRNA-seq tools like scAPAtrap or Sierra
- There is no companion gene expression count matrix, since spot–spot distances are computed from the expression profile
- The RUD matrix is already dense, or zero vs. truly missing entries cannot be distinguished — the authors note there is no method to determine whether an entry is truly missing before imputation

## Inputs

- APA-spot count matrix as a movAPA PACdataset (e.g. APA.RDA shipped with stAPAminer)
- Gene-spot expression count matrix (count.csv)
- Spot barcode-to-coordinate table (position.txt)
- Raw RUD matrix produced by computeAPAIndex(APA, rownames(position))

## Outputs

- Complete imputed RUD matrix (imputed APA usage per spot, e.g. written to RUD_imputed.csv)
- StAPAminer object (stObj) with imputed RUD for downstream clustering and spatial APA-gene identification

## How to apply

First obtain the raw RUD matrix by running computeAPAIndex on the APA-spot count matrix (movAPA PACdataset, e.g. APA.RDA) restricted to the spots in position.txt. Scale the gene-spot expression count matrix (count.csv) and compute a spot–spot distance ranking from expression similarity, so that neighborhoods reflect transcriptional rather than physical proximity. Iteratively call imputeAPAIndex with k=10 and init=TRUE: each missing RUD entry is filled with the average of the non-zero RUD values of the k nearest spots, and the iteration repeats until no new entries can be filled. Optionally tune k via optimalKvalue(RUD_RAW, count, position, ncluster=5, orders=c(seq(4,14,2))) against the expected number of spatial clusters. Any RUD entries still missing after the full iteration are set to 0 per the package convention, and the completed matrix (e.g. RUD_imputed.csv) feeds downstream makeStCluster and DEAPA/LSAPA/SVAPA identification.

## Related tools

- **stAPAminer** (R package providing computeAPAIndex, imputeAPAIndex (KNN imputation, k=10, init=TRUE) and optimalKvalue for recovering sparse APA usage signals) — https://github.com/BMILAB/stAPAminer

## Examples

```
RUD <- imputeAPAIndex(RUD_RAW, count, k=10, init = TRUE)
```

## Evaluation signals

- After the iteration terminates, no new RUD entries can be filled; any residual missing values are exactly the entries set to 0 by convention
- Each imputed entry equals the average of the non-zero RUD values among the k=10 expression-nearest spots, not among arbitrary spots
- Spot–spot neighborhood rankings must derive from the scaled gene-spot count matrix (count.csv), matching the spot order in position.txt
- Downstream sanity check: clustering the imputed RUD (makeStCluster, resolution=0.6, dims=1:10) should recover the five MOB morphological layers (GCL, GL, ONL, OPL, MCL)
- Choice of k can be validated with optimalKvalue over orders seq(4,14,2) for ncluster=5

## Limitations

- Missing APA usage entries remaining after the entire iteration process are arbitrarily set to 0
- No method is provided to determine whether an APA ratio matrix entry is truly missing before imputation (a genuine zero may be imputed as if missing)
- Transformation of the APA ratio matrix into a count matrix for downstream SPARK may not be theoretically reasonable or fully meet the model's assumptions

## Evidence

- [intro] an imputation model based on k-nearest neighbors (KNN) was designed to recover APA signals obtained from the ST data by borrowing information from the spatial gene expression profile: "an imputation model based on k-nearest neighbors (KNN) was designed to recover APA signals obtained from the ST data by borrowing information from the spatial gene expression profile"
- [methods] RUD <-imputeAPAIndex(RUD_RAW,count,k=10,init = TRUE): "RUD <-imputeAPAIndex(RUD_RAW,count,k=10,init = TRUE)"
- [discussion] If there are still missing entries after the entire iteration process, we can directly set their values to 0.: "If there are still missing entries after the entire iteration process, we can directly set their values to 0."
- [discussion] First, it is crucial to determine whether an entry into the APA ratio matrix is truly missing before imputation.: "First, it is crucial to determine whether an entry into the APA ratio matrix is truly missing before imputation."
- [methods] optimalK <- optimalKvalue(RUD_RAW,count,position,ncluster=5,orders = c(seq(4,14,2))): "optimalK <- optimalKvalue(RUD_RAW,count,position,ncluster=5,orders = c(seq(4,14,2)))"
