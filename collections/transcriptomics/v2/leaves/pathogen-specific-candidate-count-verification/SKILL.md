---
name: pathogen-specific-candidate-count-verification
description: Use when reproducing or validating candidate gene counts from GWAS SNPs
  plus coexpression modules, e.g. after running cageminer on SoyFungiGCN data with
  defense guide genes and a 2 Mb SNP window.
when_to_use_negative:
- The goal is de novo network construction from raw RNA-seq rather than counting candidates
  from the pre-built SoyFungiGCN coexpression data
- The target organism or pathogen set differs from soybean and its five phytopathogenic
  fungi (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi)
- Different SNP window sizes or scoring thresholds are intended, in which case counts
  cannot be compared to the published 188/56/11/8/3 values
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0203
tools:
- name: SoyFungiGCN
  role: Provides the integrated GWAS + coexpression data (e.g. genes_modules) used
    as the candidate search space for per-pathogen cageminer runs
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: R
  role: Language in which SoyFungiGCN is distributed and the cageminer prioritization/counting
    is executed
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:da3055d505cb99cd2b8b19e6bd84093427e0ff8c70ecd1a1089668318b7c03de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - R
  - SoyFungiGCN
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# pathogen-specific-candidate-count-verification

## Summary

Verifies that the cageminer-based prioritization of soybean candidate resistance genes reproduces the per-pathogen high-confidence candidate counts (188/56/11/8/3) reported for F. virguliforme, F. graminearum, C. gregata, M. phaseolina, and P. pachyrhizi using the SoyFungiGCN package.

## When to use

Use when reproducing or validating candidate gene counts from GWAS SNPs plus coexpression modules, e.g. after running cageminer on SoyFungiGCN data with defense guide genes and a 2 Mb SNP window.

## When NOT to use

- The goal is de novo network construction from raw RNA-seq rather than counting candidates from the pre-built SoyFungiGCN coexpression data
- The target organism or pathogen set differs from soybean and its five phytopathogenic fungi (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi)
- Different SNP window sizes or scoring thresholds are intended, in which case counts cannot be compared to the published 188/56/11/8/3 values

## Inputs

- genes_modules dataset (SoyFungiGCN R package) with GWAS SNP results and RNA-seq-derived coexpression modules
- MapMan defense-related gene annotations from PLAZA 3.0 Dicots (guide genes)
- Significant resistance SNP positions for each of the five fungal pathogens
- cageminer prioritization parameters: 2 Mb window, rpb threshold 0.2

## Outputs

- candidate_counts.csv: per-pathogen count of high-confidence candidate resistance genes
- Ranked high-confidence candidate gene lists for F. virguliforme, F. graminearum, C. gregata, M. phaseolina, and P. pachyrhizi

## How to apply

Load the integrated GWAS and RNA-seq coexpression data shipped with the SoyFungiGCN R package (e.g. the genes_modules dataset) in R. Assign defense-related guide genes using MapMan annotations from PLAZA 3.0 Dicots so candidates are weighted by defense relevance. Restrict the search space to genes within a 2 Mb window around each significant resistance SNP, then run the cageminer prioritization separately for each of the five pathogens with an rpb (point-biserial correlation) threshold of 0.2. Tabulate the number of high-confidence candidates per pathogen in candidate_counts.csv and compare each count to the published values of 188 (F. virguliforme), 56 (F. graminearum), 11 (C. gregata), 8 (M. phaseolina), and 3 (P. pachyrhizi).

## Related tools

- **SoyFungiGCN** (Provides the integrated GWAS + coexpression data (e.g. genes_modules) used as the candidate search space for per-pathogen cageminer runs) — https://github.com/almeidasilvaf/SoyFungiGCN
- **R** (Language in which SoyFungiGCN is distributed and the cageminer prioritization/counting is executed) — https://github.com/almeidasilvaf/SoyFungiGCN

## Examples

```
remotes::install_github("almeidasilvaf/SoyFungiGCN"); library(SoyFungiGCN); data(genes_modules); run_app()
```

## Evaluation signals

- Reproduced counts equal exactly 188, 56, 11, 8, and 3 high-confidence candidates for F. virguliforme, F. graminearum, C. gregata, M. phaseolina, and P. pachyrhizi, respectively
- All reported candidates fall within 2 Mb of at least one significant resistance SNP for the corresponding pathogen
- Every high-confidence candidate exceeds the rpb threshold of 0.2 and is scored against defense-related guide genes
- candidate_counts.csv contains exactly five rows, one per pathogen, with non-negative integer counts

## Limitations

- The SoyFungiGCN package is marked as experimental in its README, so function interfaces may change
- Counts are only comparable when the same guide gene set (MapMan/PLAZA 3.0 Dicots), 2 Mb window, and rpb threshold of 0.2 are used
- No changelog or package version is stated, so data drift in the GitHub package could affect exact count reproduction

## Evidence

- [abstract] Reported counts to reproduce: "We identified 188, 56, 11, 8, and 3 high-confidence candidates for resistance to F. virguliforme, F. graminearum, C. gregata, M. phaseolina and P. pachyrhizi, respectively"
- [intro] Integrated public data objective: "we integrated multiple publicly available RNA-seq and GWAS datasets to identify high-confidence candidate genes for resistance to five phytopathogenic fungi"
- [methods] Package installation method: "You can install the released version of SoyFungiGCN from GitHub with:"
- [readme] Package purpose and runtime: "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
