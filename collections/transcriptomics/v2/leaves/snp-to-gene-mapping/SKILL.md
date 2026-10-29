---
name: snp-to-gene-mapping
description: Use when per-tissue significant ASE SNP coordinate sets are available
  and the goal is to convert SNP-level ASE calls into gene-level ASE calls, or to
  quantify cross-tissue overlap at the gene rather than SNP level.
when_to_use_negative:
- When only SNP-level overlap statistics are needed and gene-level aggregation is
  not the goal — Bedtools coordinate intersection alone suffices
- When SNPs cannot be confidently assigned to genes (e.g., intergenic SNPs lacking
  VEP consequence annotation), since aggregation to ASE genes would be unreliable
- For cross-build comparisons — the paper notes mapping issues with muscle tissue
  samples likely due to the current genome build, and repeated the analysis with the
  Gallus_gallus_6.0 GTF
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Produces the per-tissue meta-analysis significant ASE SNP lists (Benjamini–Hochberg
    FDR cutoff 0.05) that serve as the input to SNP-to-gene mapping
  repo: https://github.com/mjtiv/VADT
- name: Bedtools
  role: Intersects SNP coordinate sets across tissues to establish the SNP-level overlap
    baseline before gene aggregation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:68195ca7be5d8d0e5419ab89486aa718bb787b40ae5a08fef3edbd54178d5fce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - Bedtools
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# snp-to-gene-mapping

## Summary

Maps tissue-specific significant ASE SNPs (detected by VADT meta-analysis) to their associated genes using Ensembl VEP consequence annotations, then aggregates SNP-level overlaps to the gene level to reveal higher cross-tissue concordance (20.1% shared ASE genes vs 3.7% shared ASE SNPs).

## When to use

Use when per-tissue significant ASE SNP coordinate sets are available and the goal is to convert SNP-level ASE calls into gene-level ASE calls, or to quantify cross-tissue overlap at the gene rather than SNP level.

## When NOT to use

- When only SNP-level overlap statistics are needed and gene-level aggregation is not the goal — Bedtools coordinate intersection alone suffices
- When SNPs cannot be confidently assigned to genes (e.g., intergenic SNPs lacking VEP consequence annotation), since aggregation to ASE genes would be unreliable
- For cross-build comparisons — the paper notes mapping issues with muscle tissue samples likely due to the current genome build, and repeated the analysis with the Gallus_gallus_6.0 GTF

## Inputs

- Per-tissue VADT meta-analysis significant ASE SNP lists (e.g., breast muscle 20,775; abdominal fat 28,190; liver 21,916 SNPs) in VCF/coordinate format
- Ensembl VEP consequence/annotation files for Gallus_gallus_5.0 (Ensembl release 86) to assign SNPs to genes

## Outputs

- Per-tissue ASE gene sets (breast muscle 4784, abdominal fat 5709, liver 4095 genes)
- Three-tissue SNP overlap count and fraction (2196 SNPs, 3.7%)
- Three-tissue ASE gene overlap fraction (20.1%)
- Venn diagrams of SNP-level and gene-level cross-tissue overlaps (Fig. 4 / Fig. S6 style)

## How to apply

Start from the per-tissue VADT meta-analysis significant SNP lists (breast muscle 20,775 ASE SNPs, abdominal fat 28,190, liver 21,916) and intersect SNP coordinate sets with Bedtools to establish the SNP-level baseline (target 2196 shared SNPs, 3.7%, with ~83% single-tissue-specific). Then annotate each significant ASE SNP with Ensembl VEP consequence annotations per tissue to assign SNPs to their associated genes, yielding per-tissue ASE gene sets (breast muscle 4784 genes, abdominal fat 5709, liver 4095). Recompute the three-tissue overlap at the gene level; aggregation should raise the shared fraction to ~20.1% because multiple ASE SNPs in one gene consolidate tissue-specific signals. Visualize both SNP-level and gene-level overlaps as Venn diagrams comparable to Fig. 4 and Fig. S6. Validation: the computed three-tissue SNP overlap must match 3.7% (2196 SNPs) and the three-tissue ASE gene overlap must match 20.1%.

## Related tools

- **VADT (VCF ASE Detection Tool)** (Produces the per-tissue meta-analysis significant ASE SNP lists (Benjamini–Hochberg FDR cutoff 0.05) that serve as the input to SNP-to-gene mapping) — https://github.com/mjtiv/VADT
- **Bedtools** (Intersects SNP coordinate sets across tissues to establish the SNP-level overlap baseline before gene aggregation)

## Examples

```
bedtools intersect -a breast_muscle_ase_snps.bed -b abdominal_fat_ase_snps.bed liver_ase_snps.bed -f 1.0 -wa -u > shared_ase_snps.bed
```

## Evaluation signals

- Three-tissue SNP overlap reproduces 2196 shared SNPs, i.e., 3.7% of all ASE SNPs
- Three-tissue ASE gene overlap equals 20.1% after mapping SNPs to genes
- Per-tissue ASE gene counts match 4784 (breast muscle), 5709 (abdominal fat), and 4095 (liver)
- Single-tissue-specific ASE SNPs account for ~83% of all ASE SNPs
- Gene-level shared fraction (20.1%) exceeds SNP-level shared fraction (3.7%), confirming aggregation consolidates tissue-specific SNP signals

## Limitations

- Only 3.7% of ASE SNPs are shared across all three tissues and ~83% are single-tissue-specific, so gene-level conclusions rest on aggregating sparse SNP signals
- Mapping issues with muscle tissue samples, likely due to the current genome build and annotation of muscle-related genes; the analysis was repeated with the Gallus_gallus_6.0 GTF
- Without eQTL data, association of ASE SNPs with biological mechanisms remains speculative

## Evidence

- [methods] Only 2196 (3.7%) SNPs were shared among all three tissues (Fig. 4): "Only 2196 (3.7%) SNPs were shared among all three tissues (Fig. 4)"
- [abstract] Gene overlap increase: "When ASE genes (genes containing ASE SNPs) were compared between tissues, the overlap among all three tissues increased to 20.1%."
- [abstract] Tissue-specific fraction: "Of all ASE SNPs, only 3.7% exhibited ASE in all three tissues, with ~ 83% showing ASE specific to a single tissue."
- [discussion] Genome build mapping issues: "The biggest problem we encountered with our entire analysis pipeline was mapping issues with muscle tissue samples"
