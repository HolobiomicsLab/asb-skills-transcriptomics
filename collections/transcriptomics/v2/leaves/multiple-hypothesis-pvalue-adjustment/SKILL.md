---
name: multiple-hypothesis-pvalue-adjustment
description: Use when a large set of per-feature association tests (e.g., one Fisher's
  exact test per GCF in an absence/presence matrix) produces raw p-values that must
  be BH-corrected before ranking significant hits.
when_to_use_negative:
- Input is a single pre-computed p-value or a small pre-planned hypothesis set with
  no multiplicity — BH correction would needlessly inflate the threshold
- The enrichment features are individual genes (pLAGs) rather than GCFs; bacLIFE uses
  different thresholds for genes (>70% group presence, >2 log2 fold change) and applies
  machine learning instead of per-GCF Fisher tests
- GCFs below the >15% group presence threshold — sparse clusters are excluded before
  testing rather than corrected after
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0797
tools:
- name: R (base fisher.test and p.adjust)
  role: fisher.test() computes the per-GCF 2x2 enrichment p-values; p.adjust() with
    the Benjamini-Hochberg method corrects them for multiple testing at P < 0.01
- name: bacLIFE
  role: Supplies the BiG-SCAPE GCF absence/presence matrix (0.70 distance) and lifestyle
    metadata that the per-GCF tests and p-value correction are applied to
  repo: https://github.com/Carrion-lab/bacLIFE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:0c3d720b2f35dc6da610fa082574d4a3039a60c1b4167e794215106c50fca0b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - R (base fisher.test and p.adjust)
  - bacLIFE
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# multiple-hypothesis-pvalue-adjustment

## Summary

Corrects per-feature enrichment p-values (e.g., per-GCF Fisher's exact tests) for multiple testing using Benjamini-Hochberg via base R p.adjust(), so that lifestyle-associated biosynthetic gene cluster families can be flagged at a controlled significance threshold (P < 0.01 in bacLIFE).

## When to use

Use when a large set of per-feature association tests (e.g., one Fisher's exact test per GCF in an absence/presence matrix) produces raw p-values that must be BH-corrected before ranking significant hits.

## When NOT to use

- Input is a single pre-computed p-value or a small pre-planned hypothesis set with no multiplicity — BH correction would needlessly inflate the threshold
- The enrichment features are individual genes (pLAGs) rather than GCFs; bacLIFE uses different thresholds for genes (>70% group presence, >2 log2 fold change) and applies machine learning instead of per-GCF Fisher tests
- GCFs below the >15% group presence threshold — sparse clusters are excluded before testing rather than corrected after

## Inputs

- Vector of raw per-GCF Fisher's exact test p-values (one test per GCF from the BiG-SCAPE 0.70 absence/presence matrix)
- GCF absence/presence matrix (binary, genomes x GCFs) with lifestyle metadata annotations (plant-pathogen vs non-plant-pathogen) per genome
- Group-presence fractions per GCF for applying the >15% presence threshold

## Outputs

- Ranked table of significant plant-pathogen-associated GCFs with raw p-values, BH-adjusted p-values, and group presence (plant_pathogen_associated_gcfs.csv)
- Binary significance flags at adjusted P < 0.01 per GCF

## How to apply

After computing one Fisher's exact test per individual GCF from the BiG-SCAPE 0.70-distance absence/presence matrix versus lifestyle metadata (plant-pathogen vs non-plant-pathogen), collect the raw per-GCF p-values into a vector. Restrict the tested set to GCFs meeting the >15% group presence threshold so associations are limited to clusters reasonably represented in the lifestyle group. Apply base R's p.adjust() with method='BH' (Benjamini-Hochberg) to the retained p-value vector to obtain adjusted p-values, then flag GCFs significant at P < 0.01. Rank surviving GCFs by adjusted p-value and annotate each with its biosynthetic class (e.g., NRPS, NRPS-PKS hybrid) so known families like the syringofactin and syringomycin NRPS clusters can be identified alongside novel hits. Write the ranked table of p-values, adjusted p-values, and group presence to plant_pathogen_associated_gcfs.csv.

## Related tools

- **R (base fisher.test and p.adjust)** (fisher.test() computes the per-GCF 2x2 enrichment p-values; p.adjust() with the Benjamini-Hochberg method corrects them for multiple testing at P < 0.01)
- **bacLIFE** (Supplies the BiG-SCAPE GCF absence/presence matrix (0.70 distance) and lifestyle metadata that the per-GCF tests and p-value correction are applied to) — https://github.com/Carrion-lab/bacLIFE

## Examples

```
padj <- p.adjust(pvals, method="BH"); sig <- names(padj)[padj < 0.01]
```

## Evaluation signals

- Adjusted p-values from p.adjust(method='BH') must be monotonically >= the corresponding raw p-values, and ranking by adjusted p-value preserves the raw p-value ordering
- The output table must reproduce the associations reported in Supplementary Datasets 7-8
- The syringofactin and syringomycin NRPS families and the novel NRPS-PKS hybrid cluster must appear in the significant set, with the novel cluster entry containing the substring 'NRPS'
- All retained GCFs must satisfy the >15% group presence threshold before testing
- Every flagged hit must have adjusted P < 0.01

## Limitations

- Significant GCFs are annotated as 'likely encoding' known compounds (e.g., syringofactin, syringomycin) — the statistical association is not direct functional proof; the paper separately validated candidate LAGs by site-directed mutagenesis and plant bioassays
- Restricting tests to GCFs with >15% group presence means rare clusters cannot be detected as lifestyle-associated
- Fisher's exact test on 2x2 presence/absence tables ignores abundance and copy-number variation of BGCs across genomes

## Evidence

- [other] Correct the per-GCF p-values for multiple testing using the base R p.adjust() function with the Benjamini-Hochberg method and flag GCFs significant at P < 0.01.: [evidence span withheld — anchor status: not_found]
- [other] For each individual GCF, construct a 2x2 contingency table of presence/absence versus plant-pathogen versus non-plant-pathogen genomes and compute an enrichment p-value using the base R fisher.test() function.: [evidence span withheld — anchor status: not_found]
- [other] Retain only GCFs meeting the >15% group presence threshold, so that associations are restricted to clusters reasonably represented in the lifestyle group.: [evidence span withheld — anchor status: not_found]
- [methods] an absence/presence matrix of the individual GCFs at 0.70 BiG-SCAPE distance in the input genomes is generated: "an absence/presence matrix of the individual GCFs at 0.70 BiG-SCAPE distance in the input genomes is generated"
- [other] In Pseudomonas, the plant pathogen-associated GCFs found via Fisher's exact test include BGCs likely encoding the well-studied syringofactin and syringomycin, two NRPS-encoded lipopeptides that are critical for pathogenicity: [evidence span withheld — anchor status: not_found]
