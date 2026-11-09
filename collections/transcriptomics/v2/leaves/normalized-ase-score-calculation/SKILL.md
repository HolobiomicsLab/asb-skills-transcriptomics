---
name: normalized-ase-score-calculation
description: Use when ASE SNP call sets (e.g., VADT sig_meta_analysis_variants.txt
  per tissue) must be converted to gene-level ASE magnitudes to select strongly biased
  genes for pathway or PPI analyses.
when_to_use_negative:
- When the goal is per-SNP ASE significance rather than gene-level magnitude — VADT's
  meta-analysis p-values already provide that and no normalization is needed
- When tissues have systematically different ASE SNP detection rates unrelated to
  informative SNP coverage, since averaging scores across tissues assumes comparability
- When working with multi-allelic or indel-adjacent variants — these are excluded
  by VADT filtering and would violate the biallelic informative-SNP assumption
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Produces the ASE SNP and informative SNP call sets per tissue via binomial
    test and Benjamini–Hochberg FDR control, from which the normalized scores are
    computed
  repo: https://github.com/mjtiv/VADT
- name: Ensembl Variant Effect Predictor (VEP)
  role: Maps ASE SNPs to Ensembl gene IDs so SNP-level counts can be aggregated per
    gene
- name: STRING
  role: Receives the high-ASE gene set (score ≥ 50% with symbols) for protein-protein
    interaction network and KEGG pathway enrichment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:7be8171846b8c250f4d1270e60cde6f2b2f2f467e4e9726d91d34e1c179a525d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - Ensembl Variant Effect Predictor (VEP)
  - STRING
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# normalized-ase-score-calculation

## Summary

Computes a per-gene normalized ASE score — (number of ASE SNPs / number of informative SNPs) × 100 — per tissue, then averages across tissues to rank genes by strength of allele-specific expression for downstream network/functional analyses.

## When to use

Use when ASE SNP call sets (e.g., VADT sig_meta_analysis_variants.txt per tissue) must be converted to gene-level ASE magnitudes to select strongly biased genes for pathway or PPI analyses.

## When NOT to use

- When the goal is per-SNP ASE significance rather than gene-level magnitude — VADT's meta-analysis p-values already provide that and no normalization is needed
- When tissues have systematically different ASE SNP detection rates unrelated to informative SNP coverage, since averaging scores across tissues assumes comparability
- When working with multi-allelic or indel-adjacent variants — these are excluded by VADT filtering and would violate the biallelic informative-SNP assumption

## Inputs

- VADT significant ASE variant call sets per tissue (sig_meta_analysis_variants.txt for breast muscle, abdominal fat, liver)
- Per-tissue informative SNP call sets (SNPs passing VADT filtering)
- SNP-to-gene mapping (e.g., Ensembl VEP annotation to Ensembl gene IDs)

## Outputs

- Per-gene, per-tissue normalized ASE scores ((ASE SNPs / informative SNPs) × 100)
- Averaged three-tissue global ASE score per gene
- Filtered list of high-ASE genes above threshold (e.g., 344 genes at ≥ 50%, of which 208 had assigned gene symbols)

## How to apply

For each gene in a tissue, count the ASE SNPs (those significant in the VADT meta-analysis, Benjamini–Hochberg FDR adjusted p < 0.05) and the informative SNPs (biallelic SNPs passing VADT filters: quality ≥ 20, read count ≥ 20 per sample, indel exclusion distance of 75 bp), then compute the normalized score as (ASE SNPs / informative SNPs) × 100. Repeat per tissue (breast muscle, abdominal fat, liver), average the three per-tissue scores to obtain a global ASE score, and select genes exceeding a chosen threshold (the paper used ≥ 50%). This normalizes for how thoroughly each gene was testable across tissues, so genes with many informative SNPs are not penalized relative to sparsely covered genes. Retain only genes with assigned symbols before uploading to enrichment/network tools such as STRING.

## Related tools

- **VADT (VCF ASE Detection Tool)** (Produces the ASE SNP and informative SNP call sets per tissue via binomial test and Benjamini–Hochberg FDR control, from which the normalized scores are computed) — https://github.com/mjtiv/VADT
- **Ensembl Variant Effect Predictor (VEP)** (Maps ASE SNPs to Ensembl gene IDs so SNP-level counts can be aggregated per gene)
- **STRING** (Receives the high-ASE gene set (score ≥ 50% with symbols) for protein-protein interaction network and KEGG pathway enrichment)

## Examples

```
python VADT_V1.1.py --File_Name breast_muscle.vcf --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20 --Meta_BH_adj_p_value_cutoff 0.05  # then score = (n_ASE_SNP / n_informative_SNP) * 100 per gene
```

## Evaluation signals

- Three-tissue ASE gene overlap computed from the resulting gene sets equals the reported 20.1% (1715 shared genes)
- Genes with global ASE score ≥ 50% number 344, of which 208 have assigned gene symbols
- Per-tissue scores are bounded 0–100 by construction; check no score exceeds 100 or is negative
- Each ASE SNP counted in the numerator must also appear in the informative SNP denominator set (nested counts)
- Downstream STRING analysis of the selected 208-gene set shows significant PPI enrichment (p < 1.0e-16; 176 nodes, 374 edges, average node degree 4.25)

## Limitations

- The score depends on how many informative SNPs a gene has; genes with few informative SNPs yield coarse-grained scores
- Association of strongly scored ASE genes with translation-related pathways is speculative without eQTL data
- Large numbers of SNPs were excluded due to low overlap with the 600K chicken genotyping panel, which can reduce informative SNP coverage for some genes

## Evidence

- [results] For shared genes, compute the normalized gene ASE score as (ASE SNPs / informative SNPs) x 100 per tissue, average across the three tissues, and select the 344 genes with a global ASE score >= 50%, of which 208 have assigned gene symbols.: [evidence span withheld — anchor status: not_found]
- [intro] VADT filters a VCF, tests for ASE using the binomial test, controls the false discovery rate, and reports significant findings with documented intermediate processing steps.: "streamline the entire ASE analysis process into one self-contained user-friendly program that filters a VCF, tests for ASE using the binomial test, controls for the false discovery rate"
- [abstract] On average ~174,000 SNPs per tissue passed filtering and ~24,000 (~14%) showed ASE.: "On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE."
- [readme] Sample-level filtering removes homozygous, low-coverage, or low-minor-allele-count samples, defining the informative SNP denominator.: "Sample level filtering consists of filtering based on removing all samples that are homozygous, low read count (user parameter), allele count is <1% of the total counts"
