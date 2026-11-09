---
name: apa-usage-ratio-quantification
description: Use when you have a poly(A) site-by-spot APA count matrix from spatially
  barcoded ST data (e.g. MOB ST) plus a gene-spot expression count matrix, and need
  dense per-spot APA usage (RUD) values for downstream spatial APA analysis.
when_to_use_negative:
- Input is image-based spatial data such as MERFISH or seqFISH — the article notes
  APA sites can only be identified from spatially barcoded ST data using existing
  scRNA-seq tools like scAPAtrap or Sierra
- The APA usage matrix is already dense/imputed — running KNN imputation on a complete
  RUD matrix is unnecessary and could distort observed usage values
- The goal is downstream classification of DEAPA/LSAPA/SVAPA genes; that uses the
  imputed RUD output as input (findContrastAPA, findAllMarkers, findSVAPA) rather
  than this quantification step itself
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: stAPAminer
  role: R package providing computeAPAIndex to calculate APA ratios (RUD) from the
    APA-spot matrix and imputeAPAIndex to impute missing RUD entries using a k-nearest-neighbor
    model over gene-expression-based spot distances
  repo: https://github.com/BMILAB/stAPAminer
- name: movAPA
  role: Required R package; the APA-spot count matrix input is stored as a movAPA
    PACdataset (APA.RDA)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:c73103647502b0cb6c32b8bf6f6b8f1b00ad3dbad5a9b971b2466c4892e96d11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - movAPA
  - stAPAminer
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# apa-usage-ratio-quantification

## Summary

Quantifies alternative polyadenylation (APA) usage in spatially barcoded ST data as per-spot relative usage deviation (RUD) values via stAPAminer's computeAPAIndex, then recovers the highly sparse RUD matrix with a KNN-based imputation model (imputeAPAIndex) that borrows APA usage information from expression-similar neighboring spots.

## When to use

Use when you have a poly(A) site-by-spot APA count matrix from spatially barcoded ST data (e.g. MOB ST) plus a gene-spot expression count matrix, and need dense per-spot APA usage (RUD) values for downstream spatial APA analysis.

## When NOT to use

- Input is image-based spatial data such as MERFISH or seqFISH — the article notes APA sites can only be identified from spatially barcoded ST data using existing scRNA-seq tools like scAPAtrap or Sierra
- The APA usage matrix is already dense/imputed — running KNN imputation on a complete RUD matrix is unnecessary and could distort observed usage values
- The goal is downstream classification of DEAPA/LSAPA/SVAPA genes; that uses the imputed RUD output as input (findContrastAPA, findAllMarkers, findSVAPA) rather than this quantification step itself

## Inputs

- APA-spot count matrix stored as a movAPA PACdataset (APA.RDA) with rows as poly(A) sites and columns as spots
- gene-spot expression count matrix (count.csv)
- table of individual spots and their spatial coordinates (position.txt)

## Outputs

- raw sparse RUD (relative usage deviation) matrix from computeAPAIndex
- imputed dense RUD matrix from imputeAPAIndex (exported as RUD_imputed.csv)

## How to apply

Load the APA quantification stored as a movAPA PACdataset (APA.RDA) and the gene-spot count matrix (count.csv), then obtain the raw sparse RUD matrix with computeAPAIndex(APA, rownames(position)) restricted to the spots in position.txt. Scale the gene-spot count matrix and compute a spot-spot distance ranking from expression similarity, because stAPAminer's KNN model uses the spatial gene expression profile to find neighbors for APA usage recovery. Iteratively fill missing RUD entries by averaging the non-zero RUD values of the k=10 nearest spots (imputeAPAIndex with init=TRUE), repeating until no new entries can be filled; any entries still missing after the full iteration are set to 0 per the package convention. Write the completed imputed RUD matrix (e.g. RUD_imputed.csv) and verify it by checking that clustering of the imputed RUD values reproduces the expected MOB morphological layers (GCL, GL, ONL, OPL, MCL).

## Related tools

- **stAPAminer** (R package providing computeAPAIndex to calculate APA ratios (RUD) from the APA-spot matrix and imputeAPAIndex to impute missing RUD entries using a k-nearest-neighbor model over gene-expression-based spot distances) — https://github.com/BMILAB/stAPAminer
- **movAPA** (Required R package; the APA-spot count matrix input is stored as a movAPA PACdataset (APA.RDA))

## Examples

```
RUD_RAW <- computeAPAIndex(APA, rownames(position)); RUD <- imputeAPAIndex(RUD_RAW, count, k = 10, init = TRUE)
```

## Evaluation signals

- After imputeAPAIndex, no missing entries should remain that the iteration could fill; entries never filled must equal 0 per the package convention
- Imputation should use exactly k=10 nearest spots (k=10 passed to imputeAPAIndex), averaging only non-zero RUD values of neighbors
- RUD values are relative usage deviations, so each gene's per-spot values should remain within a valid ratio-deviation range (roughly bounded, not unbounded counts)
- Clustering the imputed RUD matrix (makeStCluster, resolution=0.6) should recover the MOB morphological layers GCL, GL, ONL, OPL, MCL
- optimalKvalue over orders seq(4,14,2) with ncluster=5 should support the chosen neighborhood parameter on the raw RUD matrix

## Limitations

- There is no method to determine whether an entry into the APA ratio matrix is truly missing before imputation
- Missing APA usage entries remaining after the imputation iterations are arbitrarily set to 0
- The article notes it will be necessary to benchmark different methods for obtaining gene expression matrices and/or RUD matrices

## Evidence

- [intro] an imputation model based on k-nearest neighbors (KNN) was designed to recover APA signals obtained from the ST data by borrowing information from the spatial gene expression profile: "an imputation model based on k-nearest neighbors (KNN) was designed to recover APA signals obtained from the ST data by borrowing information from the spatial gene expression profile"
- [methods] RUD_RAW<-computeAPAIndex(APA,rownames(position)): "RUD_RAW<-computeAPAIndex(APA,rownames(position))"
- [methods] RUD <-imputeAPAIndex(RUD_RAW,count,k=10,init = TRUE): "RUD <-imputeAPAIndex(RUD_RAW,count,k=10,init = TRUE)"
- [discussion] If there are still missing entries after the entire iteration process, we can directly set their values to 0.: "If there are still missing entries after the entire iteration process, we can directly set their values to 0."
- [readme] ## calculating APA ratios RUD <- computeAPAIndex(APA,rownames(position)) ## imputation RUD <- imputeAPAIndex(RUD_RAW,count,k=10): [evidence span withheld — anchor status: external_doc]
