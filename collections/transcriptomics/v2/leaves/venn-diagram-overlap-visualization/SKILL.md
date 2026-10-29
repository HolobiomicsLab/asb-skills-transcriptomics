---
name: venn-diagram-overlap-visualization
description: Use when comparing lists of significant features (e.g., ASE SNPs or ASE
  genes) across two or more tissues or conditions and you need intersection counts,
  overlap fractions, and Venn diagrams to report shared vs. tissue-specific results.
when_to_use_negative:
- The input is a single tissue/condition list with nothing to intersect — no multi-set
  overlap exists to compute
- The features being compared are not commensurable across tissues (e.g., different
  genome builds or coordinate systems not lifted over), since Bedtools intersection
  requires matching coordinates
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: Bedtools
  role: Intersects the per-tissue ASE SNP coordinate sets to count SNPs shared across
    all three tissues and tally tissue-specific SNPs
- name: VADT (VCF ASE Detection Tool)
  role: Produces the per-tissue significant ASE SNP lists (meta-analysis, Benjamini–Hochberg
    FDR cutoff 0.05) that feed the overlap computation
  repo: https://github.com/mjtiv/VADT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:9c3e82f9e6041c1d436cc562a5f78413f42e343d6f014590ca86ea52f878f03b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - Bedtools
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# venn-diagram-overlap-visualization

## Summary

Quantify and visualize set overlap of allele-specific expression (ASE) results across multiple tissues at both the SNP and gene level, reproducing the broiler chicken study's 3.7% three-tissue ASE SNP overlap and its rise to 20.1% when ASE SNPs are aggregated to genes.

## When to use

Use when comparing lists of significant features (e.g., ASE SNPs or ASE genes) across two or more tissues or conditions and you need intersection counts, overlap fractions, and Venn diagrams to report shared vs. tissue-specific results.

## When NOT to use

- The input is a single tissue/condition list with nothing to intersect — no multi-set overlap exists to compute
- The features being compared are not commensurable across tissues (e.g., different genome builds or coordinate systems not lifted over), since Bedtools intersection requires matching coordinates

## Inputs

- Per-tissue significant ASE SNP lists (e.g., VADT meta-analysis output VCFs or BED coordinate sets: breast muscle 20,775; abdominal fat 28,190; liver 21,916)
- Ensembl VEP consequence/annotation files mapping SNP coordinates to genes per tissue (Gallus_gallus_5.0 build context)
- Optional gene-level ASE gene lists (breast muscle 4784; abdominal fat 5709; liver 4095) if pre-computed

## Outputs

- Three-tissue overlap counts and fractions at the SNP level (target: 2196 SNPs, 3.7%)
- Single-tissue-specific fraction (~83% of ASE SNPs)
- Three-tissue ASE gene overlap fraction (target: 20.1%)
- Venn diagrams of SNP-level and gene-level overlaps (comparable to Fig. 4 and Fig. S6)

## How to apply

Load the per-tissue significant feature sets (here, VADT meta-analysis significant ASE SNPs: breast muscle 20,775, abdominal fat 28,190, liver 21,916) and intersect their coordinate sets with Bedtools to count multi-tissue and tissue-specific members; the study targeted 2196 shared SNPs (3.7%) with ~83% single-tissue-specific. Then map the significant ASE SNPs to their associated genes using Ensembl VEP consequence/annotations per tissue and recompute overlaps at the gene level (breast muscle 4784 genes, abdominal fat 5709, liver 4095). The key rationale is that aggregating SNPs to genes absorbs tissue-specific SNPs into shared genes, raising the three-tissue overlap from 3.7% to 20.1%. Render both the SNP-level and gene-level overlaps as Venn diagrams (comparable to Fig. 4 and Fig. S6) and validate the computed fractions against the published values.

## Related tools

- **Bedtools** (Intersects the per-tissue ASE SNP coordinate sets to count SNPs shared across all three tissues and tally tissue-specific SNPs)
- **VADT (VCF ASE Detection Tool)** (Produces the per-tissue significant ASE SNP lists (meta-analysis, Benjamini–Hochberg FDR cutoff 0.05) that feed the overlap computation) — https://github.com/mjtiv/VADT

## Examples

```
bedtools intersect -a breast_muscle_ase_snps.bed -b abdominal_fat_ase_snps.bed liver_ase_snps.bed -f 1.0 > three_tissue_shared_snps.bed
```

## Evaluation signals

- Computed three-tissue SNP overlap fraction matches 3.7% (2196 of the union of ASE SNPs)
- Single-tissue-specific SNP fraction approximates the reported ~83%
- After mapping ASE SNPs to genes, the three-tissue gene overlap fraction matches 20.1%
- Per-tissue gene counts are in the expected range: breast muscle 4784, abdominal fat 5709, liver 4095
- Venn diagram region counts sum consistently to the input set sizes (no features lost or double-counted)

## Limitations

- Mapping SNPs to genes depends on annotation quality; the study noted mapping issues with muscle tissue samples, likely due to the current genome build and annotation of muscle-related genes
- Large numbers of SNPs were excluded due to low overlap with the 600K chicken genotyping panel, limiting the informative SNP pool entering the overlap
- Aggregating SNPs to genes inflates apparent overlap; the 20.1% gene-level figure should not be conflated with the 3.7% SNP-level figure

## Evidence

- [abstract] Only 3.7% of ASE SNPs exhibited ASE in all three tissues: "Of all ASE SNPs, only 3.7% exhibited ASE in all three tissues, with ~ 83% showing ASE specific to a single tissue."
- [abstract] Three-tissue ASE gene overlap increased to 20.1%: "When ASE genes (genes containing ASE SNPs) were compared between tissues, the overlap among all three tissues increased to 20.1%."
- [methods] 2196 SNPs shared among all three tissues (Fig. 4): "Only 2196 (3.7%) SNPs were shared among all three tissues (Fig. 4)"
- [readme] Fisher's meta-analysis with Benjamini–Hochberg FDR produces the significant sets: "After getting the final p-value from the meta-analysis an FDR (Benjamini-Hochberg 1995) is implemented on the final meta p-values."
