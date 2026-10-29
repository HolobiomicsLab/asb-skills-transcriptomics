---
name: motif-enrichment-testing-with-pwm-collection
description: Use when you need to identify which TF binding motifs are enriched among
  an upregulated gene set (e.g., the top 200 topic-4 proerythroblast genes), especially
  when motif signal may be masked by technical batch effects.
when_to_use_negative:
- When the analysis goal is de novo motif discovery rather than testing a curated
  PWM collection — in silico deletion only scores pre-defined JASPAR motifs
- When no scATAC-seq accessibility data are available, since regulatory potential
  and peak masking depend on accessible peaks linked to UCSC TSS annotations
- When technical-effect correction is irrelevant (e.g., single-batch data), where
  the CODAL-augmented model adds no value over the unaugmented regulatory potential
  model
tools:
- name: MIRA
  role: Trains the regulatory potential models and runs probabilistic in silico deletion
    of JASPAR motif-containing accessible peaks to score motif-gene associations
  repo: https://github.com/cistrome/MIRA
- name: CODAL
  role: Provides technical effect correction for the regulatory potential model, fixed
    as an additive non-trainable component of the negative binomial generative distribution
  repo: https://mira-multiome.readthedocs.io
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:5c750b89365ca3ce38a7a80ddcc6b85d28259ae104ee83f1df240b3aa06c2427
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL
  - MIRA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Motif enrichment testing with PWM collection

## Summary

Tests transcription factor motif enrichment among a target gene set by scoring motif-gene associations over a curated PWM collection (here, the 1641 JASPAR 2020 vertebrate PWMs) via probabilistic in silico deletion on a MIRA regulatory potential model, then comparing score distributions with a Wilcoxon test and Bonferroni correction.

## When to use

Use when you need to identify which TF binding motifs are enriched among an upregulated gene set (e.g., the top 200 topic-4 proerythroblast genes), especially when motif signal may be masked by technical batch effects.

## When NOT to use

- When the analysis goal is de novo motif discovery rather than testing a curated PWM collection — in silico deletion only scores pre-defined JASPAR motifs
- When no scATAC-seq accessibility data are available, since regulatory potential and peak masking depend on accessible peaks linked to UCSC TSS annotations
- When technical-effect correction is irrelevant (e.g., single-batch data), where the CODAL-augmented model adds no value over the unaugmented regulatory potential model

## Inputs

- Top 200 genes ranked by topic-4 contribution from a CODAL expression topic model of the NEURIPS bone marrow dataset
- Multimodal NEURIPS bone marrow scRNA-seq + scATAC-seq data (2841 highly variable genes with UCSC TSS annotations, all batches)
- JASPAR 2020 vertebrate PWM collection (1641 position weight matrices)
- Accessible peak sets per gene for motif masking

## Outputs

- Motif-gene association scores (likelihood drop on in silico deletion) per PWM per gene
- Wilcoxon enrichment p-values for each of the 1641 PWMs (target 200 genes vs. 2641-gene background)
- Bonferroni-corrected enrichment p-values, e.g., written to fig4d_motif_enrichment_table.csv comparing unaugmented vs. CODAL-augmented models

## How to apply

Extract the target gene set (e.g., the 200 genes ranked by topic-4 contribution in the CODAL expression topic model of the NEURIPS bone marrow dataset), using the remaining 2641 highly variable genes as background. Train a MIRA regulatory potential model on the 2841 highly variable genes with UCSC TSS annotations across all batches, optionally with CODAL technical effects fixed as an additive, non-trainable component of the negative binomial generative distribution. For each of the 1641 JASPAR 2020 vertebrate PWMs, run probabilistic in silico deletion: mask accessible peaks containing the motif and score the motif-gene association as the drop in likelihood of the generative parameters. Assess enrichment of each motif's association scores among the target 200 genes versus all others with a Wilcoxon test, then Bonferroni-correct p-values across all 1641 PWMs before comparing models.

## Related tools

- **MIRA** (Trains the regulatory potential models and runs probabilistic in silico deletion of JASPAR motif-containing accessible peaks to score motif-gene associations) — https://github.com/cistrome/MIRA
- **CODAL** (Provides technical effect correction for the regulatory potential model, fixed as an additive non-trainable component of the negative binomial generative distribution) — https://mira-multiome.readthedocs.io

## Examples

```
pip3 install mira-multiome
```

## Evaluation signals

- Exactly 1641 JASPAR 2020 vertebrate PWMs are scored; the enrichment table must have one row per PWM with a Bonferroni-corrected p-value
- Wilcoxon test compares the 200 topic-4 target genes against the 2641-gene background set; counts must sum to the 2841 highly variable genes
- The CODAL-corrected model should reproduce the published result: TAL1-GATA1 enrichment significance increased by ~3 orders of magnitude, plus newly significant GATA family and TAL1-TCF3 complex motifs absent in the uncorrected model
- Comparison output (e.g., fig4d_motif_enrichment_table.csv) must contain paired corrected p-values for unaugmented and CODAL-augmented models so relative significance can be checked

## Limitations

- Motif enrichment depends on the quality of batch-effect handling; the authors caution that new batch-confounded or perturbation-induced signals should be validated using orthogonal biological evidence to rule out under-correction of technical effects
- Only motifs represented in the JASPAR 2020 vertebrate collection (1641 PWMs) can be detected; motifs outside that collection are invisible to the test
- Bonferroni correction across 1641 tests is conservative and may miss weaker-but-real motif enrichments

## Evidence

- [methods] using the JASPAR79 2020 vertebrate position weight matrix (PWM) collection: "using the JASPAR79 2020 vertebrate position weight matrix (PWM) collection"
- [methods] 2841 genes with UCSC TSS annotations: "2841 genes which were both highly variable and had UCSC TSS annotations78"
- [readme] Regulator and functional enrichment is a MIRA capability: "* Regulator and functional enrichment"
- [intro] MIRA underpins CODAL as the multi-modal topic model: "The CODAL model architecture extends MIRA28, our earlier method for variational topic modeling of single-cell RNA-seq and ATAC-seq data."
