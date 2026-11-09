---
name: probabilistic-in-silico-deletion
description: Use when you need to identify which transcription factor motifs drive
  a gene expression program in single-cell multimodal (scRNA+scATAC) data, by probabilistically
  masking motif-bearing accessible peaks and measuring generative likelihood drops
  in a MIRA regulatory potential model.
when_to_use_negative:
- Data is single-modality or not linked multimodal scRNA-seq + scATAC-seq with shared
  cell barcodes — MIRA's regulatory potential models require both accessibility and
  expression measurements
- The goal is generic differential accessibility peak calling rather than attributing
  gene programs to transcription factor motifs
- No defined target gene set exists to test enrichment against a background — the
  Wilcoxon enrichment step requires a gene set contrast
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MIRA
  role: Provides the regulatory potential topic models on which probabilistic in silico
    deletion is performed and the framework for regulator/functional enrichment of
    motif scores
  repo: https://github.com/cistrome/MIRA
- name: CODAL
  role: Augments the MIRA generative model with disentangled technical effects (fixed
    as an additive, non-trainable negative-binomial component) so in silico deletion
    reflects biology rather than batch artifacts
  repo: https://github.com/cistrome/MIRA
- name: MACS
  role: Peak caller for deriving accessible chromatin regions used as the maskable
    elements in regulatory potential modeling
  repo: https://github.com/macs3-project/MACS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:7c7f564aee189fef6556f78855586d2d216ff4f90264fab9687f1620fb72d238
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL
  - MACS
  - MIRA
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# probabilistic-in-silico-deletion

## Summary

A MIRA-based attribution procedure that scores transcription factor motif–gene associations by masking accessible peaks containing each motif and measuring the resulting drop in likelihood of the generative regulatory-potential model, then testing motif score enrichment across gene sets. It was used to identify JASPAR motif enrichments (e.g., TAL1-GATA1) in the NEURIPS bone marrow proerythroblast program with and without CODAL batch correction.

## When to use

Use when you need to identify which transcription factor motifs drive a gene expression program in single-cell multimodal (scRNA+scATAC) data, by probabilistically masking motif-bearing accessible peaks and measuring generative likelihood drops in a MIRA regulatory potential model.

## When NOT to use

- Data is single-modality or not linked multimodal scRNA-seq + scATAC-seq with shared cell barcodes — MIRA's regulatory potential models require both accessibility and expression measurements
- The goal is generic differential accessibility peak calling rather than attributing gene programs to transcription factor motifs
- No defined target gene set exists to test enrichment against a background — the Wilcoxon enrichment step requires a gene set contrast

## Inputs

- Multimodal single-cell scRNA-seq + scATAC-seq count data (AnnData; e.g., the NEURIPS bone marrow multimodal batch-effect benchmark dataset)
- Gene set to test: top 200 genes ranked by topic-4 contribution (proerythroblast-upregulated set)
- Background gene set: remaining 2641 highly variable genes
- JASPAR 2020 vertebrate position weight matrix (PWM) collection (1641 PWMs)
- UCSC TSS annotations for the 2841 highly variable genes

## Outputs

- Motif–gene association scores (likelihood drop upon masking motif-bearing accessible peaks) for each of the 1641 JASPAR PWMs
- Bonferroni-corrected Wilcoxon enrichment p-values per motif, per model variant
- Comparison table, e.g., fig4d_motif_enrichment_table.csv

## How to apply

Train a MIRA regulatory potential topic model on the multimodal data (e.g., 2841 highly variable genes with UCSC TSS annotations from the NEURIPS bone marrow dataset); optionally train a second model with CODAL technical effects fixed as an additive, non-trainable component of the negative binomial generative distribution so batch artifacts do not distort motif–gene associations. For each PWM in the JASPAR 2020 vertebrate collection (1641 PWMs), run probabilistic in silico deletion: mask the accessible peaks containing that motif and score the motif–gene association as the drop in likelihood of the generative parameters. Assess enrichment of each motif's association scores among the target gene set (e.g., the top 200 genes ranked by topic-4 contribution, the proerythroblast-upregulated set) versus all other genes (the 2641 remaining highly variable genes) using a Wilcoxon test. Bonferroni-correct p-values across all 1641 PWMs and compare corrected enrichments between model variants (e.g., unaugmented vs. CODAL-augmented).

## Related tools

- **MIRA** (Provides the regulatory potential topic models on which probabilistic in silico deletion is performed and the framework for regulator/functional enrichment of motif scores) — https://github.com/cistrome/MIRA
- **CODAL** (Augments the MIRA generative model with disentangled technical effects (fixed as an additive, non-trainable negative-binomial component) so in silico deletion reflects biology rather than batch artifacts) — https://github.com/cistrome/MIRA
- **MACS** (Peak caller for deriving accessible chromatin regions used as the maskable elements in regulatory potential modeling) — https://github.com/macs3-project/MACS

## Examples

```
conda create --name mira-env -c conda-forge -c pytorch -c bioconda scanpy jupyter leidenalg; pip3 install mira-multiome
```

## Evaluation signals

- CODAL-corrected models should yield a 3-orders-of-magnitude increase in significance of TAL1-GATA1 motif enrichment relative to unaugmented models
- CODAL-augmented models should additionally recover significant enrichment for GATA family and TAL1-TCF3 complex motifs beyond the TAL1-GATA1 signal seen uncorrected
- Exactly 1641 JASPAR 2020 vertebrate PWMs scored, with p-values Bonferroni-corrected across that count
- Enrichment contrast resolves to the 200 topic-4 (proerythroblast-upregulated) genes versus the 2641 remaining highly variable genes
- Output written as a full comparison to fig4d_motif_enrichment_table.csv with corrected p-values for both unaugmented and CODAL-augmented models

## Limitations

- New batch-confounded or perturbation-induced signals should be validated with orthogonal biological evidence to rule out under-correction of technical effects
- CODAL correction does not remove the need for good experimental design and appropriate controls (e.g., include perturbation-unaffected control cells in both treatment and control batches)
- Further improvements may require better characterization of technical sources of bias in conjunction with the CODAL model

## Evidence

- [methods] JASPAR PWM collection scope: "using the JASPAR79 2020 vertebrate position weight matrix (PWM) collection"
- [methods] Gene universe with TSS annotations: "2841 genes which were both highly variable and had UCSC TSS annotations78"
- [intro] MIRA as the topic modeling substrate for CODAL: "The CODAL model architecture extends MIRA28, our earlier method for variational topic modeling of single-cell RNA-seq and ATAC-seq data."
- [readme] MIRA enrichment capability from README: "Regulator and functional enrichment"
- [readme] CODAL inference used in all MIRA topic models: [evidence span withheld — anchor status: external_doc]
- [discussion] Caution on validation: "the existence of new batch-confounded or perturbation-induced cell states should be validated using orthogonal biological evidence to rule out the under-correction of technical effects"
