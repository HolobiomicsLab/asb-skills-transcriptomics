---
name: cell-type-similarity-quartile-stratification
description: Use when evaluating BIRD-predicted differential DH between many cell-type
  pairs and you need to know how prediction–truth correlation (Pearson, all loci vs
  differential loci) depends on the similarity of the two cell types' DH profiles.
when_to_use_negative:
- You need single-sample (not pairwise differential) DH prediction accuracy — use
  direct per-sample prediction–truth correlation instead
- Your comparison involves fewer than ~8–10 cell types, where quartile stratification
  of pairs is unstable
- You lack a precomputed DH-profile similarity measure and cannot compute a defensible
  one — arbitrary strata will not reproduce the reported pattern
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3170
tools:
- name: BIRD_predict
  role: Generates the genome-wide predicted DH matrix (log2(x+1) scale) for each test
    cell type from its gene expression input
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD-data
  role: Source of preprocessed true DNase-seq profiles and train-test distance data
    used for similarity quantification
  repo: https://github.com/WeiqiangZhou/BIRD-data
- name: R
  role: Used to load .rds DH data, compute pairwise differential DH, Pearson correlations,
    and quartile-aggregated strata
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:84c594e2084d5eecac14f50545fbb4b47e99d23497b4b337e2feb9ffae0b7733
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WeiqiangZhou/BIRD
  tools:
  - BIRD
  - BIRD-data
  - BIRD_predict
  - R
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cell-type-similarity-quartile-stratification

## Summary

Stratifies all pairwise comparisons of BIRD test cell types into quartiles of global DH-profile similarity, then aggregates per-pair differential DNase I hypersensitivity (DH) prediction correlations within each quartile to characterize prediction accuracy as a function of cell-type similarity.

## When to use

Use when evaluating BIRD-predicted differential DH between many cell-type pairs and you need to know how prediction–truth correlation (Pearson, all loci vs differential loci) depends on the similarity of the two cell types' DH profiles.

## When NOT to use

- You need single-sample (not pairwise differential) DH prediction accuracy — use direct per-sample prediction–truth correlation instead
- Your comparison involves fewer than ~8–10 cell types, where quartile stratification of pairs is unstable
- You lack a precomputed DH-profile similarity measure and cannot compute a defensible one — arbitrary strata will not reproduce the reported pattern

## Inputs

- BIRD-predicted DH matrix (log2(x+1) transformed) for the 17 test cell types
- True DNase-seq DH profiles for the same 17 test cell types (200bp loci, hg19/hg38)
- Precomputed cell-type similarity (or train-test distance) measure for all cell-type pairs
- R environment for correlation and stratification analysis

## Outputs

- Per-pair Pearson correlations of differential DH across all loci and across differential loci only
- Mean prediction–truth correlation per similarity quartile per locus subset (quartile × {all loci, differential loci} strata)

## How to apply

Load the log2(x+1)-transformed BIRD prediction matrix (produced by BIRD_predict with the human_hg38 or hg19 model) alongside the true DNase-seq profiles for the 17 test cell types. Form all 136 unique cell-type pairs and compute differential DH per 200bp locus as the difference between the two cell types' DH values, separately for predicted and true data. For each pair, compute Pearson correlations across all loci and across the subset of differential loci only. Assign each pair to one of four quartiles using a precomputed cell-type similarity measure (e.g., the train-test distance metric released with BIRD-data), then average correlations within each quartile-by-locus-subset stratum. Validate by checking that the highest-similarity quartile shows the lowest mean correlations (~0.42 all loci, ~0.55 differential loci) while the remaining quartiles reach ~0.60–0.66 and ~0.69–0.75, indicating better differential prediction for more dissimilar pairs.

## Related tools

- **BIRD_predict** (Generates the genome-wide predicted DH matrix (log2(x+1) scale) for each test cell type from its gene expression input) — https://github.com/WeiqiangZhou/BIRD
- **BIRD-data** (Source of preprocessed true DNase-seq profiles and train-test distance data used for similarity quantification) — https://github.com/WeiqiangZhou/BIRD-data
- **R** (Used to load .rds DH data, compute pairwise differential DH, Pearson correlations, and quartile-aggregated strata)

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt
```

## Evaluation signals

- Number of unique cell-type pairs equals n(n−1)/2 = 136 for 17 test cell types
- Highest-similarity quartile mean correlation falls near 0.42 (all loci) and 0.55 (differential loci)
- Remaining quartiles show higher mean correlations in the ~0.60–0.66 and ~0.69–0.75 ranges
- Monotonic pattern: more dissimilar cell-type pairs yield higher differential DH prediction correlations
- Predicted and true differential DH vectors are computed on the same 200bp locus coordinates after id/coordinate matching

## Limitations

- BIRD will not discover new locations of DHSs when applied to a new gene expression sample, so differential analysis is restricted to known loci
- The most dissimilar cell-type pairs show the LOWEST prediction accuracy here, meaning BIRD's differential predictions are weakest precisely where true differences are subtle between similar cell types

## Evidence

- [readme] Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed).: "Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed)."
- [abstract] using a biological sample's transcriptome to predict its genome-wide regulatory element activities measured by DNase I hypersensitivity (DH): "We evaluate the feasibility of using a biological sample's transcriptome to predict its genome-wide regulatory element activities measured by DNase I hypersensitivity (DH)."
- [readme] DNase-seq data loading via R readRDS with location columns: [evidence span withheld — anchor status: external_doc]
- [readme] locus window size: [evidence span withheld — anchor status: external_doc]
- [discussion] BIRD limitation on discovering new DHSs: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
