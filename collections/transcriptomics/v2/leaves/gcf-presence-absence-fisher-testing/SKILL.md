---
name: gcf-presence-absence-fisher-testing
description: Use when you have a GCF absence/presence matrix (e.g., from BiG-SCAPE
  at 0.70 distance) plus binary lifestyle labels and need to rank BGC families significantly
  enriched in one lifestyle group.
when_to_use_negative:
- Input is a gene-level (not GCF-level) presence/absence table — use bacLIFE's pLAG
  workflow (70% group presence, >2 log2 fold change) instead
- You need lifestyle prediction for unlabeled genomes — use bacLIFE's random forest
  lifestyle prediction module (confidence >0.8) rather than enrichment testing
- GCFs are sparse within the lifestyle group (<15% group presence), since associations
  are restricted to clusters reasonably represented in the group
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0121
tools:
- name: bacLIFE
  role: Generates the GCF absence/presence matrix (via BiG-SCAPE clustering at 0.70
    distance) and genome annotations that feed the Fisher exact test workflow
  repo: https://github.com/Carrion-lab/bacLIFE
- name: R (base fisher.test and p.adjust)
  role: Performs per-GCF 2x2 Fisher's exact tests and Benjamini-Hochberg multiple-testing
    correction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:dce00fe8c207e98cd2ff9a95bec659d0a96184d0d743190290f315d50ec05bc2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - R (base fisher.test and p.adjust)
  - bacLIFE
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gcf-presence-absence-fisher-testing

## Summary

Tests each biosynthetic gene cluster family (GCF) from a BiG-SCAPE absence/presence matrix (0.70 distance) for statistical association with a plant-pathogenic lifestyle using per-GCF 2x2 Fisher's exact tests, with Benjamini-Hochberg correction at P < 0.01, as done in bacLIFE for Burkholderia/Paraburkholderia and Pseudomonas.

## When to use

Use when you have a GCF absence/presence matrix (e.g., from BiG-SCAPE at 0.70 distance) plus binary lifestyle labels and need to rank BGC families significantly enriched in one lifestyle group.

## When NOT to use

- Input is a gene-level (not GCF-level) presence/absence table — use bacLIFE's pLAG workflow (70% group presence, >2 log2 fold change) instead
- You need lifestyle prediction for unlabeled genomes — use bacLIFE's random forest lifestyle prediction module (confidence >0.8) rather than enrichment testing
- GCFs are sparse within the lifestyle group (<15% group presence), since associations are restricted to clusters reasonably represented in the group

## Inputs

- GCF absence/presence matrix from BiG-SCAPE clustering at 0.70 distance
- Lifestyle metadata annotations (e.g., plant pathogen vs. non-plant-pathogen) for each genome, collected from the literature
- Per-GCF biosynthetic class annotations (e.g., NRPS, NRPS-PKS hybrid) for labeling significant clusters

## Outputs

- Ranked table of significant plant-pathogen-associated GCFs with p-values, Benjamini-Hochberg adjusted p-values, and group presence (plant_pathogen_associated_gcfs.csv)

## How to apply

Load the BiG-SCAPE GCF absence/presence matrix generated at 0.70 distance together with literature-based lifestyle metadata for each genome. For each individual GCF, build a 2x2 contingency table of presence/absence versus plant-pathogen versus non-plant-pathogen genomes and compute an enrichment p-value with base R's fisher.test(). Restrict associations to GCFs meeting the >15% group presence threshold so only reasonably represented clusters are tested, then correct per-GCF p-values with p.adjust() using the Benjamini-Hochberg method and flag GCFs significant at P < 0.01. Annotate significant GCFs with their biosynthetic class (e.g., NRPS, NRPS-PKS hybrid) so families such as the syringofactin and syringomycin NRPS clusters and the novel NRPS-PKS hybrid cluster can be identified in the ranked output table.

## Related tools

- **bacLIFE** (Generates the GCF absence/presence matrix (via BiG-SCAPE clustering at 0.70 distance) and genome annotations that feed the Fisher exact test workflow) — https://github.com/Carrion-lab/bacLIFE
- **R (base fisher.test and p.adjust)** (Performs per-GCF 2x2 Fisher's exact tests and Benjamini-Hochberg multiple-testing correction)

## Examples

```
Rscript -e 'm <- read.csv("gcf_matrix.csv"); tab <- table(m$gcf_presence, m$plant_pathogen); fisher.test(tab); p.adjust(pvals, method="BH")'
```

## Evaluation signals

- Output must reproduce the associations reported in Supplementary Datasets 7-8
- The syringofactin and syringomycin NRPS families must be recovered among significant plant-pathogen-associated GCFs in Pseudomonas
- The novel plant pathogen-associated NRPS-PKS hybrid cluster entry must contain the substring 'NRPS'
- All reported GCFs pass the >15% group presence threshold and have Benjamini-Hochberg adjusted P < 0.01
- Output CSV contains p-values, adjusted p-values, and group presence columns for each significant GCF

## Limitations

- Associations are restricted to GCFs present in >15% of the lifestyle group, so rare clusters cannot be tested
- Fisher's exact test on a 2x2 presence/absence table captures enrichment only, not directionality of biosynthetic function or cluster novelty beyond class labels
- Results depend on upstream BiG-SCAPE clustering distance (0.70); a different cutoff changes GCF family definitions and thus the tested features

## Evidence

- [methods] BiG-SCAPE GCF matrix input: "an absence/presence matrix of the individual GCFs at 0.70 BiG-SCAPE distance in the input genomes is generated"
- [results] Fisher test on lifestyle-associated BGCs: [evidence span withheld — anchor status: not_found]
- [readme] bacLIFE purpose: "a streamlined computational workflow that annotates bacterial genomes and performs large-scale comparative genomics to predict bacterial lifestyles and to pinpoint candidate genes"
