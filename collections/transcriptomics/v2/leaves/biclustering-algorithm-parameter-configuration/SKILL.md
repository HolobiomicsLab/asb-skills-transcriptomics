---
name: biclustering-algorithm-parameter-configuration
description: Use when you need to run UniBic/runibic on a continuous gene expression
  matrix (or discrete data with BCUnibicD) and must set its algorithm parameters (significance
  level, extension error rates, number of output biclusters) correctly or reproduce
  published runs.
when_to_use_negative:
- The input is already discrete/symbolic and you want the continuous-data defaults
  — use BCUnibicD instead of BCUnibic, since the wrapper differs by data type
- The goal is finding narrow biclusters, where BicSPAM was specifically designed and
  outperformed UniBic
- The task requires identifying general trend-preserving biclusters without seeds,
  which the article notes remains a challenging problem
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: runibic
  role: R/Bioconductor package providing the parallel UniBic implementation invoked
    via BCUnibic()/BCUnibicD() through the biclust wrapper
  repo: github.com/athril/runibic
- name: UniBic
  role: Original sequential algorithm whose default parameter set (significance 0.05,
    error rates, p-value cutoff) the configuration reproduces; code and datasets at
    the UniBic SourceForge deposit
  repo: sourceforge.net/projects/unibic
- name: biclust
  role: R package used as the Biclust wrapper to call BCUnibic on the input matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:d825ac08594c7e975a6dc4261dd057c1867d3b3112e7ca30ad0f2df92fb10663
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - UniBic
  - biclust
  - runibic
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# biclustering-algorithm-parameter-configuration

## Summary

Configures the UniBic trend-preserving biclustering algorithm (via the runibic R package) with its documented default parameters — significance level 0.05 for index-matrix partitioning, column-extension error rate r ≤ 0.3, row-extension error rate ≤ 0.15, and p-value < 0.05 for reported biclusters — so that reproducible bicluster discovery can be run on gene expression data.

## When to use

Use when you need to run UniBic/runibic on a continuous gene expression matrix (or discrete data with BCUnibicD) and must set its algorithm parameters (significance level, extension error rates, number of output biclusters) correctly or reproduce published runs.

## When NOT to use

- The input is already discrete/symbolic and you want the continuous-data defaults — use BCUnibicD instead of BCUnibic, since the wrapper differs by data type
- The goal is finding narrow biclusters, where BicSPAM was specifically designed and outperformed UniBic
- The task requires identifying general trend-preserving biclusters without seeds, which the article notes remains a challenging problem

## Inputs

- Continuous gene expression data matrix (e.g. BicatYeast, airway SummarizedExperiment, or synthetic trend-preserving matrices such as 200x150 overlapping-bicluster matrices)
- Discrete/symbolic matrix (for BCUnibicD)
- User-adjustable parameters: significance level (default 0.05), column error rate r ≤ 0.3, row error rate ≤ 0.15, maximum number of biclusters, separation percentage q

## Outputs

- A Biclust-compatible result object containing up to 100 trend-preserving biclusters with p-values < 0.05, ordered decreasingly by significance
- Per-bicluster row and column index sets suitable for downstream relevance/recovery scoring against ground truth

## How to apply

Load the input matrix into R and invoke UniBic through the biclust wrapper using BCUnibic() (continuous data) or BCUnibicD() (discrete data), keeping defaults unless reproducing a specific study. The algorithm first derives an index matrix from the input via row-wise ordering (unisort), then computes an integer k from the significance level, default 0.05, to partition the index matrix, and applies LCS to pairs of rows to seed biclusters. Seeded order-preserving biclusters are extended to approximately trend-preserving biclusters by adding columns one at a time with error rate r ≤ 0.3 and extending rows with error rate ≤ 0.15. Output biclusters are significance-filtered: those with p-values < 0.05 are ordered decreasingly by significance, with up to 100 trend-preserving biclusters returned. Preprocess noisy input data first to alleviate adverse impacts from genes not activated under all conditions, optionally using the separation percentage parameter q (default 15/m, or q = 0.5 for data without separation preprocessing).

## Related tools

- **runibic** (R/Bioconductor package providing the parallel UniBic implementation invoked via BCUnibic()/BCUnibicD() through the biclust wrapper) — github.com/athril/runibic
- **UniBic** (Original sequential algorithm whose default parameter set (significance 0.05, error rates, p-value cutoff) the configuration reproduces; code and datasets at the UniBic SourceForge deposit) — sourceforge.net/projects/unibic
- **biclust** (R package used as the Biclust wrapper to call BCUnibic on the input matrix)

## Examples

```
library(runibic); library(biclust); data(BicatYeast); res <- biclust(method=BCUnibic(), BicatYeast)
```

## Evaluation signals

- All returned biclusters have p-values < 0.05 and are ordered decreasingly by significance, with no more than 100 biclusters output
- Column-extension steps never exceed error rate r ≤ 0.3 and row extension stays ≤ 0.15, as fixed by the UniBic defaults
- The index-matrix partition integer k matches what is computed from the significance level default of 0.05
- Rerunning the configured call on the same matrix (e.g. BicatYeast) yields the same bicluster row/column sets, confirming reproducibility of the parameter configuration

## Limitations

- The identification of general trend-preserving biclusters remains a challenging problem; UniBic relies on finding a seed per bicluster
- UniBic was outperformed by BicSPAM on narrow biclusters, the task BicSPAM was specifically designed for
- Preprocessing is advised to alleviate adverse impacts of data entries whose genes are not activated under all conditions; q = 0.5 is the special value for data without separation preprocessing

## Evidence

- [methods] Default parameter configuration of UniBic: "We calculate an integer k based on the significance (default set to 0.05) of the to-be-identified trend-preserving biclusters"
- [methods] Column extension error rate: "we extend it by first repeatedly adding new columns one at a time with an error rate r ≤ 0.3 until none is available."
- [methods] Output significance filtering: "We calculate the significance value for those trend-preserving biclusters obtained in step 5. Those with p-values less than 0.05 are decreasingly ordered in their significance."
- [readme] runibic functions and installation: "This package provides the following main functions: * `BCUnibic`/`runibic` - parallel UniBic for continuous data * `BCUnibicD` - parallel UniBic for discrete data"
- [intro] Wrapper-based invocation: "We use a `Biclust` package wrapper."
- [methods] Preprocessing rationale: "it is better to preprocess the input data to alleviate the adverse impacts to data entries since corresponding genes are not activated under all conditions"
- [methods] Separation parameter default: "we chose a percentage parameter q with the default value set to 15/m (the value q = 0.5 is specially provided for data without separation preprocessing)"
