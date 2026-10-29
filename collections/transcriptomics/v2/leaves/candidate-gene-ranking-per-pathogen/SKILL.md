---
name: candidate-gene-ranking-per-pathogen
description: Use when a set of high-confidence candidate genes per pathogen species
  must be converted into an ordered per-species ranked list (e.g., top-10 shortlist)
  using network/annotation-based weighting and prior GWAS-expression association scores.
when_to_use_negative:
- When the goal is identifying candidate genes de novo from GWAS SNPs and coexpression
  modules rather than ranking an existing candidate set
- When the species of interest is not one of the five soybean pathogenic fungi covered
  by SoyFungiGCN (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi)
- When no hub/transcription-factor annotations are available to assign the kappa multiplier,
  since S cannot be computed without kappa
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0203
tools:
- name: SoyFungiGCN
  role: Provides the genes_modules, hubs, and scaled_degree datasets and the Shiny
    app for exploring the soybean-fungus coexpression network underlying the ranking
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: remotes
  role: Installs the SoyFungiGCN package from GitHub so its datasets can be loaded
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: R
  role: Language in which the point-biserial correlation, kappa assignment, and ranking
    are computed
- name: Shiny
  role: Web application framework in which the coexpression network is made explorable
    as a public resource
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:f4f69448620880634c1100a346423456ea73873ba7e179c565ad7859718427e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - R
  - Shiny
  - SoyFungiGCN
  - remotes
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# candidate-gene-ranking-per-pathogen

## Summary

Ranks high-confidence soybean resistance gene candidates per fungal pathogen by multiplying each gene's point-biserial correlation (rpb) with a kappa multiplier derived from hub/transcription-factor status, then selects the top 10 genes per species for experimental validation.

## When to use

Use when a set of high-confidence candidate genes per pathogen species must be converted into an ordered per-species ranked list (e.g., top-10 shortlist) using network/annotation-based weighting and prior GWAS-expression association scores.

## When NOT to use

- When the goal is identifying candidate genes de novo from GWAS SNPs and coexpression modules rather than ranking an existing candidate set
- When the species of interest is not one of the five soybean pathogenic fungi covered by SoyFungiGCN (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi)
- When no hub/transcription-factor annotations are available to assign the kappa multiplier, since S cannot be computed without kappa

## Inputs

- genes_modules dataset (SoyFungiGCN): gene-to-module membership table
- hubs dataset (SoyFungiGCN): hub and transcription-factor annotations used to set kappa
- scaled_degree dataset (SoyFungiGCN): node degree information
- Resistance-associated trait labels (resistant vs. susceptible accessions) from USDA soybean germplasm GWAS data
- High-confidence candidate gene sets per species (188 F. virguliforme, 56 F. graminearum, 11 C. gregata, 8 M. phaseolina, 3 P. pachyrhizi)

## Outputs

- Ordered candidate list per fungal species (ranked by S = rpb x kappa)
- top10_resistance_candidates.csv with columns: gene ID, fungal species, point-biserial correlation, hub/TF status, kappa, final score

## How to apply

Load the genes_modules, hubs, and scaled_degree datasets from the SoyFungiGCN R package, which is installed from GitHub with remotes::install_github("almeidasilvaf/SoyFungiGCN"). For each candidate gene, compute the point-biserial correlation between its module membership/expression pattern and the resistance-associated trait (binary-continuous association, e.g., cor.test in R). Assign the kappa multiplier from the hubs dataset annotations: kappa = 2 if the gene is a hub or a transcription factor, kappa = 3 if it is both, otherwise kappa = 1. Compute the final score S = rpb x kappa and rank genes within each of the five fungal species (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi), then select the top 10 per species and export the ranked table.

## Related tools

- **SoyFungiGCN** (Provides the genes_modules, hubs, and scaled_degree datasets and the Shiny app for exploring the soybean-fungus coexpression network underlying the ranking) — https://github.com/almeidasilvaf/SoyFungiGCN
- **remotes** (Installs the SoyFungiGCN package from GitHub so its datasets can be loaded) — https://github.com/almeidasilvaf/SoyFungiGCN
- **R** (Language in which the point-biserial correlation, kappa assignment, and ranking are computed)
- **Shiny** (Web application framework in which the coexpression network is made explorable as a public resource) — https://github.com/almeidasilvaf/SoyFungiGCN

## Examples

```
remotes::install_github("almeidasilvaf/SoyFungiGCN"); library(SoyFungiGCN); data(genes_modules); data(hubs); score <- rpb * kappa
```

## Evaluation signals

- Each ranked gene appears in the corresponding full high-confidence candidate set for its species (188 F. virguliforme, 56 F. graminearum, 11 C. gregata, 8 M. phaseolina, 3 P. pachyrhizi)
- top10_resistance_candidates.csv contains at most 10 rows per species and includes gene, species, and score fields
- Every kappa value is in {1, 2, 3} and matches the gene's hub/TF status from the hubs dataset (2 = hub or TF, 3 = both, 1 = neither)
- Final scores reproduce exactly as point-biserial correlation x kappa for every row (no residual ordering jitter)
- Ranked lists are ordered per species (no cross-species mixing of ranks) and scores are non-decreasing down each list

## Limitations

- The ranking inherits the article's scope: it applies to the five soybean-pathogenic fungi and the USDA-germplasm-derived resistance trait; other species/traits require re-derivation
- Kappa depends on hub and transcription-factor annotations in SoyFungiGCN; missing or noisy annotations directly change the multiplier and thus the ranking
- Supplementary figures and tables referenced by the article were not accessible in the provided text, limiting direct reproduction of per-gene values

## Evidence

- [abstract] High-confidence candidate counts per species: "We identified 188, 56, 11, 8, and 3 high-confidence candidates for resistance to F. virguliforme, F. graminearum, C. gregata, M. phaseolina and P. pachyrhizi, respectively"
- [intro] Prioritization strategy of combining evidence: "This strategy consists in investigating the transcriptional patterns of all the genes near a significant SNP"
- [readme] Package installation command: "remotes::install_github("almeidasilvaf/SoyFungiGCN")"
- [readme] Package purpose: "SoyFungiGCN is an R package that contains a Shiny app to let users"
