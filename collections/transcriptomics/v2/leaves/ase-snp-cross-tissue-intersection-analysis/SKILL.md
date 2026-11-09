---
name: ase-snp-cross-tissue-intersection-analysis
description: Use when you have per-tissue lists of significant ASE SNPs (e.g., VADT
  meta-analysis significant variants with coordinates for two or more tissues) and
  need to quantify SNP-level and gene-level overlap across tissues via Venn/set-intersection
  analysis.
when_to_use_negative:
- You only have a single tissue's ASE results with no second tissue to intersect against
- You need per-SNP significance testing or ASE detection itself — that is VADT's job;
  this skill only analyzes already-significant ASE calls
- Your SNPs lack consistent genome coordinates (same genome build) across tissues,
  making coordinate-based intersection invalid
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Generates the per-tissue significant ASE SNP lists (via binomial test, Fisher's
    method meta-analysis, and Benjamini-Hochberg FDR cutoff 0.05) that are intersected
    across tissues
  repo: https://github.com/mjtiv/VADT
- name: Bedtools
  role: Performs coordinate-based set intersection of per-tissue ASE SNP interval
    sets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:956c6848e2ff2e4de8824b217cfe52f4938d493a8586b8f78cb4475627b503e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - Bedtools
  - VADT (VCF ASE Detection Tool)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ase-snp-cross-tissue-intersection-analysis

## Summary

Quantifies how many allele-specific expression (ASE) SNPs and ASE genes are shared across multiple tissues (e.g., breast muscle, abdominal fat, liver) by intersecting per-tissue significant ASE SNP lists with Bedtools, then re-aggregating SNPs to genes to capture convergent ASE (3.7% of SNPs vs. 20.1% of genes shared across all three tissues in the broiler chicken study).

## When to use

Use when you have per-tissue lists of significant ASE SNPs (e.g., VADT meta-analysis significant variants with coordinates for two or more tissues) and need to quantify SNP-level and gene-level overlap across tissues via Venn/set-intersection analysis.

## When NOT to use

- You only have a single tissue's ASE results with no second tissue to intersect against
- You need per-SNP significance testing or ASE detection itself — that is VADT's job; this skill only analyzes already-significant ASE calls
- Your SNPs lack consistent genome coordinates (same genome build) across tissues, making coordinate-based intersection invalid

## Inputs

- Per-tissue VADT meta-analysis significant ASE SNP lists with genomic coordinates (e.g., breast muscle 20,775; abdominal fat 28,190; liver 21,916 ASE SNPs)
- BED-formatted SNP coordinate files per tissue for set intersection
- Ensembl VEP annotation output (Ensembl release 86 / Gallus_gallus_5.0 or Gallus_gallus_6.0 GTF) mapping SNPs to gene IDs

## Outputs

- Three-tissue shared ASE SNP count and fraction (target: 2,196 SNPs / 3.7%)
- Single-tissue-specific ASE SNP fraction (~83%)
- Per-tissue ASE gene lists (breast muscle 4,784; abdominal fat 5,709; liver 4,095 genes)
- Three-tissue ASE gene overlap fraction (target: 20.1%)
- SNP-level and gene-level Venn diagrams comparable to Fig. 4 and Fig. S6

## How to apply

Load each tissue's significant ASE SNP list from VADT meta-analysis output (in the broiler study: 20,775 ASE SNPs in breast muscle, 28,190 in abdominal fat, 21,916 in liver) and convert coordinates to BED intervals. Intersect the SNP coordinate sets with Bedtools to count SNPs shared across all tissue pairs and all three tissues, recording the single-tissue-specific fraction (~83% in this study) and the three-tissue share (2,196 SNPs = 3.7%). Map the significant ASE SNPs to genes using Ensembl VEP consequence annotations (Gallus_gallus_5.0/6.0 annotations), then recompute the three-tissue overlap at the gene level (breast muscle 4,784 genes, abdominal fat 5,709, liver 4,095), targeting the reported 20.1%. The rationale is that gene-level aggregation reveals convergent ASE masked by SNP-level sparsity, since different SNPs within the same gene can show ASE in different tissues. Visualize both levels as Venn diagrams comparable to Fig. 4 and Fig. S6, and validate that the computed fractions match 3.7% (SNPs) and 20.1% (genes).

## Related tools

- **VADT (VCF ASE Detection Tool)** (Generates the per-tissue significant ASE SNP lists (via binomial test, Fisher's method meta-analysis, and Benjamini-Hochberg FDR cutoff 0.05) that are intersected across tissues) — https://github.com/mjtiv/VADT
- **Bedtools** (Performs coordinate-based set intersection of per-tissue ASE SNP interval sets)

## Examples

```
bedtools intersect -a breast_muscle_ASE_snps.bed -b abdominal_fat_ASE_snps.bed liver_ASE_snps.bed -wa -wb
```

## Evaluation signals

- Computed three-tissue SNP overlap fraction matches the reported 3.7% (2,196 SNPs)
- Computed three-tissue ASE gene overlap fraction matches the reported 20.1%
- Single-tissue-specific ASE SNP fraction falls near ~83%
- Per-tissue ASE gene counts reconcile with reported totals: breast muscle 4,784, abdominal fat 5,709, liver 4,095
- Venn diagram per-tissue totals equal the input significant SNP list sizes (no SNPs lost in BED conversion)

## Limitations

- A large number of SNPs were excluded due to low overlap with the 600K chicken genotyping panel, attributed to limitations of commercial genotyping panels
- Mapping issues with muscle tissue samples (higher 'too short' read fraction, avg. 15.39%) may distort breast muscle ASE call counts relative to fat and liver
- Without eQTL data, biological interpretation of shared ASE SNPs (e.g., links to translation pathways) remains speculative

## Evidence

- [abstract] Three-tissue SNP overlap: "Of all ASE SNPs, only 3.7% exhibited ASE in all three tissues, with ~ 83% showing ASE specific to a single tissue."
- [abstract] Gene-level overlap increase: "When ASE genes (genes containing ASE SNPs) were compared between tissues, the overlap among all three tissues increased to 20.1%."
- [methods] Shared SNP count: "Only 2196 (3.7%) SNPs were shared among all three tissues (Fig. 4)"
- [readme] VADT generates ASE calls: "Program takes in a raw VCF file, filters the data and then performs various statistical analysis for"
