---
name: cross-replicate-apa-profile-correlation
description: Use when you have spatially barcoded ST poly(A) site data for multiple
  tissue-section replicates (e.g., ST-MOB Replicates 5, 11, 12) and need to test whether
  spatial APA patterns and gene-level DEAPA calls reproduce across sections.
when_to_use_negative:
- Input is single-cell or imaging-based spatial data (MERFISH/seqFISH) — the article
  states APA sites can only be identified from spatially barcoded ST data using tools
  like scAPAtrap or Sierra
- You only have one ST tissue section and no replicate APA profile to correlate against
  — the skill's acceptance criteria depend on cross-replicate comparison
- The goal is gene-level spatial expression variability rather than APA usage — use
  gene-expression spatial tools (SPARK, SpatialDE, Trendsceek) on the count matrix
  instead
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: stAPAminer
  role: 'Core toolkit: computeAPAIndex, imputeAPAIndex, createStAPAminerObject, makeStCluster,
    computePearsonIndex, findContrastAPA for quantifying, imputing, clustering and
    correlating replicate APA profiles'
  repo: https://github.com/BMILAB/stAPAminer
- name: scAPAtrap
  role: Used upstream to identify poly(A) sites from mapped reads for each spatially
    barcoded ST replicate
- name: Seurat
  role: Underlies spot clustering (makeStCluster) that defines the MOB layers used
    for within- vs between-layer correlation checks
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:7e4c5846d04f54b0630ef633f082f6bac3936a6eb9991cd571f8f0adf5c5e559
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - SPARK
  - Seurat
  - movAPA
  - org.Mm.eg.db
  - scAPAtrap
  - stAPAminer
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-replicate-apa-profile-correlation

## Summary

A robustness-analysis skill for spatial APA studies: rerun the stAPAminer pipeline (RUD quantification, KNN imputation, spot clustering, DEAPA calling) on independent adjacent-tissue-section replicates and quantify reproducibility by Pearson correlation of APA profiles against a reference replicate (e.g., ST-MOB Replicates 5 and 12 vs Replicate 11).

## When to use

Use when you have spatially barcoded ST poly(A) site data for multiple tissue-section replicates (e.g., ST-MOB Replicates 5, 11, 12) and need to test whether spatial APA patterns and gene-level DEAPA calls reproduce across sections.

## When NOT to use

- Input is single-cell or imaging-based spatial data (MERFISH/seqFISH) — the article states APA sites can only be identified from spatially barcoded ST data using tools like scAPAtrap or Sierra
- You only have one ST tissue section and no replicate APA profile to correlate against — the skill's acceptance criteria depend on cross-replicate comparison
- The goal is gene-level spatial expression variability rather than APA usage — use gene-expression spatial tools (SPARK, SpatialDE, Trendsceek) on the count matrix instead

## Inputs

- Poly(A) site annotation object per replicate (APA.RDA, movAPA PACdataset) from stAPAminer extdata
- Gene-spot spatial expression count matrix per replicate (count.csv)
- Spot barcode coordinate table per replicate (position.txt)
- Reference replicate APA profile (ST-MOB Replicate 11)

## Outputs

- Per-replicate RUD_RAW and KNN-imputed RUD APA-index matrices
- Clustered and layer-labeled stAPAminer objects (GCL, GL, ONL, OPL, MCL)
- Pearson correlations of each replicate's APA profile vs Replicate 11 (~0.87, ~0.86) and within- vs between-layer correlation comparison
- Per-replicate DEAPA gene lists (adjusted P < 0.05, log2FC > 0.5) with reproduced gene-level patterns (e.g., Adarb2 GCL vs ONL)

## How to apply

For each replicate, load the poly(A) site PACdataset (APA.RDA), gene-spot count matrix (count.csv), and spot coordinates (position.txt) from the stAPAminer extdata, then compute raw APA ratios with computeAPAIndex (RUD_RAW). Impute the sparse RUD matrix with imputeAPAIndex using a KNN model (k=10, init=TRUE) that borrows information from the spatial gene expression profile, setting entries still missing after iteration to 0. Build the stAPAminer object with createStAPAminerObject, cluster spots with makeStCluster (resolution=0.7, dims=1:10, k=30, nfeatures=4500), and label the MOB layers (GCL, GL, ONL, OPL, MCL) with findLabels. Assess reproducibility two ways: (1) within-layer vs between-layer Pearson correlations of the imputed APA profiles via computePearsonIndex(source='RUD'), and (2) the correlation of each replicate's APA profile with the reference Replicate 11 profile, expecting ~0.87 (Replicate 5) and ~0.86 (Replicate 12). Finally, call DEAPA genes with findContrastAPA (adjusted P < 0.05, log2FC > 0.5) and confirm that replicate-specific patterns such as Adarb2's differential APA usage between GCL and ONL are recovered in both replicates.

## Related tools

- **stAPAminer** (Core toolkit: computeAPAIndex, imputeAPAIndex, createStAPAminerObject, makeStCluster, computePearsonIndex, findContrastAPA for quantifying, imputing, clustering and correlating replicate APA profiles) — https://github.com/BMILAB/stAPAminer
- **scAPAtrap** (Used upstream to identify poly(A) sites from mapped reads for each spatially barcoded ST replicate)
- **Seurat** (Underlies spot clustering (makeStCluster) that defines the MOB layers used for within- vs between-layer correlation checks)

## Examples

```
stObj <- createStAPAminerObject(RUD,position); stObj <- makeStCluster(stObj,resolution=0.7,dims=1:10,k=30,nfeatures=4500); pearson_RUD <- computePearsonIndex(stObj,"RUD")
```

## Evaluation signals

- Pearson correlation of each replicate's APA profile with the Replicate 11 profile reaches approximately 0.87 (Replicate 5) and 0.86 (Replicate 12)
- After imputation, within-layer spot correlations of imputed APA profiles exceed between-layer correlations (computePearsonIndex(source='RUD'))
- Clustering with resolution=0.7, dims=1:10, k=30, nfeatures=4500 recovers the five MOB layers (GCL, GL, ONL, OPL, MCL) in each replicate
- DEAPA calls at adjusted P < 0.05 and log2FC > 0.5 reproduce known gene-level patterns, e.g., Adarb2 differential APA usage between GCL and ONL in Replicate 12
- Residual missing APA-index entries after KNN imputation are set to 0 so correlation matrices contain no NAs

## Limitations

- Missing APA usage entries that remain after the entire imputation iteration are arbitrarily set to 0, with no method to determine whether an APA ratio matrix entry is truly missing before imputation
- The article notes the need to propose methods that mitigate discrepancies among replicates to identify more reproducible spatial patterns; replicate-level discrepancies are not otherwise corrected
- No version numbers are reported for any of the software tools used, complicating exact reproduction

## Evidence

- [abstract] abstract: "By extending this analysis to two additional replicates of the MOB ST data, we observed that the spatial APA patterns of several genes were reproducible among replicates"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [methods] methods_imputation: "RUD <-imputeAPAIndex(RUD_RAW,count,k=10,init = TRUE)"
- [discussion] discussion_missing_entries: "If there are still missing entries after the entire iteration process, we can directly set their values to 0."
- [discussion] discussion_replicate_discrepancies: "as well as to propose methods that can mitigate discrepancies among replicates to identify more reproducible spatial patterns"
