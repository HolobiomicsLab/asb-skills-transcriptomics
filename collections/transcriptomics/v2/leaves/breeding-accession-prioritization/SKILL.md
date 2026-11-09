---
name: breeding-accession-prioritization
description: Use when you have accession-level SNP genotype data (e.g., VCF from Soybase
  SoySNP50k) plus a curated table of resistance-SNP effect sizes per pathogen, and
  need to rank breeding accessions by predicted resistance.
when_to_use_negative:
- The target pathogen is F. graminearum but the genotype matrix is limited to SoySNP50k
  — no F. graminearum SNPs were found there, so predictions are impossible and the
  species must be flagged as a no-call instead.
- You need candidate gene prioritization or coexpression exploration rather than accession
  ranking — use the SoyFungiGCN network/web app (genes_modules, hubs, mod_enrich)
  instead of allele-dosage scoring.
- Phenotypic resistance assay data are available for the accessions — direct phenotypic
  ranking is more reliable than SNP-allele-based scoring, which only captures known
  resistance loci.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0622
tools:
- name: SoyFungiGCN
  role: Companion R/Shiny package providing the coexpression network, curated SNP/context
    datasets, and web app through which the resistance accessions and candidate genes
    can be explored; the accession-ranking analysis is implemented in R.
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: R
  role: Language used to parse genotype calls into allele dosages, compute effect-size-weighted
    resistance scores, normalize by theoretical maximum, and rank the top five accessions
    per pathogen.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:5ab07fda8fe2fd4c82a3d34353fccdbbea1b3504a66f2c8d65ccd121aa370813
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - R
  - SoyFungiGCN
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# breeding-accession-prioritization

## Summary

A marker-based scoring workflow that ranks USDA soybean germplasm accessions by resistance allele dosage at GWAS-derived SNP loci for five fungal pathogens (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi), identifying the top-5 most resistant accessions per pathogen and benchmarking them against the maximum theoretical resistance potential.

## When to use

Use when you have accession-level SNP genotype data (e.g., VCF from Soybase SoySNP50k) plus a curated table of resistance-SNP effect sizes per pathogen, and need to rank breeding accessions by predicted resistance.

## When NOT to use

- The target pathogen is F. graminearum but the genotype matrix is limited to SoySNP50k — no F. graminearum SNPs were found there, so predictions are impossible and the species must be flagged as a no-call instead.
- You need candidate gene prioritization or coexpression exploration rather than accession ranking — use the SoyFungiGCN network/web app (genes_modules, hubs, mod_enrich) instead of allele-dosage scoring.
- Phenotypic resistance assay data are available for the accessions — direct phenotypic ranking is more reliable than SNP-allele-based scoring, which only captures known resistance loci.

## Inputs

- USDA soybean germplasm VCF from Soybase (SoySNP50k genotype calls)
- Curated table of resistance SNP effect sizes for each of the five fungal pathogens (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi)

## Outputs

- Ranked list of the five most resistant accessions per fungal species
- Per-accession raw resistance scores per pathogen
- Per-accession resistance potential fraction (score normalized by maximum theoretical potential)
- No-call flag for pathogen/species with insufficient genotype data

## How to apply

Parse the USDA germplasm VCF from Soybase into per-accession allele dosages (S in {0, 1, 2}) at the curated resistance SNP loci, then multiply each dosage by the corresponding SNP effect size from the curated resistance table and sum across loci to get a raw resistance score per accession per fungal species. Normalize each raw score by the maximum theoretical potential (sum of |effect sizes| × 2) to obtain a resistance potential fraction. Rank accessions in descending order and report the top five per pathogen, flagging species with insufficient genotype data (e.g., F. graminearum as a no-call). The rationale is that allele-count-based scoring from published GWAS loci surfaces breeding-ready material: the resulting top accessions are highlighted as important information for breeding programs and genetic engineering.

## Related tools

- **SoyFungiGCN** (Companion R/Shiny package providing the coexpression network, curated SNP/context datasets, and web app through which the resistance accessions and candidate genes can be explored; the accession-ranking analysis is implemented in R.) — https://github.com/almeidasilvaf/SoyFungiGCN
- **R** (Language used to parse genotype calls into allele dosages, compute effect-size-weighted resistance scores, normalize by theoretical maximum, and rank the top five accessions per pathogen.)

## Examples

```
remotes::install_github("almeidasilvaf/SoyFungiGCN")
```

## Evaluation signals

- Reproduces the reported Table 3 reference values: PI594466 with raw score 1020 and potential 0.73 for C. gregata, and PI479752 with 240/0.75 for M. phaseolina.
- F. graminearum yields an explicit no-call rather than a spurious ranking, since no F. graminearum SNPs are present in SoySNP50k.
- All resistance potential fractions fall within [0, 1], consistent with normalization by sum of |effect sizes| × 2.
- Exactly five accessions are reported per pathogen, sorted in descending resistance score.
- Top accessions per pathogen match the allele-count-based selection described in the paper ('Based on the number of resistance alleles').

## Limitations

- The most resistant accessions do not reach the maximum theoretical resistance potential, so the top-5 list is not a ceiling — material can be further improved in breeding programs or through genetic engineering.
- No F. graminearum SNPs were found in SoySNP50k, preventing prediction for that species.
- The SoyFungiGCN R package that packages the resource is marked as experimental lifecycle and has no changelog or stated version.

## Evidence

- [abstract] Top-5 selection per species is based on resistance allele count: "Based on the number of resistance alleles, we selected the five most resistant accessions against each fungal species in the soybean USDA germplasm"
- [abstract] Top accessions fall short of maximum theoretical resistance potential and can be improved: "the most resistant accessions do not reach the maximum theoretical resistance potential"
- [readme] SoyFungiGCN is an R package exposing the resource via a Shiny app: "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
- [methods] Concrete installation invocation for the R tooling: "remotes::install_github("almeidasilvaf/SoyFungiGCN")"
