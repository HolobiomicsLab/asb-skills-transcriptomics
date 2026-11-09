---
name: tcga-clinical-data-integration
description: Use when cluster assignments for TCGA pan-cancer samples must be joined
  to clinical survival metadata to determine which cancer-type subtypes show significantly
  distinct survival (e.g., at p<0.01 by log-rank).
when_to_use_negative:
- Input is raw transcriptome expression data with no clinical survival metadata available
  — clustering (e.g., RCC_clus) must precede survival comparison
- The goal is novel subtype discovery rather than clinical validation of existing
  cluster assignments; use RCC clustering (RCC_clus, clusterAttr) instead
- Cluster/clinical sample sizes fall below the minSample threshold for the cancer-type/cluster
  groups, making log-rank tests underpowered
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: RCC
  role: Provides the survival() function that integrates cluster assignments with
    the clinical info file, generates Kaplan-Meier curves, and applies the minSample
    filter
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: R
  role: R environment/package in which the RCC survival analysis workflow is executed
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:cb212d86c8593f2e56b9005cf72e9e3831b32ee9f98ed53f7ec12c049170b10e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MSCTR/RecursiveConsensusClustering
  tools:
  - R
  - RCC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# tcga-clinical-data-integration

## Summary

Integrates TCGA pan-cancer clinical information (survival times, cancer-type attribute) with cluster assignment files to test whether clustering-derived subtypes have significantly distinct Kaplan-Meier survival profiles, as done in the RCC pan-cancer benchmark against tSNE+k-means, hclust, Chen et al., and Hoadley et al. baselines.

## When to use

Use when cluster assignments for TCGA pan-cancer samples must be joined to clinical survival metadata to determine which cancer-type subtypes show significantly distinct survival (e.g., at p<0.01 by log-rank).

## When NOT to use

- Input is raw transcriptome expression data with no clinical survival metadata available — clustering (e.g., RCC_clus) must precede survival comparison
- The goal is novel subtype discovery rather than clinical validation of existing cluster assignments; use RCC clustering (RCC_clus, clusterAttr) instead
- Cluster/clinical sample sizes fall below the minSample threshold for the cancer-type/cluster groups, making log-rank tests underpowered

## Inputs

- Cluster assignment files for one or more clustering methods (RCC, tSNE+k-means, hclust, Chen et al., Hoadley et al.) covering TCGA pan-cancer samples
- TCGA pan-cancer clinical info file with survival times and a cancer-type attribute column
- minSample threshold for excluding underpowered cancer-type/cluster groups

## Outputs

- Per-cancer-type Kaplan-Meier survival curves per cluster
- Log-rank p-values for survival differences across subtypes within each cancer type
- survival_results.csv summarizing, per clustering method, the count of cancer types with significantly distinct survival at p<0.01

## How to apply

Load the cluster assignment files for each method being compared (e.g., RCC, tSNE+k-means, hclust, Chen et al., Hoadley et al.) together with the TCGA pan-cancer clinical info file containing survival times and a cancer-type attribute. For each clustering, call the RCC function survival(x, infoFile, attribute, clus, minSample, outputDir), setting the cancer-type column as the attribute and applying the minSample filter to exclude underpowered cancer-type/cluster groups. Within each cancer type, generate per-cluster Kaplan-Meier survival curves and compute log-rank significance of survival differences across subtypes, then count cancer types whose clusters differ significantly at p<0.01. Validate the resulting counts against the published Table S8 values (10, 3, 5, 0, and 3 cancers with distinct survival at p<0.01 for RCC, tSNE+k-means, hclust, Chen et al., and Hoadley et al., respectively), compiled into survival_results.csv.

## Related tools

- **RCC** (Provides the survival() function that integrates cluster assignments with the clinical info file, generates Kaplan-Meier curves, and applies the minSample filter) — https://github.com/MSCTR/RecursiveConsensusClustering
- **R** (R environment/package in which the RCC survival analysis workflow is executed) — https://github.com/MSCTR/RecursiveConsensusClustering

## Examples

```
survival(x, infoFile, attribute, clus, minSample, outputDir)
```

## Evaluation signals

- Counts of cancer types with significantly distinct survival at p<0.01 reproduce Table S8: 10 for RCC, 3 for tSNE+k-means, 5 for hclust, 0 for Chen et al., 3 for Hoadley et al.
- survival_results.csv is produced in the output directory with one row per clustering method and its cancer-type count
- Kaplan-Meier curves are generated per cluster within each cancer type, and log-rank p-values accompany each cancer-type comparison
- Cancer-type/cluster groups failing the minSample filter are excluded from the log-rank testing
- The same clinical info file and minSample setting are applied consistently across all compared clustering methods

## Limitations

- Underpowered comparisons: cancer-type/cluster groups with fewer than minSample samples must be excluded, limiting which cancers can be tested
- Significance of survival differences depends on the input clustering quality; e.g., Chen et al. yielded 0 cancers with distinct survival at p<0.01 in the published benchmark
- Survival significance (p<0.01) is a thresholded summary; borderline cancer types may shift counts if thresholds or minSample values change

## Evidence

- [methods] RCC survival function signature with minSample filter: "survival(x, infoFile, attribute, clus, minSample, outputDir)"
- [intro] TCGA pan-cancer dataset description: "consider large-scale TCGA pan-cancer dataset5 which includes samples from multiple cancer types (breast, prostate, brain, etc.)"
- [readme] Sample input/output data availability from README: "For sample input and ouput files please download data from: https://www.msctr.org/2019/05/30/recursive-consensus-clustering/"
- [intro] Prior TCGA analyses lacked clinically relevant within-cancer subtypes: "Two major publications that analyzed this transcriptome data were able to largely find clusters with tissue-specific cancers and not the subtypes within each cancer type with clinical relevance"
