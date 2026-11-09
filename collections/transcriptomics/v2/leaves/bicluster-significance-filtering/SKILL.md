---
name: bicluster-significance-filtering
description: Use when a biclustering run (e.g. UniBic on GEO microarray matrices like
  GDS589) has produced candidate trend-preserving biclusters and only statistically
  significant ones should be reported or counted downstream.
when_to_use_negative:
- The input is already a set of statistically validated/filtered biclusters with reported
  p-values — re-filtering would change reported counts.
- The goal is to enumerate ALL trend-preserving biclusters regardless of significance
  (e.g. exploratory seed inspection), not a significance-gated set.
- The clustering method is not UniBic-style trend-preserving biclustering and its
  significance model (based on techniques from reference 19 in the paper) does not
  apply.
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3366
tools:
- name: UniBic
  role: Original sequential algorithm whose step 6 computes significance values and
    retains biclusters with p-values < 0.05
  repo: http://sourceforge.net/projects/unibic/files/?source=navbar
- name: runibic
  role: Parallel C++11/R implementation of UniBic that performs bicluster seeding,
    expanding and significance-based output within its `cluster` function
  repo: https://github.com/athril/runibic
- name: biclust
  role: R package wrapper through which runibic's BCUnibic method is invoked for biclustering
    runs whose results are then filtered by significance
- name: R
  role: Environment in which the filtering-enabled runibic pipeline is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:4ffb8739a0ba1414d41f6bd3e302f1570697c5c0b54860a4b5fa7a6af075c437
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - UniBic
  - biclust
  - runibic
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# bicluster-significance-filtering

## Summary

A statistical filtering step from the UniBic biclustering workflow that computes significance values for trend-preserving biclusters and retains only those with p-values < 0.05, reporting them in decreasing order of significance.

## When to use

Use when a biclustering run (e.g. UniBic on GEO microarray matrices like GDS589) has produced candidate trend-preserving biclusters and only statistically significant ones should be reported or counted downstream.

## When NOT to use

- The input is already a set of statistically validated/filtered biclusters with reported p-values — re-filtering would change reported counts.
- The goal is to enumerate ALL trend-preserving biclusters regardless of significance (e.g. exploratory seed inspection), not a significance-gated set.
- The clustering method is not UniBic-style trend-preserving biclustering and its significance model (based on techniques from reference 19 in the paper) does not apply.

## Inputs

- Set of candidate approximately trend-preserving biclusters produced by UniBic's seed-extension step (step 5)
- Integer gene expression matrix (e.g. granulated GEO microarray matrix A'' from datasets such as GDS589)

## Outputs

- Filtered list of significant trend-preserving biclusters with p-values < 0.05, decreasingly ordered by significance
- Per-dataset counts of significant biclusters (e.g. the total of 151 across eight GEO datasets)

## How to apply

After UniBic's step 5 has extended seeds into approximately trend-preserving biclusters (columns added with error rate ≤ 0.3, rows with error rate ≤ 0.15), calculate a significance value for each bicluster. Retain biclusters whose p-values are less than 0.05 and order the survivors decreasingly by significance, outputting as many trend-preserving biclusters as the user needs. This p-value gate is what turns the raw seed-and-extension output into a defensible bicluster count, e.g. the 151 significant biclusters UniBic reported across the eight GEO datasets (GDS181, GDS589, GDS1406, GDS1451, GDS1490, GDS2520, GDS3715, GDS3716) with preprocessing q = 15/m and r = 15. In practice the filtering happens inside the runibic package's `cluster` function (bicluster seeding and expanding) when invoked through the Biclust wrapper.

## Related tools

- **UniBic** (Original sequential algorithm whose step 6 computes significance values and retains biclusters with p-values < 0.05) — http://sourceforge.net/projects/unibic/files/?source=navbar
- **runibic** (Parallel C++11/R implementation of UniBic that performs bicluster seeding, expanding and significance-based output within its `cluster` function) — https://github.com/athril/runibic
- **biclust** (R package wrapper through which runibic's BCUnibic method is invoked for biclustering runs whose results are then filtered by significance)
- **R** (Environment in which the filtering-enabled runibic pipeline is executed)

## Examples

```
library(runibic); library(biclust); res <- biclust(method=BCUnibic(), BicatYeast)
```

## Evaluation signals

- Every retained bicluster has a computed p-value strictly less than 0.05.
- Retained biclusters are output in decreasing order of significance.
- On the eight GEO datasets (GDS181, GDS589, GDS1406, GDS1451, GDS1490, GDS2520, GDS3715, GDS3716) with q = 15/m and r = 15, the summed count of significant UniBic biclusters should equal 151, matching the reported result.
- Biclusters filtered out at this step correspond exactly to those whose significance values failed the p < 0.05 cutoff in step 6.
- Of the 151 significant biclusters reported, downstream GO-enrichment checks should be consistent with the paper's 62 GO-enriched biclusters (41.1% average enrichment level).

## Limitations

- The significance calculation depends on partitioning the index matrix into k subsets based on a significance level (default 0.05) using techniques from reference 19; changing this default changes which biclusters survive.
- Identification of general trend-preserving biclusters remains a challenging problem, so significance filtering only guarantees statistical, not biological, meaningfulness.
- The discussion section provides no versions, parameters, or implementation details for UniBic, so exact significance computations may need to be verified against the runibic implementation.

## Evidence

- [methods] We calculate the significance value for those trend-preserving biclusters obtained in step 5. Those with p-values less than 0.05 are decreasingly ordered in their significance.: "We calculate the significance value for those trend-preserving biclusters obtained in step 5. Those with p-values less than 0.05 are decreasingly ordered in their significance."
- [readme] This package contains implementation of UniBic biclustering algorithm for gene expression data [Wang2016]: "This package contains implementation of UniBic biclustering algorithm for gene expression data [Wang2016]"
- [readme] cluster - main part of UniBic algorithm (biclusters seeding and expanding): "cluster - main part of UniBic algorithm (biclusters seeding and expanding)"
- [results] UniBic discovered 151 biclusters across the eight real GEO datasets, of which 62 were GO-enriched, yielding the highest average enrichment level (41.1%) among the compared tools.: [evidence span withheld — anchor status: not_found]
- [discussion] the identification of general trend-preserving biclusters remains a challenging problem: "the identification of general trend-preserving biclusters remains a challenging problem"
