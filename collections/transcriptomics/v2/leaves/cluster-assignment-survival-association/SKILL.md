---
name: cluster-assignment-survival-association
description: Use when you have per-sample cluster assignments plus clinical survival
  data (survival time and cancer-type attribute) and need to quantify how many cancer
  types have subtypes with significantly distinct survival profiles.
when_to_use_negative:
- Samples lack survival times or a cancer-type attribute in the clinical info file,
  so Kaplan-Meier/log-rank testing is undefined
- The goal is subtype discovery or cluster characterization itself rather than validating
  clusters against clinical outcome — use RCC clustering or clusterAttr/clusterAnnotation
  instead
- Cancer-type/cluster groups are so small that the minSample filter removes them,
  leaving no powered groups for log-rank comparison
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: RCC
  role: Provides the survival(x, infoFile, attribute, clus, minSample, outputDir)
    function that generates Kaplan-Meier curves and log-rank tests per cluster within
    each cancer type
  repo: github.com/MSCTR/RecursiveConsensusClustering
- name: R
  role: Runtime environment; RCC is installed as an R package via devtools::install_github
  repo: github.com/MSCTR/RecursiveConsensusClustering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:9e783f35978fdaa14495a63c0019b6529faa0ec69e8a14844a847b41e7603110
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RCC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cluster-assignment-survival-association

## Summary

Assess whether cluster assignments (RCC, tSNE+k-means, hclust, Chen et al., Hoadley et al.) produce clinically relevant subtypes by generating Kaplan-Meier curves per cluster within each TCGA pan-cancer type and testing survival differences with log-rank tests at p<0.01.

## When to use

Use when you have per-sample cluster assignments plus clinical survival data (survival time and cancer-type attribute) and need to quantify how many cancer types have subtypes with significantly distinct survival profiles.

## When NOT to use

- Samples lack survival times or a cancer-type attribute in the clinical info file, so Kaplan-Meier/log-rank testing is undefined
- The goal is subtype discovery or cluster characterization itself rather than validating clusters against clinical outcome — use RCC clustering or clusterAttr/clusterAnnotation instead
- Cancer-type/cluster groups are so small that the minSample filter removes them, leaving no powered groups for log-rank comparison

## Inputs

- Cluster assignment files for RCC, tSNE+k-means, hclust, Chen et al., and Hoadley et al. clusterings
- TCGA pan-cancer clinical info file with survival times and cancer-type attribute
- minSample threshold to exclude underpowered cancer-type/cluster groups

## Outputs

- Kaplan-Meier survival curves per cluster within each cancer type
- Log-rank test p-values for survival differences across subtypes
- survival_results.csv with per-method counts of cancer types significant at p<0.01

## How to apply

Load the cluster assignment files (RCC, tSNE+k-means, hclust, Chen et al., Hoadley et al.) together with the TCGA pan-cancer clinical info file containing survival times and the cancer-type attribute. For each clustering, call the RCC function survival(x, infoFile, attribute, clus, minSample, outputDir), setting the cancer-type column as the attribute and applying the minSample filter to exclude underpowered cancer-type/cluster groups. The function generates Kaplan-Meier curves per cluster within each cancer type and computes log-rank significance of survival differences across subtypes. Count, per clustering method, the cancer types whose clusters show significantly distinct survival at p<0.01, compile the counts into survival_results.csv, and compare against the published Table S8 values (10, 3, 5, 0, 3 cancers for RCC, tSNE+k-means, hclust, Chen et al., and Hoadley et al., respectively).

## Related tools

- **RCC** (Provides the survival(x, infoFile, attribute, clus, minSample, outputDir) function that generates Kaplan-Meier curves and log-rank tests per cluster within each cancer type) — github.com/MSCTR/RecursiveConsensusClustering
- **R** (Runtime environment; RCC is installed as an R package via devtools::install_github) — github.com/MSCTR/RecursiveConsensusClustering

## Examples

```
survival(x, infoFile, attribute = "cancer_type", clus = x$clus, minSample = minSample, outputDir = "survival_out")
```

## Evaluation signals

- survival_results.csv is produced with one count of significant cancer types (p<0.01) per clustering method
- Counts reproduce Table S8: 10 cancers (RCC), 3 (tSNE+k-means), 5 (hclust), 0 (Chen et al.), 3 (Hoadley et al.)
- Kaplan-Meier curves are generated per cluster within each cancer type, with underpowered cancer-type/cluster groups removed by the minSample filter
- Log-rank significance calls are reported only at p<0.01 for the per-cancer-type comparisons
- RCC assignments yield significantly more survival-distinct cancer subtypes than the baseline clusterings

## Limitations

- minSample filtering excludes underpowered cancer-type/cluster groups, so very rare subtypes cannot be tested for survival differences
- The comparison is limited to within-cancer-type subtype survival; pan-cancer clusters reflecting tissue of origin are not clinically informative in this sense
- RCC's underlying base algorithm (consensus clustering with k-means) carries a computational-time cost, which affects generating the cluster assignments upstream

## Evidence

- [methods] RCC survival function signature: "survival(x, infoFile, attribute, clus, minSample, outputDir)"
- [intro] Prior pan-cancer clustering found tissue-specific rather than clinically relevant clusters: "Two major publications that analyzed this transcriptome data were able to largely find clusters with tissue-specific cancers and not the subtypes within each cancer type with clinical relevance"
- [discussion] RCC recapitulates complex multi-step analyses with few commands: "with just a few commands we can recapitulate findings obtained through multi-step complex analysis"
- [readme] README package installation instructions: "devtools::install_github("MSCTR/RecursiveConsensusClustering")"
