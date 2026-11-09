---
name: accession-resistance-allele-ranking
description: Use when genotype data (e.g., SoySNP50k VCF) for a germplasm collection
  must be scored against curated resistance SNP loci to rank accessions by resistance
  allele burden per pathogen.
when_to_use_negative:
- The goal is candidate-gene prioritization or coexpression network exploration rather
  than accession ranking (use SoyFungiGCN's Shiny app for network exploration instead)
- No genotype data are available for the target pathogen — e.g., F. graminearum SNPs
  are absent from SoySNP50k, so ranking is impossible and should be reported as a
  no-call
- Input is already a ranked resistance table or the question concerns phenotypic (not
  allele-based) resistance measurements
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: R
  role: Implementation language for parsing VCF genotype calls, computing dosage-weighted
    resistance scores, and ranking accessions
- name: SoyFungiGCN
  role: R/Shiny package from the same study providing the coexpression network and
    derived resources; installable from GitHub to support exploration of soybean-fungal
    pathogen interactions
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:c23cdb70b15effd6cccc506daaab1f0ebfd76209a58ef6e380d6b885a51fc951
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - R
  - SoyFungiGCN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# accession-resistance-allele-ranking

## Summary

A reproducible R workflow that ranks USDA soybean germplasm accessions by a resistance score computed from resistance-allele dosages at curated SNP loci for five fungal pathogens (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi), reporting the top five most resistant accessions per pathogen and their fraction of the maximum theoretical resistance potential.

## When to use

Use when genotype data (e.g., SoySNP50k VCF) for a germplasm collection must be scored against curated resistance SNP loci to rank accessions by resistance allele burden per pathogen.

## When NOT to use

- The goal is candidate-gene prioritization or coexpression network exploration rather than accession ranking (use SoyFungiGCN's Shiny app for network exploration instead)
- No genotype data are available for the target pathogen — e.g., F. graminearum SNPs are absent from SoySNP50k, so ranking is impossible and should be reported as a no-call
- Input is already a ranked resistance table or the question concerns phenotypic (not allele-based) resistance measurements

## Inputs

- USDA soybean germplasm genotype VCF from Soybase (SoySNP50k SNP calls)
- Curated table of resistance SNP effect sizes for each of the five fungal pathogens (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi)

## Outputs

- Ranked table (top 5) of most resistant accessions per fungal species with raw resistance scores
- Per-accession resistance potential fraction (score normalized by maximum theoretical potential)
- No-call/flag for pathogen species lacking genotype data (e.g., F. graminearum)

## How to apply

Load the USDA soybean germplasm VCF from Soybase and parse genotype calls into per-accession allele dosages (S in {0,1,2}) at the curated resistance SNP loci. Multiply each accession's per-locus dosage by the corresponding SNP effect size from the curated resistance-SNP table for each of the five fungal pathogens and sum across loci to obtain a raw resistance score per accession per pathogen. Normalize each raw score by the maximum theoretical potential (sum of |effect sizes| × 2) to derive a resistance potential fraction. Rank accessions per fungal species in descending order and report the top five, flagging species with insufficient genotype data. Validate against reference values such as PI594466 (raw score 1020, potential 0.73) for C. gregata and PI479752 (240/0.75) for M. phaseolina.

## Related tools

- **R** (Implementation language for parsing VCF genotype calls, computing dosage-weighted resistance scores, and ranking accessions)
- **SoyFungiGCN** (R/Shiny package from the same study providing the coexpression network and derived resources; installable from GitHub to support exploration of soybean-fungal pathogen interactions) — https://github.com/almeidasilvaf/SoyFungiGCN

## Examples

```
remotes::install_github("almeidasilvaf/SoyFungiGCN"); library(SoyFungiGCN); run_app()
```

## Evaluation signals

- Reproduces reported Table 3 reference values: PI594466 with raw score 1020 and potential 0.73 for C. gregata, and PI479752 with 240/0.75 for M. phaseolina
- F. graminearum yields a no-call, since no F. graminearum SNPs were found in SoySNP50k
- Allele dosages S are restricted to {0, 1, 2} and the normalized potential fraction never exceeds 1 (raw score ≤ sum of |effect sizes| × 2)
- Top-ranked accessions do not reach the maximum theoretical resistance potential, consistent with the paper's finding that the most resistant genotypes lack some resistance alleles

## Limitations

- No F. graminearum SNPs are present in SoySNP50k, so resistance potential cannot be predicted for that species (reported as a no-call)
- The most resistant accessions do not carry all resistance alleles and do not reach the maximum theoretical resistance potential, indicating results are upper-bounded by the curated SNP panel
- The SoyFungiGCN package is marked experimental (lifecycle badge) and no changelog or version is stated for it

## Evidence

- [abstract] Based on the number of resistance alleles, we selected the five most resistant accessions against each fungal species in the soybean USDA germplasm: "Based on the number of resistance alleles, we selected the five most resistant accessions against each fungal species in the soybean USDA germplasm"
- [abstract] Top accessions do not reach maximum potential: "the most resistant accessions do not reach the maximum theoretical resistance potential. Hence, they can be further improved to increase resistance in breeding programs"
- [intro] Highlighting top accessions as an objective: "we highlighted the five most resistant accessions against each fungal species in the USDA germplasm, uncovering important information for breeding programs"
- [methods] SoyFungiGCN installation: "You can install the released version of SoyFungiGCN from GitHub with"
- [readme] README package description: "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
