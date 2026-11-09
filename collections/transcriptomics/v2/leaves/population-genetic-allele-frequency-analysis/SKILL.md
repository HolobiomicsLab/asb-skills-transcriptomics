---
name: population-genetic-allele-frequency-analysis
description: Use when a PAIRADISE results table of significant ASAS events (FDR <=
  10%) must be tested for LD-based enrichment against GWAS catalog SNPs and quantified
  per dataset (e.g., GM12878, Geuvadis CEU/FIN/GBR/TSI/YRI).
when_to_use_negative:
- 'You need the causal splicing variant itself: PAIRADISE only tests tagging SNPs
  — ''the causal SNPs could be located within the alternative exons or nearby intronic
  regions and are not tested by PAIRADISE'''
- The ASAS event lacks a heterozygous SNP outside the alternative exon on the same
  RNA-seq read; such events cannot be detected and thus cannot enter the LD overlap
  analysis
- You want direct positional overlap rather than LD-proxy overlap; r2 > 0.8 linking
  via 1000 Genomes phase 3 is intrinsic to this enrichment design
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PAIRADISE
  role: Produces the table of significant ASAS events (FDR <= 10%) whose SNPs are
    tested for LD enrichment against GWAS catalog variants
  repo: https://github.com/Xinglab/PAIRADISE
- name: STAR
  role: Spliced aligner (>= 2.6.0a) used in PAIRADISE Step 1 to map reads to haplotype-modified
    personal genomes, generating the allelic counts underlying ASAS calls
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:d9d4d954f0eda88a1bfaf0e22a651fea289469992539d7abe8b358907290942a
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
  - STAR
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# population-genetic-allele-frequency-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tests whether significant ASAS SNPs identified by PAIRADISE are enriched for linkage disequilibrium (r2 > 0.8) with GWAS-catalog trait/disease-associated SNPs relative to a random ±1,000 bp control null, and tallies the number of ASAS events linked to GWAS SNPs per population dataset.

## When to use

Use when a PAIRADISE results table of significant ASAS events (FDR <= 10%) must be tested for LD-based enrichment against GWAS catalog SNPs and quantified per dataset (e.g., GM12878, Geuvadis CEU/FIN/GBR/TSI/YRI).

## When NOT to use

- You need the causal splicing variant itself: PAIRADISE only tests tagging SNPs — 'the causal SNPs could be located within the alternative exons or nearby intronic regions and are not tested by PAIRADISE'
- The ASAS event lacks a heterozygous SNP outside the alternative exon on the same RNA-seq read; such events cannot be detected and thus cannot enter the LD overlap analysis
- You want direct positional overlap rather than LD-proxy overlap; r2 > 0.8 linking via 1000 Genomes phase 3 is intrinsic to this enrichment design

## Inputs

- PAIRADISE results table of significant ASAS events (FDR <= 10%) with ExonID, IJC/SJC counts, and pval columns
- GWAS catalog SNP list of trait/disease-associated variants
- 1000 Genomes phase 3 haplotype/VCF data for LD (r2) computation
- Sample genotype VCF files (one per chromosome, per sample subdirectory) used by the PAIRADISE pipeline

## Outputs

- List of linked ASAS-GWAS SNP pairs with r2 > 0.8
- Per-dataset counts of ASAS events in high LD with GWAS SNPs (52 CEU, 35 FIN, 18 GBR, 22 TSI, 42 YRI)
- Null distribution of control-SNP overlap counts from 10,000 permutations
- Empirical enrichment p-value per dataset (expected < 1e-4)

## How to apply

Parse the PAIRADISE output table (default significance FDR <= 10%) to extract significant ASAS SNPs per dataset (33 events in GM12878; Geuvadis populations CEU, FIN, GBR, TSI, YRI). Compute linkage disequilibrium (r2) between each significant ASAS SNP and nearby GWAS catalog SNPs using 1000 Genomes phase 3 haplotype data, retaining pairs with r2 > 0.8 as linked ASAS-GWAS pairs, and tally overlap counts per dataset. Build the null distribution by generating 10,000 sets of control SNPs randomly sampled from within ±1,000 bp of the significant ASAS SNPs and counting how many control SNPs in each permutation are LD-linked to GWAS catalog SNPs. Calculate the empirical enrichment p-value as the fraction of permutations whose overlap count is >= the observed count. Validation requires p < 1e-4 in each of the six datasets, with expected linked-event counts of 52, 35, 18, 22, and 42 in CEU, FIN, GBR, TSI, and YRI respectively.

## Related tools

- **PAIRADISE** (Produces the table of significant ASAS events (FDR <= 10%) whose SNPs are tested for LD enrichment against GWAS catalog variants) — https://github.com/Xinglab/PAIRADISE
- **STAR** (Spliced aligner (>= 2.6.0a) used in PAIRADISE Step 1 to map reads to haplotype-modified personal genomes, generating the allelic counts underlying ASAS calls) — https://github.com/alexdobin/STAR

## Examples

```
qsub -cwd -l h_vmem=10G qsub_files/pairadise_step5_stat.qsub
```

## Evaluation signals

- Empirical enrichment p-value from 10,000 permutations must be < 1e-4 in each of the six datasets
- High-LD (r2 > 0.8) ASAS-GWAS linked event counts should reproduce 52, 35, 18, 22, and 42 in CEU, FIN, GBR, TSI, and YRI
- GM12878 single-individual dataset should yield 33 ASAS events entering the overlap analysis
- Input event list must come from PAIRADISE output at the default FDR <= 10% significance threshold
- Control SNPs in the null must be sampled only from within ±1,000 bp of the significant ASAS SNPs

## Limitations

- Requires a heterozygous SNP outside the alternative exon but close enough to the AS event to be detected on the same RNA-seq read; SNPs within or too distant from alternative exons cannot be analyzed
- Only tagging SNPs are tested — causal SNPs affecting splicing regulation within alternative exons or nearby intronic regions are not tested by PAIRADISE
- Results are intentionally conservative; running PAIRADISE with sigma_i1 = sigma_i2 increases significant events (116 to 133 in GM12878), which would change overlap counts
- LD estimation depends on 1000 Genomes phase 3 population panels; events in populations not represented in the panel cannot be assessed

## Evidence

- [results] ASAS SNPs were significantly enriched for LD with GWAS SNPs compared to a random ±1,000 bp control expectation (p value < 1e-4 in each of six datasets): [evidence span withheld — anchor status: not_found]
