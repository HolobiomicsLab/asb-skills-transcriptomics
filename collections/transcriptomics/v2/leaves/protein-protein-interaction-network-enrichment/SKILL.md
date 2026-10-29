---
name: protein-protein-interaction-network-enrichment
description: Use when you have a small, validated gene list (ideally with gene symbols)
  and need to test whether its members form a non-random interaction network and which
  KEGG pathways are enriched, as done for shared broiler ASE genes.
when_to_use_negative:
- The gene list is not validated (e.g., the three-tissue overlap has not been confirmed
  at the expected 20.1% / 1715 shared genes), so network results would be uninterpretable
- Genes lack assigned symbols and cannot be reliably mapped to STRING identifiers
- The question is SNP-level ASE overlap rather than gene-level functional enrichment
  (the study found only 3.7% of ASE SNPs shared across tissues, versus 20.1% of ASE
  genes)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0203
tools:
- name: STRING
  role: Web server used to build the PPI network from 208 gene symbols and extract
    top KEGG pathways with FDR-adjusted p-values
- name: Ensembl Variant Effect Predictor (VEP)
  role: Maps ASE SNPs from VADT meta-analysis call sets to Ensembl gene IDs to construct
    the gene list uploaded to STRING
- name: VADT (VCF ASE Detection Tool)
  role: Upstream tool producing the sig_meta_analysis_variants.txt ASE SNP call sets
    per tissue that define the gene input
  repo: https://github.com/mjtiv/VADT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:7ed152dcf730a1fdacd55debe529322c612b15f8e05da4957d1c6686e603b77e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - Ensembl Variant Effect Predictor (VEP)
  - STRING
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# protein-protein-interaction-network-enrichment

## Summary

Build a STRING protein-protein interaction network from a curated list of genes with symbols and extract KEGG pathway enrichments to identify shared functional biology (e.g., ribosome/translation pathways) among candidate genes.

## When to use

Use when you have a small, validated gene list (ideally with gene symbols) and need to test whether its members form a non-random interaction network and which KEGG pathways are enriched, as done for shared broiler ASE genes.

## When NOT to use

- The gene list is not validated (e.g., the three-tissue overlap has not been confirmed at the expected 20.1% / 1715 shared genes), so network results would be uninterpretable
- Genes lack assigned symbols and cannot be reliably mapped to STRING identifiers
- The question is SNP-level ASE overlap rather than gene-level functional enrichment (the study found only 3.7% of ASE SNPs shared across tissues, versus 20.1% of ASE genes)

## Inputs

- List of gene symbols (n = 208 in the study) derived from shared high-ASE genes (global ASE score >= 50% across breast muscle, abdominal fat, liver)
- Prior ASE gene sets per tissue (4784 breast muscle, 5709 abdominal fat, 4095 liver) mapped from ASE SNPs via Ensembl VEP

## Outputs

- STRING PPI network with node count, edge count, average node degree, and PPI enrichment p-value
- KEGG pathway enrichment table (pathway, gene count, FDR-adjusted p-value) written to string_kegg_pathway_enrichment.tsv

## How to apply

Convert gene identifiers to symbols if needed (in the source study, 344 shared ASE genes with global ASE score >= 50% yielded 208 with assigned gene symbols). Upload the symbol list to STRING and confirm the network shows significant PPI enrichment versus random expectation (study value: p < 1.0e-16; 176 nodes, 374 edges, average node degree 4.25). Extract the top enriched KEGG pathways with their gene counts and FDR-adjusted p-values — here ribosome (count = 18, FDR p = 8.62e-15), metabolic pathways (count = 32, FDR p = 3.33e-07), and oxidative phosphorylation (count = 11, FDR p = 8.93e-07). Record the enrichment table to a tabular output (string_kegg_pathway_enrichment.tsv) for validation against reported values.

## Related tools

- **STRING** (Web server used to build the PPI network from 208 gene symbols and extract top KEGG pathways with FDR-adjusted p-values)
- **Ensembl Variant Effect Predictor (VEP)** (Maps ASE SNPs from VADT meta-analysis call sets to Ensembl gene IDs to construct the gene list uploaded to STRING)
- **VADT (VCF ASE Detection Tool)** (Upstream tool producing the sig_meta_analysis_variants.txt ASE SNP call sets per tissue that define the gene input) — https://github.com/mjtiv/VADT

## Examples

```
python VADT.py --File_Name sig_meta_analysis_variants.txt --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20 --Meta_BH_adj_p_value_cutoff 0.05  # then upload the 208 gene symbols to STRING and export KEGG enrichment to string_kegg_pathway_enrichment.tsv
```

## Evaluation signals

- Three-tissue ASE gene overlap must equal the reported 20.1% (1715 shared genes)
- Selected gene set must be n = 208 genes with assigned symbols (from 344 genes with global ASE score >= 50%)
- PPI enrichment must be significant at p < 1.0e-16 with 176 nodes, 374 edges, average node degree 4.25
- Top three KEGG pathways must match: ribosomes (count 18, FDR p = 8.62e-15), metabolic pathways (count 32, FDR p = 3.33e-07), oxidative phosphorylation (count 11, FDR p = 8.93e-07)
- Enrichment table must be written and consistent with the Table-level Results text

## Limitations

- Without eQTL data, association of ASE SNPs with translation improvements is speculative
- A large number of SNPs were excluded due to low overlap with the 600K chicken genotyping panel, attributed to limitations of commercial genotyping panels
- Mapping issues with muscle tissue samples were encountered, likely due to the current genome build and annotation of muscle-related genes

## Evidence

- [results] ASE genes show tissue-specific pathway enrichment patterns, but all three tissues converged on enrichment for pathways involved in translation (ribosome-related KEGG categories).: [evidence span withheld — anchor status: not_found]
- [abstract] When ASE genes (genes containing ASE SNPs) were compared between tissues, the overlap among all three tissues increased to 20.1%.: "When ASE genes (genes containing ASE SNPs) were compared between tissues, the overlap among all three tissues increased to 20.1%."
- [discussion] however, without eQTL data, this is just speculative: "however, without eQTL data, this is just speculative"
- [abstract] ASE analysis was performed using a custom software called VCF ASE Detection Tool (VADT), which detects ASE of biallelic SNPs using a binomial test.: "ASE analysis was performed using a custom software called VCF ASE Detection Tool (VADT), which detects ASE of biallelic SNPs using a binomial test."
