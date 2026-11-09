---
name: kegg-pathway-enrichment-interpretation
description: Use when a filtered gene set with gene symbols (e.g., 208 high-confidence
  ASE genes with global ASE score >= 50%) needs functional interpretation, and PPI/KEGG
  enrichment statistics must be reported from STRING.
when_to_use_negative:
- The gene list lacks gene symbols (STRING requires symbol-level identifiers; 136
  of the 344 high-ASE-score genes had no symbol and were excluded)
- Only SNP-level ASE overlap is of interest — the article reports only 3.7% of ASE
  SNPs are shared across all three tissues, whereas gene-level overlap (20.1%) is
  what this skill targets
- A quantitative pathway over-representation test with explicit background control
  is required rather than STRING's enrichment annotation
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: STRING
  role: Receives the 208 gene symbols, builds the PPI network, tests PPI enrichment,
    and returns top KEGG pathways with counts and FDR-adjusted p-values
- name: Ensembl Variant Effect Predictor (VEP)
  role: Maps ASE SNPs to Ensembl gene IDs so SNP-level ASE calls can be aggregated
    to the gene level for enrichment
- name: VADT (VCF ASE Detection Tool)
  role: Produces the per-tissue meta-analysis ASE SNP call sets that supply ASE and
    informative SNP counts for gene scoring
  repo: https://github.com/mjtiv/VADT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:cc03aa76ea1688a2bc4936edcf48135d2b29e5ec6ac8670facce37baca07adf0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - Ensembl Variant Effect Predictor (VEP)
  - STRING
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# kegg-pathway-enrichment-interpretation

## Summary

Interprets a curated gene list (e.g., 208 broiler ASE genes with assigned symbols) by uploading the symbols to STRING to test protein-protein interaction enrichment and extract top KEGG pathways, quantifying which functional categories (ribosome, metabolic pathways, oxidative phosphorylation) explain shared cis-regulatory biology across tissues.

## When to use

Use when a filtered gene set with gene symbols (e.g., 208 high-confidence ASE genes with global ASE score >= 50%) needs functional interpretation, and PPI/KEGG enrichment statistics must be reported from STRING.

## When NOT to use

- The gene list lacks gene symbols (STRING requires symbol-level identifiers; 136 of the 344 high-ASE-score genes had no symbol and were excluded)
- Only SNP-level ASE overlap is of interest — the article reports only 3.7% of ASE SNPs are shared across all three tissues, whereas gene-level overlap (20.1%) is what this skill targets
- A quantitative pathway over-representation test with explicit background control is required rather than STRING's enrichment annotation

## Inputs

- List of genes with assigned gene symbols (e.g., 208 of 344 shared ASE genes with global ASE score >= 50%)
- Per-tissue ASE SNP counts and informative SNP counts from VADT meta-analysis call sets (sig_meta_analysis_variants.txt for breast muscle, abdominal fat, liver)
- Ensembl gene IDs mapped from ASE SNPs via Ensembl VEP

## Outputs

- STRING PPI network statistics (node count, edge count, average node degree, PPI enrichment p-value)
- KEGG pathway enrichment table (pathway name, gene count, FDR-adjusted p-value) written to string_kegg_pathway_enrichment.tsv
- Functional interpretation of tissue-specific vs. tissue-converged pathway enrichment

## How to apply

Map ASE SNPs to Ensembl gene IDs (via Ensembl VEP), compute per-gene ASE scores as (ASE SNPs / informative SNPs) x 100 per tissue, average across tissues, and select genes with a global ASE score >= 50%. Keep only genes with assigned gene symbols (the study's shared set yielded 344 such genes, of which 208 had symbols), since STRING requires symbol-level input. Upload the gene list to STRING to build the protein-protein interaction network and confirm PPI enrichment relative to the background genome; in this study it yielded 176 nodes, 374 edges, average node degree 4.25, p < 1.0e-16. Extract the top KEGG pathways from STRING's enrichment output, recording pathway name, gene count, and FDR-adjusted p-value, and write the table (string_kegg_pathway_enrichment.tsv). Interpret enrichment patterns per tissue while checking whether all tissues converge on a shared category — here all three tissues converged on translation-related (ribosome) pathways.

## Related tools

- **STRING** (Receives the 208 gene symbols, builds the PPI network, tests PPI enrichment, and returns top KEGG pathways with counts and FDR-adjusted p-values)
- **Ensembl Variant Effect Predictor (VEP)** (Maps ASE SNPs to Ensembl gene IDs so SNP-level ASE calls can be aggregated to the gene level for enrichment)
- **VADT (VCF ASE Detection Tool)** (Produces the per-tissue meta-analysis ASE SNP call sets that supply ASE and informative SNP counts for gene scoring) — https://github.com/mjtiv/VADT

## Evaluation signals

- Three-tissue ASE gene overlap must match the reported 20.1% (1715 shared genes) before enrichment is attempted
- Gene selection must yield 344 genes with global ASE score >= 50% and n = 208 with assigned gene symbols
- STRING PPI enrichment must be significant (p < 1.0e-16) with 176 nodes, 374 edges, average node degree 4.25
- Top KEGG pathways must match reported values: ribosomes (count = 18, FDR 8.62e-15), metabolic pathways (count = 32, FDR 3.33e-07), oxidative phosphorylation (count = 11, FDR 8.93e-07)
- The KEGG enrichment table must be written as string_kegg_pathway_enrichment.tsv

## Limitations

- No eQTL data were available, so the association of ASE SNPs with translation improvements is speculative
- Pathway enrichment depends on prior VADT filtering; a large number of SNPs were excluded due to low overlap with the 600K chicken genotyping panel
- Tissue-specific pathway patterns coexist with convergence — enrichment must be examined per tissue, not only across the union

## Evidence

- [abstract] Pathway enrichment converges on translation across tissues: "Our results indicate that ASE genes show tissue-specific enrichment patterns, but all three tissues showed enrichment for pathways involved in translation."
- [abstract] Gene-level overlap is the enrichment input: "When ASE genes (genes containing ASE SNPs) were compared between tissues, the overlap among all three tissues increased to 20.1%."
- [methods] FDR control underlies significance calls: "the false discovery rate (FDR) is controlled using the Benjamini–Hochberg p-value adjustment method47 generating a final adjusted p-value"
