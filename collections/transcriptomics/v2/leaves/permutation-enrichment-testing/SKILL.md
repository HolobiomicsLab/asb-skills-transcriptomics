---
name: permutation-enrichment-testing
description: Use when a set of significant events (e.g., ASAS SNPs at FDR <= 10%)
  needs to be tested for enrichment of overlap with an external annotation (e.g.,
  GWAS catalog SNPs) against a matched random null.
when_to_use_negative:
- The ASAS SNP itself cannot be tested for LD when it lies within the alternative
  exon or too far from it — PAIRADISE only detects tagging SNPs heterozygous outside
  but near the alternative exon, so causal SNPs inside exons are not captured
- The input events lack genomic coordinates or the population lacks matching 1000
  Genomes haplotype data for r2 estimation
- The question is direct association testing (e.g., sQTL testing) rather than enrichment
  of overlap with an external catalog
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: PAIRADISE
  role: Produces the table of significant ASAS events (FDR <= 10%) whose SNPs are
    tested for GWAS LD enrichment
  repo: https://github.com/Xinglab/PAIRADISE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:02fe0bebbbe5dbf33a7f2a8aba9cbc14696faaa898748bb6ac2556b5161d043d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Xinglab/PAIRADISE
  repo_url: https://github.com/Xinglab/PAIRADISE
  tools:
  - PAIRADISE
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# permutation-enrichment-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An empirical permutation-based enrichment test that assesses whether significant ASAS SNPs identified by PAIRADISE are overrepresented for linkage disequilibrium (r2 > 0.8) with GWAS catalog trait/disease SNPs, relative to randomly sampled ±1,000 bp control SNPs using 1000 Genomes phase 3 haplotypes.

## When to use

Use when a set of significant events (e.g., ASAS SNPs at FDR <= 10%) needs to be tested for enrichment of overlap with an external annotation (e.g., GWAS catalog SNPs) against a matched random null.

## When NOT to use

- The ASAS SNP itself cannot be tested for LD when it lies within the alternative exon or too far from it — PAIRADISE only detects tagging SNPs heterozygous outside but near the alternative exon, so causal SNPs inside exons are not captured
- The input events lack genomic coordinates or the population lacks matching 1000 Genomes haplotype data for r2 estimation
- The question is direct association testing (e.g., sQTL testing) rather than enrichment of overlap with an external catalog

## Inputs

- PAIRADISE results table of significant ASAS events (FDR <= 10%) per dataset
- GWAS catalog SNP positions (trait/disease-associated variants)
- 1000 Genomes phase 3 haplotype data for LD (r2) computation
- Genomic coordinates of significant ASAS SNPs (for ±1,000 bp control sampling)

## Outputs

- Count of ASAS events in high LD (r2 > 0.8) with GWAS catalog SNPs per dataset (e.g., 52 CEU, 35 FIN, 18 GBR, 22 TSI, 42 YRI; 33 events in GM12878)
- Null distribution of GWAS-overlap counts across 10,000 permutations
- Empirical enrichment p-value per dataset (expected < 1e-4)

## How to apply

Parse the PAIRADISE results table (FDR <= 10%) to extract significant ASAS SNPs per dataset (GM12878 and Geuvadis populations CEU, FIN, GBR, TSI, YRI). Compute LD (r2) between each significant ASAS SNP and nearby GWAS catalog SNPs using 1000 Genomes phase 3 haplotype data, retaining pairs with r2 > 0.8 as linked ASAS-GWAS pairs and tallying overlap counts per dataset. Generate 10,000 sets of control SNPs randomly sampled from within ±1,000 bp of the significant ASAS SNPs, compute each permutation's overlap with GWAS catalog SNPs to build a null distribution, and calculate the empirical enrichment p-value as the fraction of permutations with overlap count >= the observed count. This tests whether ASAS SNPs are significantly enriched for LD with GWAS signals beyond what genomic proximity alone would explain.

## Related tools

- **PAIRADISE** (Produces the table of significant ASAS events (FDR <= 10%) whose SNPs are tested for GWAS LD enrichment) — https://github.com/Xinglab/PAIRADISE

## Examples

```
python scripts/run_preprocess.py CEU.txt CEU ./  # then, after Step 5, tally r2>0.8 LD links of FDR<=10% ASAS SNPs to GWAS catalog SNPs vs 10,000 ±1,000 bp controls
```

## Evaluation signals

- Empirical enrichment p-value must be < 1e-4 in each of the six datasets (GM12878, CEU, FIN, GBR, TSI, YRI) across 10,000 permutations
- Per-dataset overlap counts should match reported values: 33 events in GM12878 and 52, 35, 18, 22, 42 ASAS events in high LD in CEU, FIN, GBR, TSI, YRI
- Only SNP pairs with r2 > 0.8 should be retained as linked ASAS-GWAS pairs
- Control SNPs must be sampled from within ±1,000 bp of the significant ASAS SNPs; verify the null distribution spans the observed range

## Limitations

- PAIRADISE requires a heterozygous SNP outside the alternative exon but close enough to be detected on the same RNA-seq read; SNPs located within or too distant from alternative exons cannot be analyzed
- Only tagging SNPs are identified — causal SNPs affecting splicing may lie within alternative exons or nearby intronic regions and are not tested by PAIRADISE
- Results are conservative; relaxing sigma_i1 = sigma_i2 would increase significant events (e.g., 116 to 133 in GM12878), changing the SNP set entering the enrichment test

## Evidence

- [results] ASAS SNPs were significantly enriched for LD with GWAS SNPs compared to a random ±1,000 bp control expectation (p value < 1e-4 in each of six datasets): [evidence span withheld — anchor status: not_found]
- [results] Per-dataset overlap counts: [evidence span withheld — anchor status: not_found]
- [results] LD threshold for linked pairs: [evidence span withheld — anchor status: not_found]
