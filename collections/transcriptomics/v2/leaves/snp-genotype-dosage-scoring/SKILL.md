---
name: snp-genotype-dosage-scoring
description: Use when you need to rank germplasm accessions by resistance-allele burden
  against a pathogen from VCF genotype data plus a curated table of resistance-SNP
  effect sizes, e.g. to reproduce top-5 resistant accession tables for C. gregata,
  F. graminearum, F. virguliforme, M. phaseolina, or P.
when_to_use_negative:
- No VCF genotype data for the germplasm accessions is available (allelic dosages
  cannot be computed)
- No curated resistance-SNP effect-size table exists for the target pathogen (e.g.,
  F. graminearum SNPs are absent from SoySNP50k, so prediction yields a no-call)
- Goal is phenotypic resistance assessment from disease-assay data rather than allele-count-based
  genotype scoring
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: R
  role: Language used to parse the germplasm VCF, compute dosage-by-effect-size resistance
    scores, normalize, and rank accessions (as in the task workflow)
- name: SoyFungiGCN
  role: R/Shiny package from the same study providing the coexpression network resource
    and context for the five-pathogen candidate-gene analysis that produced the resistance
    SNPs
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:8ff0a2a6a7d1c1f25cf906e5da87d41de8b8ec96b44004176a34ca460151fd7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - R
  - SoyFungiGCN
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# snp-genotype-dosage-scoring

## Summary

Scores soybean USDA germplasm accessions for fungal-pathogen resistance by converting VCF genotype calls into allele dosages (S in {0,1,2}) at curated resistance SNP loci, weighting each dosage by pathogen-specific SNP effect sizes, normalizing by the maximum theoretical potential (sum of |effect sizes| x 2), and ranking accessions per pathogen.

## When to use

Use when you need to rank germplasm accessions by resistance-allele burden against a pathogen from VCF genotype data plus a curated table of resistance-SNP effect sizes, e.g. to reproduce top-5 resistant accession tables for C. gregata, F. graminearum, F. virguliforme, M. phaseolina, or P. pachyrhizi.

## When NOT to use

- No VCF genotype data for the germplasm accessions is available (allelic dosages cannot be computed)
- No curated resistance-SNP effect-size table exists for the target pathogen (e.g., F. graminearum SNPs are absent from SoySNP50k, so prediction yields a no-call)
- Goal is phenotypic resistance assessment from disease-assay data rather than allele-count-based genotype scoring

## Inputs

- USDA soybean germplasm collection VCF (genotype calls from Soybase, SoySNP50k platform)
- Curated table of resistance SNP effect sizes per fungal pathogen (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi)
- List of accession identifiers (PI codes) matching the VCF sample names

## Outputs

- Per-accession raw resistance score for each fungal pathogen
- Resistance potential fraction (raw score normalized by maximum theoretical potential)
- Ranked table (e.g. Table 3) of the five most resistant accessions per fungal species
- No-call flag for species lacking resistance SNPs in the genotyping platform (e.g. F. graminearum in SoySNP50k)

## How to apply

Parse the USDA soybean germplasm VCF from Soybase and convert genotype calls into per-accession allele dosages (S in {0,1,2}) at the curated resistance SNP loci for each of the five fungal pathogens. Multiply each accession's per-locus dosage by the corresponding SNP effect size and sum across loci to obtain a raw resistance score per accession per pathogen. Normalize each raw score by the maximum theoretical potential (sum of |effect sizes| times 2) to derive a resistance potential fraction. Rank accessions per species in descending order and report the top five; if resistance SNPs for a species are absent from the genotyping platform (e.g., no F. graminearum SNPs in SoySNP50k), emit a no-call for that species instead of a ranking. Validate against known reference values, e.g. PI594466 with raw score 1020 and potential 0.73 for C. gregata and PI476752/PI479752 with 240/0.75 for M. phaseolina.

## Related tools

- **R** (Language used to parse the germplasm VCF, compute dosage-by-effect-size resistance scores, normalize, and rank accessions (as in the task workflow))
- **SoyFungiGCN** (R/Shiny package from the same study providing the coexpression network resource and context for the five-pathogen candidate-gene analysis that produced the resistance SNPs) — https://github.com/almeidasilvaf/SoyFungiGCN

## Examples

```
remotes::install_github("almeidasilvaf/SoyFungiGCN"); library(SoyFungiGCN)
```

## Evaluation signals

- PI594466 must yield a raw score of 1020 and resistance potential of 0.73 for C. gregata (Table 3 reference value)
- PI479752 must yield raw score 240 and potential 0.75 for M. phaseolina (Table 3 reference value)
- F. graminearum must produce a no-call because no F. graminearum SNPs were found in SoySNP50k
- All resistance potential fractions must lie in [0, 1] since raw scores are normalized by the maximum theoretical potential (sum of |effect sizes| x 2)
- Exactly five accessions are reported per pathogen (except no-call species), ranked in descending score order

## Limitations

- The most resistant accessions do not carry all resistance alleles, so top-ranked genotypes do not reach the maximum theoretical resistance potential and can be further improved in breeding programs or through genetic engineering
- No F. graminearum SNPs were found in SoySNP50k, so resistance potential for that species cannot be predicted (no-call)
- Scores reflect allele counts only, not experimentally validated phenotype or pathogen isolation; SoyFungiGCN is marked lifecycle: experimental

## Evidence

- [abstract] Based on the number of resistance alleles, we selected the five most resistant accessions against each fungal species in the soybean USDA germplasm: "Based on the number of resistance alleles, we selected the five most resistant accessions against each fungal species in the soybean USDA germplasm"
- [abstract] the most resistant accessions do not reach the maximum theoretical resistance potential: "the most resistant accessions do not reach the maximum theoretical resistance potential. Hence, they can be further improved to increase resistance in breeding programs or through genetic engineering"
- [readme] SoyFungiGCN R/Shiny package install: "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
- [intro] we highlighted the five most resistant accessions against each fungal species in the USDA germplasm: "we highlighted the five most resistant accessions against each fungal species in the USDA germplasm, uncovering important information for breeding programs"
