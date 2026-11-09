---
name: regulatory-potential-model-technical-effect-augmentation
description: Use when a multi-batch single-cell multimodal (RNA+ATAC) dataset is being
  used to infer regulatory potential and TF motif enrichments, and technical/batch
  effects may confound motif-gene associations you want to compare between corrected
  and uncorrected models.
when_to_use_negative:
- Data are unimodal or lack scATAC-seq accessibility peaks, so no regulatory potential
  model over accessible peaks near TSS-annotated genes can be trained.
- Data come from a single batch with no technical-effect structure, so fixing CODAL
  technical effects as a non-trainable component is unnecessary.
- The question is generic batch correction of embeddings rather than testing whether
  technical-effect disentanglement changes downstream regulatory/motif inference.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MIRA
  role: Variational topic modeling framework used to train the regulatory potential
    models and run probabilistic in silico deletion over JASPAR PWMs.
  repo: https://github.com/cistrome/MIRA
- name: CODAL
  role: Technical-effect disentanglement model whose inferred batch effects are fixed
    as an additive, non-trainable component of the negative binomial generative distribution
    during MIRA training.
  repo: https://mira-multiome.readthedocs.io
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:7f75109cdd9b3c175ae2bec22c41c681d446cb66dd625b96ecb39e1a89fa3ac8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL
  - MIRA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# regulatory-potential-model-technical-effect-augmentation

## Summary

Trains MIRA regulatory potential models on multimodal single-cell data with CODAL technical effects fixed as an additive, non-trainable component of the negative binomial generative distribution, then uses probabilistic in silico deletion over JASPAR PWMs to test whether batch-effect disentanglement changes detected TF motif enrichments.

## When to use

Use when a multi-batch single-cell multimodal (RNA+ATAC) dataset is being used to infer regulatory potential and TF motif enrichments, and technical/batch effects may confound motif-gene associations you want to compare between corrected and uncorrected models.

## When NOT to use

- Data are unimodal or lack scATAC-seq accessibility peaks, so no regulatory potential model over accessible peaks near TSS-annotated genes can be trained.
- Data come from a single batch with no technical-effect structure, so fixing CODAL technical effects as a non-trainable component is unnecessary.
- The question is generic batch correction of embeddings rather than testing whether technical-effect disentanglement changes downstream regulatory/motif inference.

## Inputs

- Multimodal scRNA-seq + scATAC-seq count matrices spanning multiple batches (e.g., NEURIPS bone marrow dataset, 2841 highly variable genes with UCSC TSS annotations)
- Topic-model gene rankings (e.g., top 200 topic-4 contribution genes) plus the 2641-gene background set
- JASPAR 2020 vertebrate PWM collection (1641 PWMs)
- UCSC TSS annotations for the highly variable genes

## Outputs

- Per-motif in silico deletion association scores for each gene, for both unaugmented and CODAL-augmented models
- Wilcoxon enrichment p-values per motif, Bonferroni-corrected across 1641 PWMs
- Comparison table of corrected enrichment p-values between models (fig4d_motif_enrichment_table.csv)

## How to apply

First extract the topic-defined gene set of interest (e.g., top 200 genes by topic-4 contribution, the proerythroblast-upregulated set, from the CODAL expression topic model of the NEURIPS bone marrow dataset), using the remaining 2641 highly variable genes as the background. Train MIRA regulatory potential models on the 2841 highly variable genes with UCSC TSS annotations across all batches twice: once without augmentation and once with CODAL technical effects fixed as an additive, non-trainable component of the negative binomial generative distribution (with technical effect predictions Bernoulli-corrupted at p=0.05 during training). For each model, run probabilistic in silico deletion: for each of the 1641 JASPAR 2020 vertebrate PWMs, score the motif-gene association as the drop in likelihood of the generative parameters when accessible peaks containing the motif are masked. Assess enrichment of each motif's association scores among the 200-gene set versus all other genes with a Wilcoxon test, Bonferroni-correct across the 1641 PWMs, and compare corrected p-values between the unaugmented and CODAL-augmented models.

## Related tools

- **MIRA** (Variational topic modeling framework used to train the regulatory potential models and run probabilistic in silico deletion over JASPAR PWMs.) — https://github.com/cistrome/MIRA
- **CODAL** (Technical-effect disentanglement model whose inferred batch effects are fixed as an additive, non-trainable component of the negative binomial generative distribution during MIRA training.) — https://mira-multiome.readthedocs.io

## Examples

```
pip3 install mira-multiome
```

## Evaluation signals

- CODAL-augmented model yields a ~3-orders-of-magnitude increase in significance of TAL1-GATA1 motif enrichment versus the uncorrected model (Bonferroni-corrected across 1641 PWMs).
- Uncorrected model shows only TAL1-GATA1 enrichment; CODAL-corrected model additionally recovers relevant GATA-family and TAL1-TCF3 complex motifs among the 200 topic-4 proerythroblast genes.
- Enrichment assessed by Wilcoxon test of motif association scores (200-gene set vs. 2641-gene background), with Bonferroni correction applied across all 1641 JASPAR 2020 vertebrate PWMs.
- Output table (fig4d_motif_enrichment_table.csv) contains paired corrected p-values for every PWM from both the unaugmented and CODAL-augmented models.
- MIRA topic models train successfully with technical effects as a fixed additive term (no gradient updates to technical-effect components).

## Limitations

- Authors caution that newly detected batch-confounded or perturbation-induced states should be validated with orthogonal biological evidence to rule out under-correction of technical effects.
- CODAL correction does not remove the need for good experimental design, including control cells unaffected by the perturbation in both treatment and control batches.
- Further improvements may require better characterization of technical sources of bias in conjunction with the CODAL model; QC metrics (e.g., FRiP score) can be added as technical covariates.

## Evidence

- [methods] Bernoulli corruption of technical effects: "the technical effect predictions are Bernoulli corrupted with probability 0.05"
- [methods] TSS-annotated highly variable genes input: "2841 genes which were both highly variable and had UCSC TSS annotations78"
- [methods] JASPAR PWM collection: "using the JASPAR79 2020 vertebrate position weight matrix (PWM) collection"
- [intro] MIRA-CODAL architecture relationship: "The CODAL model architecture extends MIRA28, our earlier method for variational topic modeling of single-cell RNA-seq and ATAC-seq data."
- [readme] MIRA ships CODAL inference: [evidence span withheld — anchor status: external_doc]
- [discussion] CODAL as preprocessing for other methods: "CODAL’s correction of technical artifacts through multi-batch integration can therefore be used as a preprocessing step to improve the performance of other single-cell analysis methods"
