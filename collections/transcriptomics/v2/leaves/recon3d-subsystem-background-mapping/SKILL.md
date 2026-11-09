---
name: recon3d-subsystem-background-mapping
description: Use when a predicted gene list (e.g. essential genes from SKO/DKO in
  silico deletion of context-specific metabolic models) needs pathway enrichment against
  KEGG with a Recon3D_01 subSystems background, plus ferroptosis annotation via FerrDb.
when_to_use_negative:
- When the question is differential expression analysis itself rather than enrichment
  of a predicted knockout gene list — DEGs are computed separately with DESeq2 (|log2FC|
  > 1, adjusted p < 0.05)
- When genes or drug targets have no pathways in Recon3D_01 subSystems — such genes
  (and their connected drugs) were discarded from the network in this workflow
- When pathway evidence rests on a single gene — single-gene enrichment results are
  discarded by this workflow and cannot support conclusions
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Enrichr
  role: Web/API service used to perform pathway enrichment of the essential gene lists
    against the KEGG pathway library
- name: KEGG
  role: Pathway library against which SKO/DKO essential gene lists are tested (pathway
    analysis of the SKO and DKO)
- name: Recon3D_01
  role: Its subSystems provide the pathway background used to constrain Enrichr enrichment
    results
- name: FerrDb
  role: External ferroptosis gene database used to classify essential genes as drivers,
    suppressors, or markers
- name: R
  role: Scripting environment for submitting gene lists to the Enrichr API and filtering/compiling
    enrichment results
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:4e32b28f32c83a5f2de11717f866b802ac3647f9f5eb3fe6c38b404a7fc3cf8c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sysbiolux/DCcov
  tools:
  - Enrichr
  - FerrDb
  - KEGG
  - R
  - Recon3D_01
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# recon3d-subsystem-background-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pathway-level interpretation of essential genes predicted by in silico SKO/DKO knockout in context-specific SARS-CoV-2-infected lung models, by submitting gene lists to the Enrichr API against the KEGG pathway library with Recon3D_01 subSystems as the pathway background, then cross-referencing hits against FerrDb ferroptosis classifications.

## When to use

Use when a predicted gene list (e.g. essential genes from SKO/DKO in silico deletion of context-specific metabolic models) needs pathway enrichment against KEGG with a Recon3D_01 subSystems background, plus ferroptosis annotation via FerrDb.

## When NOT to use

- When the question is differential expression analysis itself rather than enrichment of a predicted knockout gene list — DEGs are computed separately with DESeq2 (|log2FC| > 1, adjusted p < 0.05)
- When genes or drug targets have no pathways in Recon3D_01 subSystems — such genes (and their connected drugs) were discarded from the network in this workflow
- When pathway evidence rests on a single gene — single-gene enrichment results are discarded by this workflow and cannot support conclusions

## Inputs

- Predicted essential gene lists from SKO and DKO in silico knockout of context-specific infected lung metabolic models (CSV/text of gene symbols)
- Recon3D_01 subSystems pathway definitions used as the enrichment background
- FerrDb ferroptosis gene classification table (drivers, suppressors, markers)

## Outputs

- Filtered Enrichr/KEGG enrichment results table (terms with p-value/adjusted p-value < 0.05, no single-gene enrichments)
- FerrDb cross-reference of essential genes to ferroptosis roles (driver/suppressor/marker)
- Combined CSV results table of surviving KEGG terms and ferroptosis assignments

## How to apply

Load the essential gene lists produced by single-gene (SKO) and double-gene (DKO) in silico knockout of the rFASTCORMICS context-specific infected lung models. Submit each list to the Enrichr API (Kuleshov et al., 2016) against the KEGG pathway library, using the Recon3D_01 subSystems as the pathway background so that enrichments reflect the modeled metabolic network rather than the whole genome. Filter returned terms to p-value/adjusted p-value < 0.05 and discard all enrichment results based on only one gene, which prevents spurious single-hit terms. Separately, download FerrDb ferroptosis gene classifications (drivers, suppressors, markers) and cross-reference the essential genes to assign roles (e.g. SLC3A2 and SLC7A11 as ferroptosis suppressors, SLC7A5 as a ferroptosis marker). Compile the surviving KEGG terms and FerrDb assignments into a single CSV results table, checking that pyrimidine metabolism, oxidative stress, and ferroptosis are recovered as the key COVID-19 severity-associated pathways.

## Related tools

- **Enrichr** (Web/API service used to perform pathway enrichment of the essential gene lists against the KEGG pathway library)
- **KEGG** (Pathway library against which SKO/DKO essential gene lists are tested (pathway analysis of the SKO and DKO))
- **Recon3D_01** (Its subSystems provide the pathway background used to constrain Enrichr enrichment results)
- **FerrDb** (External ferroptosis gene database used to classify essential genes as drivers, suppressors, or markers)
- **R** (Scripting environment for submitting gene lists to the Enrichr API and filtering/compiling enrichment results)

## Examples

```
res = gp.enrichr(gene_list=sko_essential_genes, gene_sets='KEGG_2021_Human', organism='Human', outdir='enrichr_kegg')
```

## Evaluation signals

- All retained enrichment terms have p-value/adjusted p-value < 0.05
- No enrichment result based on only one gene is retained in the final table
- The output table reproduces the reported enrichment of pyrimidine metabolism, oxidative stress, and ferroptosis (the abstract's key-pathway finding)
- Ferroptosis is the only Enrichr-derived pathway associated with non-ACE2 vector samples, attributable to SLC3A2 and SLC7A11
- FerrDb cross-reference correctly assigns SLC7A11 and SLC3A2 as ferroptosis suppressors and SLC7A5 as a ferroptosis marker

## Limitations

- Enrichment depends on list size: since all single-gene enrichment results were discarded, small essential-gene lists can yield few surviving terms
- Genes without pathways in Recon3D_01 subSystems fall outside the enrichment background and are effectively invisible to this analysis
- Findings are limited to the modeled lung cell lines (A549, Calu-3, H1299, NHBE) and to the KEGG library chosen as the pathway set
- Ferroptosis attribution relies on FerrDb's current gene classifications, which may change as the database is updated

## Evidence

- [results] Enrichr pathway analysis performed on SKO/DKO results: [evidence span withheld — anchor status: not_found]
