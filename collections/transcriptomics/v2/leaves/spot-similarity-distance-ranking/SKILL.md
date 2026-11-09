---
name: spot-similarity-distance-ranking
description: Use when you have a sparse spot-by-poly(A)-site RUD matrix from ST data
  plus a gene-spot count matrix for the same spots, and missing RUD entries must be
  imputed before downstream spatial APA analyses.
when_to_use_negative:
- The input is a gene expression matrix needing no APA imputation — this skill only
  recovers sparse APA usage (RUD) entries
- Data are image-based ST such as MERFISH/seqFISH, since APA sites can only be identified
  from spatially barcoded ST data using tools like scAPAtrap or Sierra
- The RUD matrix is already complete with no missing entries to impute
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: stAPAminer
  role: R package providing computeAPAIndex to compute the raw RUD matrix and imputeAPAIndex
    to impute missing RUD values using the k=10 nearest spots determined from the
    gene expression profile
  repo: https://github.com/BMILAB/stAPAminer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:7e2bfadbc3622254953b6b97836237c98ad2de33e3977231104ca0cc9a5dac49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - stAPAminer
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# spot-similarity-distance-ranking

## Summary

A KNN-based imputation step from stAPAminer that recovers sparse APA usage (RUD) values in spatially barcoded ST data by inferring spot–spot distances from the scaled gene expression profile and borrowing APA usage from neighboring spots.

## When to use

Use when you have a sparse spot-by-poly(A)-site RUD matrix from ST data plus a gene-spot count matrix for the same spots, and missing RUD entries must be imputed before downstream spatial APA analyses.

## When NOT to use

- The input is a gene expression matrix needing no APA imputation — this skill only recovers sparse APA usage (RUD) entries
- Data are image-based ST such as MERFISH/seqFISH, since APA sites can only be identified from spatially barcoded ST data using tools like scAPAtrap or Sierra
- The RUD matrix is already complete with no missing entries to impute

## Inputs

- APA-spot count matrix as a movAPA PACdataset (APA.RDA) with poly(A) sites as rows and spots as columns
- gene-spot expression count matrix (count.csv) for the same spots
- spot barcode and coordinate table (position.txt)

## Outputs

- imputed RUD (relative usage of distal poly(A)) matrix with missing APA usage entries recovered or set to 0 (RUD_imputed.csv)
- intermediate raw RUD matrix from computeAPAIndex (RUD_RAW)

## How to apply

Load the APA-spot count matrix (e.g., APA.RDA as a movAPA PACdataset), the gene-spot count matrix (count.csv), and spot coordinates (position.txt). Compute the raw RUD matrix with computeAPAIndex(APA, rownames(position)), then scale the gene-spot expression count matrix and compute a spot–spot distance ranking matrix from expression similarity. Iteratively fill missing RUD entries by averaging the non-zero RUD values of the k=10 nearest spots via imputeAPAIndex with init=TRUE, repeating until no new entries can be filled. Any RUD entries still missing after the full iteration are set to 0 per the package convention, yielding the completed imputed RUD matrix written to RUD_imputed.csv.

## Related tools

- **stAPAminer** (R package providing computeAPAIndex to compute the raw RUD matrix and imputeAPAIndex to impute missing RUD values using the k=10 nearest spots determined from the gene expression profile) — https://github.com/BMILAB/stAPAminer

## Examples

```
RUD_RAW <- computeAPAIndex(APA, rownames(position)); RUD <- imputeAPAIndex(RUD_RAW, count, k=10, init = TRUE)
```

## Evaluation signals

- The imputed RUD matrix has no missing (NA) entries: unfilled slots after the iterative KNN pass equal exactly 0 per the package convention
- The imputed matrix retains the same spot barcodes (rownames(position)) and poly(A) sites as the raw RUD matrix
- Clustering the imputed RUD (e.g., makeStCluster with resolution=0.6, k=30) recovers MOB morphological layers (GCL, GL, ONL, OPL, MCL) via createStAPAminerObject and findLabels
- Correlation-based verification with computePearsonIndex/computeIndex and drawVolcano between layers supports imputation correctness
- Optional k tuning via optimalKvalue(RUD_RAW, count, position, ncluster=5, orders=c(seq(4,14,2))) should agree with the chosen k=10

## Limitations

- Entries still missing after the entire iteration process are arbitrarily set to 0, which may distort downstream usage estimates
- No method is provided to determine whether an entry in the APA ratio matrix is truly missing before imputation
- Only applicable to spatially barcoded ST data; APA sites cannot be identified from image-based ST platforms such as MERFISH/seqFISH with existing tools
- Discrepancies among replicates are not mitigated by this imputation step

## Evidence

- [intro] an imputation model based on k-nearest neighbors (KNN) was designed to recover APA signals obtained from the ST data by borrowing information from the spatial gene expression profile: "an imputation model based on k-nearest neighbors (KNN) was designed to recover APA signals obtained from the ST data by borrowing information from the spatial gene expression profile"
- [methods] imputeAPAIndex call with k=10: "RUD <-imputeAPAIndex(RUD_RAW,count,k=10,init = TRUE)"
- [readme] README description of the KNN imputation model: [evidence span withheld — anchor status: external_doc]
- [discussion] Missing entries set to 0 after iteration: "If there are still missing entries after the entire iteration process, we can directly set their values to 0."
