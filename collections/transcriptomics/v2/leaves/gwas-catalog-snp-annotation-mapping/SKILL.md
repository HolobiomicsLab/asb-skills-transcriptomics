---
name: gwas-catalog-snp-annotation-mapping
description: Use when a PAIRADISE (FDR <= 10%) significant ASAS SNP list from GM12878
  or Geuvadis population data needs to be linked to GWAS-catalog SNPs via LD (1000
  Genomes phase 3) and tested for enrichment against random controls.
when_to_use_negative:
- When the goal is identifying causal regulatory SNPs — PAIRADISE only tests tagging
  SNPs, since causal SNPs may lie within the alternative exon or nearby intronic regions
  and are not tested
- When SNPs of interest lie within or too distant from alternative exons — PAIRADISE
  cannot analyze them using short-read RNA-seq data
- When no replicate/population RNA-seq heterozygous SNP coverage exists (a heterozygous
  SNP outside but near the alternative exon is required for ASAS detection in the
  first place)
edam_operation: http://edamontology.org/operation_0192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: PAIRADISE
  role: Produces the significant ASAS event list (FDR <= 10%) whose SNPs are mapped
    against the GWAS catalog via LD; installed from the Xinglab repository
  repo: github.com/Xinglab/PAIRADISE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:1a142aa47ff0889a6f424ba103b7354b47a773fc24a968888398f1b4f3b89258
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
  edam_operation: http://edamontology.org/operation_0192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gwas-catalog-snp-annotation-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates significant ASAS SNPs detected by PAIRADISE for linkage disequilibrium (r2 > 0.8) with GWAS-catalog trait/disease SNPs using 1000 Genomes phase 3 haplotypes, and tests enrichment over a randomized ±1,000 bp control null. It answers whether population-scale ASAS events connect to known GWAS signals and how many events per dataset do so.

## When to use

Use when a PAIRADISE (FDR <= 10%) significant ASAS SNP list from GM12878 or Geuvadis population data needs to be linked to GWAS-catalog SNPs via LD (1000 Genomes phase 3) and tested for enrichment against random controls.

## When NOT to use

- When the goal is identifying causal regulatory SNPs — PAIRADISE only tests tagging SNPs, since causal SNPs may lie within the alternative exon or nearby intronic regions and are not tested
- When SNPs of interest lie within or too distant from alternative exons — PAIRADISE cannot analyze them using short-read RNA-seq data
- When no replicate/population RNA-seq heterozygous SNP coverage exists (a heterozygous SNP outside but near the alternative exon is required for ASAS detection in the first place)

## Inputs

- PAIRADISE results table of significant ASAS events (FDR <= 10%) with ExonID, IJC/SJC count columns, pval
- GWAS catalog SNP annotations (trait/disease-associated SNPs)
- 1000 Genomes phase 3 haplotype data for LD (r2) computation
- Genomic coordinates of significant ASAS SNPs for sampling ±1,000 bp control windows

## Outputs

- Per-dataset tally of significant ASAS events in high LD (r2 > 0.8) with GWAS-catalog SNPs (GM12878 and CEU/FIN/GBR/TSI/YRI)
- Empirical enrichment p-value per dataset from 10,000 permutations (expected < 1e-4)
- Null distribution of GWAS-overlap counts across the 10,000 randomized control sets

## How to apply

Parse the PAIRADISE results table (default significance FDR <= 10%) to extract significant ASAS SNPs per dataset (GM12878 and Geuvadis CEU/FIN/GBR/TSI/YRI). For each significant ASAS SNP, compute linkage disequilibrium (r2) against nearby GWAS-catalog SNPs using 1000 Genomes phase 3 haplotype data and retain pairs with r2 > 0.8 as linked ASAS-GWAS pairs, tallying counts per dataset. Build the null by generating 10,000 control SNP sets sampled randomly from within ±1,000 bp of the significant ASAS SNPs and computing each set's overlap with GWAS-catalog SNPs. Calculate the empirical enrichment p-value as the fraction of permutations whose overlap count is >= the observed count; significance requires p < 1e-4 in each of the six datasets. The expected output matches the paper's findings: 33 GWAS-linked ASAS events in GM12878 and 52, 35, 18, 22, and 42 events in CEU, FIN, GBR, TSI, and YRI.

## Related tools

- **PAIRADISE** (Produces the significant ASAS event list (FDR <= 10%) whose SNPs are mapped against the GWAS catalog via LD; installed from the Xinglab repository) — github.com/Xinglab/PAIRADISE

## Evaluation signals

- Enrichment p-value < 1e-4 in each of the six datasets across 10,000 permutations
- Per-dataset GWAS-linked ASAS event counts reproduce 52 (CEU), 35 (FIN), 18 (GBR), 22 (TSI), 42 (YRI), and 33 events in GM12878
- All retained ASAS-GWAS pairs satisfy r2 > 0.8 computed on 1000 Genomes phase 3 haplotypes
- Input ASAS list comes from the PAIRADISE output table with default significance FDR <= 10% and the required columns (ExonID, IJC/SJC counts, pval)
- Every permutation control SNP set is sampled within ±1,000 bp of the significant ASAS SNPs

## Limitations

- Causal SNPs affecting splicing may be missed — only tagging SNPs are identified, as causal SNPs located within the alternative exons or nearby intronic regions are not tested by PAIRADISE
- Requires a heterozygous SNP outside the alternative exon but close enough to be detected on the same RNA-seq read
- SNPs located within or too distant from alternative exons cannot be analyzed using short-read RNA-seq data

## Evidence
