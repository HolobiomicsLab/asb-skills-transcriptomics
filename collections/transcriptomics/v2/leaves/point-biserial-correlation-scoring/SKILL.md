---
name: point-biserial-correlation-scoring
description: Use when you have high-confidence candidate genes linked to a binary
  resistance trait (e.g. GWAS-derived resistant vs susceptible accessions) and need
  an ordered, score-based ranking to pick a small validation set such as a top-10
  per pathogen.
when_to_use_negative:
- When your association variable is continuous-continuous rather than binary-vs-continuous
  — a point-biserial correlation is specific to a dichotomous trait such as resistant/susceptible
  accession status.
- When genes lack hub or transcription-factor annotations in the SoyFungiGCN hubs
  dataset, the kappa multiplier degenerates to 1 and ranking power is reduced.
- When the goal is coexpression network construction or module enrichment (mod_enrich)
  rather than prioritizing/ranking an existing candidate gene set.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: SoyFungiGCN
  role: R package supplying the genes_modules, hubs, and scaled_degree datasets and
    Shiny app used to obtain gene module membership, hub, and TF annotations for scoring
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: R
  role: Language used to compute point-biserial correlations (cor.test) and the final
    score = rpb x kappa, then rank and export the top-10 tables
- name: remotes
  role: Used to install SoyFungiGCN from GitHub before loading the annotation datasets
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:a16b5015a91345df9ccc524757ea03f0159b81c7f000afb24625e6beb764c10b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - R
  - SoyFungiGCN
  - remotes
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# point-biserial-correlation-scoring

## Summary

Ranks prioritized soybean candidate resistance genes by scoring each gene as the product of its point-biserial correlation with the resistance-associated trait and a kappa multiplier derived from hub and transcription-factor status, then selecting the top 10 genes per fungal pathogen.

## When to use

Use when you have high-confidence candidate genes linked to a binary resistance trait (e.g. GWAS-derived resistant vs susceptible accessions) and need an ordered, score-based ranking to pick a small validation set such as a top-10 per pathogen.

## When NOT to use

- When your association variable is continuous-continuous rather than binary-vs-continuous — a point-biserial correlation is specific to a dichotomous trait such as resistant/susceptible accession status.
- When genes lack hub or transcription-factor annotations in the SoyFungiGCN hubs dataset, the kappa multiplier degenerates to 1 and ranking power is reduced.
- When the goal is coexpression network construction or module enrichment (mod_enrich) rather than prioritizing/ranking an existing candidate gene set.

## Inputs

- genes_modules dataset from the SoyFungiGCN R package (gene-to-module membership)
- hubs dataset from SoyFungiGCN (hub and transcription-factor annotations used for kappa)
- scaled_degree dataset from SoyFungiGCN
- Resistance-associated binary trait per accession (resistant vs susceptible, derived from GWAS resistance-allele counts in the USDA soybean germplasm)

## Outputs

- Ranked candidate list per fungal species (gene ID, species, point-biserial correlation, hub/TF status, kappa, final score)
- top10_resistance_candidates.csv with ≤10 rows per species

## How to apply

Load the genes_modules, hubs, and scaled_degree datasets from the SoyFungiGCN R package (installed via remotes::install_github("almeidasilvaf/SoyFungiGCN")). For each candidate gene, compute the point-biserial correlation between its module membership/expression pattern and the resistance-associated trait (binary-continuous association, e.g. with cor.test in R). Assign the kappa multiplier from the hubs dataset annotations: kappa = 2 if the gene is a hub or a transcription factor, kappa = 3 if it is both, otherwise kappa = 1. Compute the final score as the point-biserial correlation times kappa, rank genes within each of the five fungal species (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi), and take the top 10 per species; the underlying high-confidence candidate pools per species were 188 (F. virguliforme), 56 (F. graminearum), 11 (C. gregata), 8 (M. phaseolina), and 3 (P. pachyrhizi).

## Related tools

- **SoyFungiGCN** (R package supplying the genes_modules, hubs, and scaled_degree datasets and Shiny app used to obtain gene module membership, hub, and TF annotations for scoring) — https://github.com/almeidasilvaf/SoyFungiGCN
- **R** (Language used to compute point-biserial correlations (cor.test) and the final score = rpb x kappa, then rank and export the top-10 tables)
- **remotes** (Used to install SoyFungiGCN from GitHub before loading the annotation datasets) — https://github.com/almeidasilvaf/SoyFungiGCN

## Examples

```
library(SoyFungiGCN); data(genes_modules, hubs, scaled_degree); score <- rpb * kappa; top10 <- head(genes[order(-score)], 10)
```

## Evaluation signals

- Ranked genes for each species are a subset of the expected high-confidence candidate pools: 188 (F. virguliforme), 56 (F. graminearum), 11 (C. gregata), 8 (M. phaseolina), 3 (P. pachyrhizi).
- Output CSV (top10_resistance_candidates.csv) contains ≤10 rows per species and includes gene ID, fungal species, and final score fields.
- Every gene's final score equals its point-biserial correlation multiplied by the correct kappa (1, 2, or 3) from the hubs annotation.
- Kappa assignment rule holds: kappa = 2 for hub or TF, kappa = 3 for both hub and TF, kappa = 1 otherwise.
- Ranked lists are comparable to Table 2 of the paper in composition and ordering of candidates per pathogen.

## Limitations

- The score depends on precomputed hub and transcription-factor annotations; errors in the hubs dataset propagate into the kappa multiplier and the ranking.
- Species with very small candidate pools (only 3 candidates for P. pachyrhizi, 8 for M. phaseolina) yield top-10 lists that are effectively the whole candidate set, so ranking provides little prioritization.
- The ranking is relative within each fungal species; scores are not directly comparable across pathogens.
- The SoyFungiGCN package is marked experimental (lifecycle badge) and no changelog or package version is stated, so results may vary across package revisions.

## Evidence

- [methods] Prioritized candidates ranked by rpb x kappa: "We identified 188, 56, 11, 8, and 3 high-confidence candidates for resistance to F. virguliforme, F. graminearum, C. gregata, M. phaseolina and P. pachyrhizi, respectively"
- [abstract] Candidate genes prioritized via integrated GWAS and transcriptomic data: "we integrated publicly available genome-wide association studies and transcriptomic data to prioritize candidate genes associated with resistance"
- [methods] Install SoyFungiGCN with remotes from GitHub: "remotes::install_github("almeidasilvaf/SoyFungiGCN")"
- [methods] SoyFungiGCN provides genes_modules, hubs, scaled_degree datasets: "- `hubs (SoyFungiGCN)`"
- [methods] Run the Shiny app to explore the network: "Once the package is installed, you can run the Shiny app with:"
- [abstract] Five most resistant accessions selected by resistance-allele count: "Based on the number of resistance alleles, we selected the five most resistant accessions against each fungal species in the soybean USDA germplasm"
