---
name: gene-coexpression-network-inference
description: Use when filtered RNA-seq expression data from stress conditions (e.g.,
  soybean tissues infected with C. gregata, F. graminearum, F. virguliforme, M. phaseolina,
  P. pachyrhizi) must be converted into a coexpression network for downstream gene-module
  and candidate-gene mining.
when_to_use_negative:
- Input expression data has not been filtered; the article specifies a coexpression
  network built from filtered RNA-seq data, so unfiltered matrices risk spurious correlations.
- The analysis goal is a signed network in which positive and negative correlations
  are separated; this pipeline explicitly builds an unsigned network using absolute
  Pearson r values.
- The goal is only to explore the prebuilt soybean-fungal coexpression network rather
  than construct one — in that case use the SoyFungiGCN Shiny app (run_app()) instead
  of re-inference.
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: BioNERO
  role: Performs quantile normalization/expression adjustment of the filtered RNA-seq
    data and assembles the unsigned coexpression network from Pearson correlations
- name: SoyFungiGCN
  role: R/Shiny package distributing the resulting coexpression network and gene-module
    data (genes_modules) for validation and user exploration
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: R
  role: Environment in which BioNERO-based network inference and SoyFungiGCN packaging
    are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:d86aaa987b64e833ee553293ad9c151d70181625b450d3d9ad957726286df7ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - BioNERO
  - R
  - SoyFungiGCN
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gene-coexpression-network-inference

## Summary

Reconstruct a gene coexpression network from RNA-seq expression data by quantile-normalizing and adjusting expression with BioNERO, then computing pairwise gene-gene Pearson correlations to build an unsigned network, as done for soybean infected with five phytopathogenic fungi.

## When to use

Use when filtered RNA-seq expression data from stress conditions (e.g., soybean tissues infected with C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi) must be converted into a coexpression network for downstream gene-module and candidate-gene mining.

## When NOT to use

- Input expression data has not been filtered; the article specifies a coexpression network built from filtered RNA-seq data, so unfiltered matrices risk spurious correlations.
- The analysis goal is a signed network in which positive and negative correlations are separated; this pipeline explicitly builds an unsigned network using absolute Pearson r values.
- The goal is only to explore the prebuilt soybean-fungal coexpression network rather than construct one — in that case use the SoyFungiGCN Shiny app (run_app()) instead of re-inference.

## Inputs

- filtered soybean RNA-seq expression matrix with samples from five fungal-pathogen stress conditions
- BioNERO expression preprocessing functions

## Outputs

- unsigned coexpression network with edge weights equal to Pearson r values (stored as coexpression_network.rds)
- gene-module assignments consistent with genes_modules (SoyFungiGCN)

## How to apply

Load the filtered expression matrix spanning all five fungal-pathogen stress conditions and apply BioNERO's expression preprocessing to quantile-normalize and adjust the data so sample-level distributions are comparable. On the adjusted matrix, compute pairwise gene-gene correlations using Pearson's r as the correlation method. Assemble the network as unsigned in BioNERO, taking absolute correlation values so both positively and negatively correlated gene pairs are retained as edges. Store the resulting network object (e.g., as coexpression_network.rds) and confirm gene-module assignments are consistent with the genes_modules data released in the SoyFungiGCN package.

## Related tools

- **BioNERO** (Performs quantile normalization/expression adjustment of the filtered RNA-seq data and assembles the unsigned coexpression network from Pearson correlations)
- **SoyFungiGCN** (R/Shiny package distributing the resulting coexpression network and gene-module data (genes_modules) for validation and user exploration) — https://github.com/almeidasilvaf/SoyFungiGCN
- **R** (Environment in which BioNERO-based network inference and SoyFungiGCN packaging are executed)

## Examples

```
library(SoyFungiGCN); run_app()
```

## Evaluation signals

- The network was built on quantile-normalized, expression-adjusted data, not the raw filtered matrix.
- The network is declared unsigned and edge weights correspond to Pearson correlation coefficients.
- Downstream gene-module assignments are consistent with the genes_modules data released in the SoyFungiGCN package.
- The expression matrix used spans samples from all five fungal-pathogen stress conditions (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi).
- The network object serializes cleanly (e.g., coexpression_network.rds) and loads into the SoyFungiGCN Shiny app via run_app().

## Limitations

- The SoyFungiGCN package is marked experimental (lifecycle badge: experimental), so the released network artifacts may change without a changelog (the article notes 'No changelog found.').
- The unsigned representation merges positive and negative correlations into a single edge weight via absolute values, losing sign information about the direction of co-regulation.
- Validation against genes_modules is only possible if the SoyFungiGCN package data version matches the one used in the study; no package version is stated in the article.

## Evidence

- [other] After quantile normalization with BioNERO, an unsigned coexpression network was constructed from the adjusted expression data using Pearson's r as the correlation measure.: [evidence span withheld — anchor status: not_found]
- [abstract] The coexpression network generated here is available in a user-friendly web application and R/Shiny package.: "the coexpression network generated here is available in a user-friendly web application"
- [methods] genes_modules is a SoyFungiGCN dataset used for module assignment consistency checks.: "- `genes_modules (SoyFungiGCN)`"
- [readme] SoyFungiGCN is installed from GitHub with remotes and its Shiny app launched with run_app().: "remotes::install_github("almeidasilvaf/SoyFungiGCN")"
