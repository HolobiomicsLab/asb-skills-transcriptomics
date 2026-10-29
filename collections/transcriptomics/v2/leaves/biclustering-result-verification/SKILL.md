---
name: biclustering-result-verification
description: Use when you have biclusters (e.g. trend-preserving biclusters from UniBic/runibic
  with significance p < 0.05) and need to verify they are biologically meaningful
  by GO enrichment testing and comparison against published proportions.
when_to_use_negative:
- Input is raw expression data not yet clustered into biclusters — run UniBic (BCUnibic)
  first and filter to p < 0.05 biclusters before enrichment testing
- The question is about comparing biclustering algorithms' accuracy on synthetic data
  rather than verifying biological enrichment of real-dataset results
- Biclusters are narrow column-pattern biclusters from a tool like BicSPAM where trend-preservation
  semantics differ from the row-based UniBic setting
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: runibic
  role: Parallel R/Bioconductor implementation of UniBic that produces the trend-preserving
    biclusters whose gene sets are verified
  repo: https://github.com/athril/runibic
- name: biclust
  role: Package wrapper through which BCUnibic biclustering results are obtained and
    bicluster member matrices are accessed
- name: GOstats
  role: Performs the hypergeometric GO over-representation test on each bicluster's
    gene set against the input-matrix gene universe
- name: R
  role: Runtime environment for runibic, biclust, and GOstats, and for Benjamini–Hochberg
    p-value adjustment and tallying
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:cf09a12e9d712335c9b44371e11ecb9763992a3844e4ecee48823e12d7648bc3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/athril/runibic
  tools:
  - GOstats
  - R
  - UniBic
  - biclust
  - runibic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# biclustering-result-verification

## Summary

Validates biclustering results on gene expression data by testing each bicluster's gene set for Gene Ontology over-representation (hypergeometric test with GOstats, Benjamini–Hochberg adjustment) and confirming recovered counts against reported benchmark values (e.g. 62/151 enriched biclusters, 41.1% average enrichment for UniBic on the GDS datasets).

## When to use

Use when you have biclusters (e.g. trend-preserving biclusters from UniBic/runibic with significance p < 0.05) and need to verify they are biologically meaningful by GO enrichment testing and comparison against published proportions.

## When NOT to use

- Input is raw expression data not yet clustered into biclusters — run UniBic (BCUnibic) first and filter to p < 0.05 biclusters before enrichment testing
- The question is about comparing biclustering algorithms' accuracy on synthetic data rather than verifying biological enrichment of real-dataset results
- Biclusters are narrow column-pattern biclusters from a tool like BicSPAM where trend-preservation semantics differ from the row-based UniBic setting

## Inputs

- Trend-preserving biclusters from a UniBic/runibic run (BiclustResult from BCUnibic(), filtered to significance p < 0.05, output o = 100)
- Input gene expression matrix (e.g. the eight GDS GEO datasets) whose rows define the background gene universe
- Gene Ontology annotation for the organism (consumed by GOstats)

## Outputs

- go_enrichment_summary.csv with per-bicluster enrichment calls, BH-adjusted p-values, enriched-bicluster proportion (62/151), and average enrichment level (41.1%)

## How to apply

Load the trend-preserving biclusters produced by the UniBic run (output o = 100 default, significance p < 0.05) and extract the gene members of each bicluster from the result matrix. For each bicluster, run a hypergeometric GO over-representation test with the GOstats package in R, using the gene universe defined by the input expression matrix as the background. Adjust enrichment p-values with the Benjamini–Hochberg procedure and call a bicluster GO-enriched if any GO term reaches adjusted p < 0.05. Tally the enriched-bicluster proportion and average enrichment level, writing per-bicluster enrichment calls and adjusted p-values to go_enrichment_summary.csv. Rationale: the UniBic paper's central claim is that trend-preserving biclusters found in noisy data are functionally correlated, so GO over-representation of each bicluster's gene set is the biological validation criterion.

## Related tools

- **runibic** (Parallel R/Bioconductor implementation of UniBic that produces the trend-preserving biclusters whose gene sets are verified) — https://github.com/athril/runibic
- **biclust** (Package wrapper through which BCUnibic biclustering results are obtained and bicluster member matrices are accessed)
- **GOstats** (Performs the hypergeometric GO over-representation test on each bicluster's gene set against the input-matrix gene universe)
- **R** (Runtime environment for runibic, biclust, and GOstats, and for Benjamini–Hochberg p-value adjustment and tallying)

## Examples

```
library(runibic); res <- biclust(method=BCUnibic(), BicatYeast); geneSets <- lapply(1:res@Number, function(i) which(res@RowxNumber[,i]))
```

## Evaluation signals

- Enriched-bicluster count reproduces the reported 62 out of 151 UniBic biclusters on the eight GDS datasets
- Average enrichment level reproduces the reported 41.1%
- Every bicluster is called GO-enriched only if at least one GO term reaches Benjamini–Hochberg-adjusted p < 0.05
- go_enrichment_summary.csv contains a per-bicluster row with enrichment call and adjusted p-value for all 151 biclusters
- The hypergeometric background universe equals the gene set of the input expression matrices, not a genome-wide default

## Limitations

- UniBic results are only significant trend-preserving biclusters (p < 0.05); the paper notes the identification of general trend-preserving biclusters remains a challenging problem
- Validation depends on GO annotation version and the chosen background universe; the paper reports no versions or implementation details for the discussion-section results
- The runibic package is a parallel reimplementation in C++11 and may differ from the original sequential UniBic results, so verification targets are tied to the specific UniBic run parameters (o = 100, p < 0.05)

## Evidence

- [methods] Those with p-values less than 0.05 are decreasingly ordered in their significance.: "We calculate the significance value for those trend-preserving biclusters obtained in step 5. Those with p-values less than 0.05 are decreasingly ordered in their significance."
- [discussion] UniBic is more promising in discovery of functionally correlated expression patterns in gene expression data.: "The results on both synthetic and real data sets demonstrate that UniBic is more promising in discovery of functionally correlated expression patterns in gene expression data"
- [readme] res <- biclust(method=BCUnibic(),BicatYeast): "res <- biclust(method=BCUnibic(),BicatYeast)"
- [readme] The algorithm tries to locate trend-preserving biclusters within complex and noisy data.: "The algorithm tries to locate trend-preserving biclusters within complex and noisy data."
