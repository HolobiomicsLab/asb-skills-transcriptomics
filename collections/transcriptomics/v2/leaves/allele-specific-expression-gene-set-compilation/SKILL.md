---
name: allele-specific-expression-gene-set-compilation
description: Use when you have VADT sig_meta_analysis_variants.txt call sets for multiple
  tissues and need gene-level ASE sets, cross-tissue overlap percentages, or a high-ASE-score
  gene list for pathway analysis.
when_to_use_negative:
- When the input is raw VCF data that has not yet been tested for ASE — run VADT (binomial
  test + Benjamini-Hochberg FDR, cutoff 0.05) first to generate sig_meta_analysis_variants.txt
  files
- When SNP-level ASE calls are needed rather than gene-level sets — this skill collapses
  SNPs to genes and loses per-SNP allele-ratio detail
- When the organism/annotation differs from Gallus_gallus_5.0 (Ensembl release 86)
  without re-mapping SNP-to-gene assignments via VEP
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Produces the per-tissue significant ASE SNP call sets (meta-analysis model
    with Fisher's method and Benjamini-Hochberg FDR, adjusted p cutoff 0.05) that
    feed the gene-set compilation
  repo: https://github.com/mjtiv/VADT
- name: Ensembl Variant Effect Predictor (VEP)
  role: Maps significant ASE SNPs to Ensembl gene IDs to build per-tissue ASE gene
    sets
- name: STRING
  role: Receives the 208 symbol-assigned high-ASE-score genes, confirms PPI enrichment
    and returns the top KEGG pathways (ribosomes, metabolic pathways, oxidative phosphorylation)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:e2d8a75832e59101daaea72ffea85f258996445be6107d642314a8d6ba5a7ea8
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# allele-specific-expression-gene-set-compilation

## Summary

Compiles per-tissue ASE SNP call sets from VADT meta-analysis output into gene-level ASE sets, computes cross-tissue gene overlap and a normalized per-gene ASE score, and prepares a high-confidence gene list (with symbols) for downstream PPI/pathway enrichment in STRING. It converts SNP-level ASE calls into a reusable, validated gene-level resource.

## When to use

Use when you have VADT sig_meta_analysis_variants.txt call sets for multiple tissues and need gene-level ASE sets, cross-tissue overlap percentages, or a high-ASE-score gene list for pathway analysis.

## When NOT to use

- When the input is raw VCF data that has not yet been tested for ASE — run VADT (binomial test + Benjamini-Hochberg FDR, cutoff 0.05) first to generate sig_meta_analysis_variants.txt files
- When SNP-level ASE calls are needed rather than gene-level sets — this skill collapses SNPs to genes and loses per-SNP allele-ratio detail
- When the organism/annotation differs from Gallus_gallus_5.0 (Ensembl release 86) without re-mapping SNP-to-gene assignments via VEP

## Inputs

- VADT meta-analysis significant ASE SNP call sets per tissue (sig_meta_analysis_variants.txt for breast muscle, abdominal fat, liver)
- Informative (filtered) SNP counts per gene/tissue for computing normalized ASE scores
- Genome annotation (Gallus_gallus_5.0 build, Ensembl release 86 GTF) for SNP-to-gene mapping via Ensembl VEP
- Gene symbol assignments for STRING upload

## Outputs

- Per-tissue ASE gene sets (4784 breast muscle, 5709 abdominal fat, 4095 liver Ensembl gene IDs)
- Three-tissue shared ASE gene set (1715 genes, 20.1% overlap)
- Global ASE-score-ranked gene list (344 genes with score >= 50%; 208 with symbols)
- STRING PPI network statistics (176 nodes, 374 edges, average node degree 4.25)
- KEGG pathway enrichment table (string_kegg_pathway_enrichment.tsv)

## How to apply

Parse the VADT meta-analysis ASE SNP call sets per tissue (sig_meta_analysis_variants.txt files for breast muscle, abdominal fat and liver) and map ASE SNPs to Ensembl gene IDs via Ensembl VEP, yielding 4784 breast muscle, 5709 abdominal fat and 4095 liver ASE genes. Compute the three-tissue ASE gene overlap and verify it reproduces the reported 20.1% (1715 shared genes). For shared genes, compute the normalized gene ASE score as (ASE SNPs / informative SNPs) x 100 per tissue, average across the three tissues, and select genes with a global ASE score >= 50% (344 genes, of which 208 have assigned gene symbols). Upload the 208 symbol-assigned genes to STRING to confirm PPI enrichment (p < 1.0e-16; 176 nodes, 374 edges, average node degree 4.25) and extract top KEGG pathways — ribosomes (count = 18, FDR p = 8.62e-15), metabolic pathways (count = 32, FDR p = 3.33e-07) and oxidative phosphorylation (count = 11, FDR p = 8.93e-07). Write the KEGG enrichment table to string_kegg_pathway_enrichment.tsv.

## Related tools

- **VADT (VCF ASE Detection Tool)** (Produces the per-tissue significant ASE SNP call sets (meta-analysis model with Fisher's method and Benjamini-Hochberg FDR, adjusted p cutoff 0.05) that feed the gene-set compilation) — https://github.com/mjtiv/VADT
- **Ensembl Variant Effect Predictor (VEP)** (Maps significant ASE SNPs to Ensembl gene IDs to build per-tissue ASE gene sets)
- **STRING** (Receives the 208 symbol-assigned high-ASE-score genes, confirms PPI enrichment and returns the top KEGG pathways (ribosomes, metabolic pathways, oxidative phosphorylation))

## Examples

```
python VADT.py --File_Name breast_muscle.vcf --Output_File_Location ./out/ --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20 --Meta_BH_adj_p_value_cutoff 0.05
```

## Evaluation signals

- Three-tissue ASE gene overlap must equal the reported 20.1% (1715 shared genes)
- High-score gene set after filtering must be 344 genes with global ASE score >= 50%, of which 208 have assigned gene symbols
- STRING PPI enrichment must be significant (p < 1.0e-16) with 176 nodes, 374 edges and average node degree 4.25
- Top KEGG pathway counts and FDR p-values must match reported values: ribosomes (18, 8.62e-15), metabolic pathways (32, 3.33e-07), oxidative phosphorylation (11, 8.93e-07)
- Per-tissue ASE gene counts must match 4784 (breast muscle), 5709 (abdominal fat) and 4095 (liver)

## Limitations

- Without eQTL data, association of ASE SNPs with translation-related pathway enrichment is speculative
- A large number of SNPs were excluded due to low overlap with the 600K chicken genotyping panel, attributed to limitations of commercial genotyping panels
- Muscle tissue samples had mapping issues, likely due to the current genome build and annotation of muscle-related genes; VADT's own validation dataset was not included in the paper

## Evidence

- [abstract] When ASE genes (genes containing ASE SNPs) were compared between tissues, the overlap among all three tissues increased to 20.1%.: "When ASE genes (genes containing ASE SNPs) were compared between tissues, the overlap among all three tissues increased to 20.1%."
- [methods] VADT specifically performs two different types of statistical analysis: "the false discovery rate (FDR) is controlled using the Benjamini–Hochberg p-value adjustment method47 generating a final adjusted p-value"
- [readme] VADT meta-analysis with Fisher's method and BH FDR: "After getting the final p-value from the meta-analysis an FDR (Benjamini-Hochberg 1995) is implemented on the final meta p-values."
- [methods] Reference genome masking rationale: "masked reference genome reduced the reference allele ration from 51.77 to 50.17% in breast muscle, 51.93% to 50.19% in abdominal fat and 51.91% to 50.17% in liver"
