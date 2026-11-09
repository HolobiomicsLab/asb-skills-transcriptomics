---
name: topic-association-matrix-extraction
description: Use when a fitted CODAL/MIRA topic model is available and you need per-topic
  gene rankings from the beta matrix to test whether a topic captures a specific cell-type
  identity program or to interpret topics biologically.
when_to_use_negative:
- The model has not been fitted yet — this skill operates on an already-trained CODAL/MIRA
  topic model's beta matrix, not raw counts
- The question is batch correction or cell-state integration itself rather than interpreting
  topics as gene programs (use CODAL integration skill instead)
- The gene set of interest lacks ontology annotation or cell-type-specific log-fold
  changes, so the enrichment/correlation validation is impossible
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CODAL
  role: Variational autoencoder-based topic model whose fitted beta matrix supplies
    the Ntopics x Ngenes topic-gene associations being extracted and interpreted
  repo: https://mira-multiome.readthedocs.io
- name: MIRA
  role: Python package providing multimodal topic modeling; its expression topic model
    object exposes the beta matrix used for per-topic gene rankings
  repo: https://github.com/cistrome/MIRA
- name: SciPy
  role: scipy.stats.fisher_exact (alternative='greater') runs the one-sided 2x2 ontology
    enrichment test on the top-200 gene set
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:7f5d0afeec34c1157ef528f98d895d944373bb5259b106b13d25befb00cf7af4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CODAL
  - MIRA
  - SciPy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# topic-association-matrix-extraction

## Summary

Extracts the Ntopics x Ngenes topic association matrix (beta) from a fitted CODAL/MIRA expression topic model so individual topics can be linked to gene-level signatures (e.g., ranking Topic 14's top 200 genes for ontology enrichment and log-fold-change correlation).

## When to use

Use when a fitted CODAL/MIRA topic model is available and you need per-topic gene rankings from the beta matrix to test whether a topic captures a specific cell-type identity program or to interpret topics biologically.

## When NOT to use

- The model has not been fitted yet — this skill operates on an already-trained CODAL/MIRA topic model's beta matrix, not raw counts
- The question is batch correction or cell-state integration itself rather than interpreting topics as gene programs (use CODAL integration skill instead)
- The gene set of interest lacks ontology annotation or cell-type-specific log-fold changes, so the enrichment/correlation validation is impossible

## Inputs

- Fitted CODAL/MIRA expression topic model containing the Ntopics x Ngenes beta (topic association) matrix
- Single-cell RNA-seq count matrix (AnnData/h5ad style, e.g. NEURIPS bone marrow multimodal data) preprocessed per the paper's filter steps
- Gene set membership annotation (e.g., HuBMAP CD16+ monocyte ontology gene list)
- Cell-type log-fold-change values for correlation analysis (e.g., Fig. 3f, g comparison)

## Outputs

- Top-200 gene ranking for the target topic (e.g., Topic 14) by beta weight
- One-sided Fisher's exact enrichment p-value against a cell-type ontology gene set (expected ~1.5e-19 for Topic 14/CD16+ monocytes)
- Topic-association vs. log-fold-change correlation statistic (Spearman/Pearson)
- Results table (e.g., topic14_enrichment_results.csv)

## How to apply

Load the fitted CODAL/MIRA expression topic model — CODAL extends MIRA's variational topic modeling and all MIRA topic models use CODAL inference — and pull the Ntopics x Ngenes association (beta) matrix from the model object. Rank all genes by their weight in the topic of interest and retain the top 200 genes, mirroring the paper's top-200-genes-per-topic convention. To validate biological interpretation (Topic 14 -> CD16+ monocytes), cross the top-200 set against HuBMAP CD16+ monocyte ontology membership in a 2x2 contingency table and run a one-sided Fisher's exact test (scipy.stats.fisher_exact, alternative='greater'); optionally correlate topic association scores with gene log-fold changes in the target cell type via Spearman/Pearson correlation. Inputs to the model should follow the paper's preprocessing: cells with fewer than 400 counts and genes with fewer than 30 counts filtered, normalize_total with target_sum=10000, log+1 transform, and highly variable gene selection (minimum dispersion 0.3, yielding 3500 genes), with raw counts deviance-residual transformed before encoding.

## Related tools

- **CODAL** (Variational autoencoder-based topic model whose fitted beta matrix supplies the Ntopics x Ngenes topic-gene associations being extracted and interpreted) — https://mira-multiome.readthedocs.io
- **MIRA** (Python package providing multimodal topic modeling; its expression topic model object exposes the beta matrix used for per-topic gene rankings) — https://github.com/cistrome/MIRA
- **SciPy** (scipy.stats.fisher_exact (alternative='greater') runs the one-sided 2x2 ontology enrichment test on the top-200 gene set)

## Examples

```
from scipy.stats import fisher_exact; odds, p = fisher_exact([[n_top, 200-n_top],[N_cd16-n_top, N_rest]], alternative='greater')
```

## Evaluation signals

- The one-sided Fisher's exact p-value reproduces the reported Topic 14 enrichment value of 1.5e-19 against HuBMAP CD16+ monocyte ontology genes
- The top-200 gene ranking for Topic 14 matches the paper's Fig. 3f, g gene set convention
- The topic-association vs. log-fold-change correlation reproduces the trend displayed in Fig. 3f, g
- Beta matrix dimensions equal Ntopics x Ngenes over the modeled gene set (e.g., the 3500 highly variable genes with dispersion >= 0.3)

## Limitations

- Interpretable topics must be validated with orthogonal biological evidence to rule out under-correction of technical effects by CODAL
- CODAL does not remove the need for good experimental design and appropriate controls, so topic-cell-type associations can reflect residual technical confounding
- Version numbers and some computational implementation details are not reported in the article, complicating exact reproduction

## Evidence

- [intro] The CODAL model architecture extends MIRA28, our earlier method for variational topic modeling of single-cell RNA-seq and ATAC-seq data.: "The CODAL model architecture extends MIRA28, our earlier method for variational topic modeling of single-cell RNA-seq and ATAC-seq data."
- [methods] CODAL employs a cyclical stochastic minibatch gradient ascent optimization strategy: "CODAL employs a cyclical stochastic minibatch gradient ascent optimization strategy to learn parameters"
- [methods] cells with fewer than 400 counts and genes with fewer than 30 counts filtered out: "cells with fewer than 400 counts and genes with fewer than 30 counts filtered out"
- [readme] All MIRA topic models use the new CODAL algorithm for inference.: [evidence span withheld — anchor status: external_doc]
- [readme] MIRA takes count matrices from scRNA-seq, scATAC-seq, or scRNA-seq+scATAC-seq from a single-cell multiomics experiment: [evidence span withheld — anchor status: external_doc]
