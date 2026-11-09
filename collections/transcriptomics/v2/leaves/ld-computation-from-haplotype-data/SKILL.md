---
name: ld-computation-from-haplotype-data
description: Use when a set of significant SNPs (e.g., PAIRADISE ASAS events at FDR
  <= 10%) must be tested for enrichment of linkage disequilibrium with GWAS-catalog
  trait/disease SNPs against a random-control null.
when_to_use_negative:
- You want to identify the causal SNP affecting splicing — LD tagging only surfaces
  tagging SNPs, not causal variants, which may lie within alternative exons or nearby
  intronic regions and are not tested by PAIRADISE
- Your SNPs are not covered by 1000 Genomes phase 3 haplotypes or lack nearby GWAS-catalog
  entries, making r2 computation or the enrichment null meaningless
- You need per-SNP association testing rather than a set-level enrichment test against
  a randomized ±1,000 bp control expectation
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: PAIRADISE
  role: Produces the significant ASAS event table (FDR <= 10%) whose SNPs are the
    input set for LD overlap counting against GWAS catalog SNPs
  repo: github.com/Xinglab/PAIRADISE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:514b2b3b30b85e57a0ac45551f69a508f99c7cd93ea17d26ac9286153aa15026
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Xinglab/PAIRADISE
  tools:
  - PAIRADISE
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ld-computation-from-haplotype-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes linkage disequilibrium (r2) between significant ASAS SNPs and GWAS-catalog SNPs using 1000 Genomes phase 3 haplotypes, then tests enrichment of ASAS–GWAS linked pairs against ±1,000 bp random control SNPs via 10,000 permutations. Used to show ASAS events tagged by PAIRADISE are significantly associated with trait/disease GWAS signals.

## When to use

Use when a set of significant SNPs (e.g., PAIRADISE ASAS events at FDR <= 10%) must be tested for enrichment of linkage disequilibrium with GWAS-catalog trait/disease SNPs against a random-control null.

## When NOT to use

- You want to identify the causal SNP affecting splicing — LD tagging only surfaces tagging SNPs, not causal variants, which may lie within alternative exons or nearby intronic regions and are not tested by PAIRADISE
- Your SNPs are not covered by 1000 Genomes phase 3 haplotypes or lack nearby GWAS-catalog entries, making r2 computation or the enrichment null meaningless
- You need per-SNP association testing rather than a set-level enrichment test against a randomized ±1,000 bp control expectation

## Inputs

- PAIRADISE results table of significant ASAS events (FDR <= 10%) per dataset
- GWAS catalog SNP positions (trait/disease-associated SNPs)
- 1000 Genomes phase 3 haplotype data (per population: CEU, FIN, GBR, TSI, YRI)
- Genomic coordinates of significant ASAS SNPs

## Outputs

- Per-dataset counts of ASAS events in high LD (r2 > 0.8) with GWAS catalog SNPs
- Null distribution of control-SNP/GWAS overlap counts from 10,000 permutations
- Empirical enrichment p-value per dataset

## How to apply

Parse the PAIRADISE results table (default significance FDR <= 10%) to obtain significant ASAS SNPs per dataset (GM12878 and Geuvadis populations CEU, FIN, GBR, TSI, YRI). For each significant ASAS SNP, compute LD (r2) with nearby GWAS catalog SNPs using 1000 Genomes phase 3 haplotype data, retaining pairs with r2 > 0.8 as linked ASAS–GWAS pairs and tallying overlaps per dataset. Build the null by drawing 10,000 sets of control SNPs randomly sampled from within ±1,000 bp of the significant ASAS SNPs, computing each permutation's overlap with GWAS catalog SNPs. Calculate the empirical enrichment p-value as the fraction of permutations with overlap count >= the observed count. Rationale: LD tagging links splicing-regulating alleles to GWAS signals even when the tested SNP is not the causal variant, and the ±1,000 bp control set matches the local genomic context of the observed SNPs.

## Related tools

- **PAIRADISE** (Produces the significant ASAS event table (FDR <= 10%) whose SNPs are the input set for LD overlap counting against GWAS catalog SNPs) — github.com/Xinglab/PAIRADISE

## Evaluation signals

- Only SNP pairs with r2 > 0.8 are counted as linked ASAS–GWAS pairs
- Null distribution is built from exactly 10,000 control sets sampled within ±1,000 bp of the significant ASAS SNPs
- Empirical enrichment p-value must be < 1e-4 in each of the six datasets (GM12878 plus Geuvadis populations)
- Per-population overlap counts should match the reported 52, 35, 18, 22, and 42 ASAS events in high LD with GWAS SNPs in CEU, FIN, GBR, TSI, and YRI, respectively, and 33 events in GM12878
- Input ASAS SNP list must derive from the PAIRADISE output table filtered at FDR <= 10%

## Limitations

- LD tagging identifies tagging SNPs only; causal SNPs affecting splicing could be located within the alternative exons or nearby intronic regions and are not tested by PAIRADISE
- The reported counts correspond to the conservative PAIRADISE run (sigma_i1 not restricted to equal sigma_i2); allowing sigma_i1 = sigma_i2 increases significant events (116 to 133 in GM12878), which would change the LD overlap sets
- Enrichment conclusions depend on the ±1,000 bp random-control scheme as the null model; alternative control definitions would yield different p-values

## Evidence

- [methods] Retain SNP pairs with r2 > 0.8 as linked ASAS-GWAS pairs and tally overlap counts for each dataset.: [evidence span withheld — anchor status: not_found]
- [methods] Permutation null and p-value definition: [evidence span withheld — anchor status: not_found]
- [methods] Empirical p-value calculation: [evidence span withheld — anchor status: not_found]
- [results] Reported enrichment finding: [evidence span withheld — anchor status: not_found]
- [results] Per-population overlap counts: [evidence span withheld — anchor status: not_found]
