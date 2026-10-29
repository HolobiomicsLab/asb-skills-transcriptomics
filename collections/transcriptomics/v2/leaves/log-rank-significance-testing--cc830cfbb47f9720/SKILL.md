---
name: log-rank-significance-testing
description: Use when cluster assignments (e.g., from RCC, k-means, hclust, or published
  subtype labels) must be validated for clinical relevance by testing survival differences
  across clusters within cancer-type strata.
when_to_use_negative:
- Input is transcriptomic expression data without linked clinical survival metadata
  — clustering can be performed but survival testing cannot
- Cancer-type/cluster groups fall below the minSample threshold and lack statistical
  power for log-rank testing
- The goal is validating clusters against non-survival clinical attributes; use clusterAttr
  or clusterAnnotation instead
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: RCC
  role: R package whose survival(x, infoFile, attribute, clus, minSample, outputDir)
    function performs the Kaplan-Meier plotting and log-rank significance testing
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: R
  role: Runtime environment for executing the RCC package and the survival analysis
    workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:b1fed9adc5b86aa588c47e2549a05fc8cac8614b04901c03fbaf35bfbd7f92ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MSCTR/RecursiveConsensusClustering
  tools:
  - R
  - RCC
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# log-rank-significance-testing

## Summary

Assess whether cluster-defined cancer subtypes have significantly distinct survival profiles using Kaplan-Meier curves and log-rank tests via the RCC survival() function, as demonstrated on TCGA pan-cancer data where RCC subtypes showed distinct survival in 10 cancers at p<0.01.

## When to use

Use when cluster assignments (e.g., from RCC, k-means, hclust, or published subtype labels) must be validated for clinical relevance by testing survival differences across clusters within cancer-type strata.

## When NOT to use

- Input is transcriptomic expression data without linked clinical survival metadata — clustering can be performed but survival testing cannot
- Cancer-type/cluster groups fall below the minSample threshold and lack statistical power for log-rank testing
- The goal is validating clusters against non-survival clinical attributes; use clusterAttr or clusterAnnotation instead

## Inputs

- Cluster assignment files for RCC, tSNE + k-means, hclust, Chen et al., and Hoadley et al. clusterings
- TCGA pan-cancer clinical info file with survival times and cancer-type attribute
- minSample threshold for excluding underpowered cancer-type/cluster groups

## Outputs

- Kaplan-Meier survival curves per cluster within each cancer type
- Log-rank p-values for survival differences across subtypes within each cancer type
- survival_results.csv counting cancers with significantly distinct survival (p<0.01) per clustering method

## How to apply

Load the cluster assignment files for each method (RCC, tSNE + k-means, hclust, Chen et al., Hoadley et al.) together with the TCGA pan-cancer clinical info file containing survival times and the cancer-type attribute. For each clustering method, call the RCC function survival(x, infoFile, attribute, clus, minSample, outputDir), setting the cancer-type column as the attribute and applying the minSample filter to exclude underpowered cancer-type/cluster groups. The function generates Kaplan-Meier survival curves per cluster within each cancer type and computes log-rank significance of survival differences across subtypes. Count, per method, the cancer types whose clusters show significantly distinct survival at p<0.01, compile into survival_results.csv, and compare against published Table S8 values (10, 3, 5, 0, 3 cancers for RCC, tSNE+k-means, hclust, Chen et al., Hoadley et al. respectively). The rationale is that clusters reflecting clinically meaningful subtypes should partition patients into groups with distinguishable survival outcomes.

## Related tools

- **RCC** (R package whose survival(x, infoFile, attribute, clus, minSample, outputDir) function performs the Kaplan-Meier plotting and log-rank significance testing) — https://github.com/MSCTR/RecursiveConsensusClustering
- **R** (Runtime environment for executing the RCC package and the survival analysis workflow)

## Examples

```
survival(x, infoFile, attribute = "cancer_type", clus, minSample, outputDir = "survival_output/")
```

## Evaluation signals

- survival_results.csv exists and contains per-method counts of cancers with log-rank p<0.01
- RCC counts should reach 10 cancers with significantly distinct survival, matching Table S8; tSNE+k-means=3, hclust=5, Chen et al.=0, Hoadley et al.=3
- Each KM plot corresponds to clusters within a single cancer type, with groups below minSample excluded
- Log-rank p-values are reported for every evaluated cancer-type/cluster combination in the output directory

## Limitations

- Underpowered cancer-type/cluster groups must be excluded via the minSample filter, or the log-rank test results are unreliable
- RCC's recursive consensus clustering has high computational time, tied to its consensus-clustering-with-k-means base algorithm, which affects the upstream clusters this survival test evaluates

## Evidence

- [methods] survival() signature: "survival(x, infoFile, attribute, clus, minSample, outputDir)"
- [other] pan-cancer survival comparison finding: [evidence span withheld — anchor status: not_found]
- [intro] prior analyses lacking clinically relevant subtypes: "Two major publications that analyzed this transcriptome data were able to largely find clusters with tissue-specific cancers and not the subtypes within each cancer type with clinical relevance"
- [readme] README install instructions: "devtools::install_github("MSCTR/RecursiveConsensusClustering")"
