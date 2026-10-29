---
name: embryo-development-marker-gene-identification
description: Use when a 2D latent embedding of single-cell RNA-seq data (e.g., VASC
  features of the Petropoulos pre-implantation embryo dataset) is available and the
  goal is to find and functionally annotate genes whose expression tracks embryo developmental
  dynamics.
when_to_use_negative:
- The input is raw scRNA-seq data with no reduced latent representation and the goal
  is dimension reduction/visualization itself rather than downstream marker discovery
- The dataset is not single-cell developmental transcriptomics (e.g., bulk RNA-seq
  or non-embryo cell population data) where the correlation-to-embryo-dynamics logic
  does not apply
- No web-based enrichment service (DAVID) is usable, since the enrichment step depends
  on submitting gene lists to DAVID 6.8
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: VASC
  role: Produces the 2D latent representation of the Petropoulos embryo cells via
    the VASC class in vasc.py; correlations are computed against these latent features
  repo: https://github.com/wang-research/VASC
- name: Python
  role: Analysis environment (Python 3.5+ via Anaconda) in which Spearman correlations
    between gene expression and latent features are computed
- name: numpy
  role: Numerical library (numpy 1.12.1) used for computing per-gene rank correlations
    with the latent features
- name: sklearn
  role: Machine-learning library (sklearn 0.18.1) used in the Python workflow for
    ranking genes by correlation
- name: h5py
  role: I/O library (h5py 2.7.0) for loading the single-cell expression data in the
    VASC workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:89f63e1ddd9f4bd485e3d6701777a4371980d75f00823c0ab8862a4c18f194bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wang-research/VASC
  tools:
  - Anaconda
  - Python
  - VASC
  - h5py
  - numpy
  - sklearn
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# embryo-development-marker-gene-identification

## Summary

Identifies candidate marker genes associated with human pre-implantation embryo development by correlating per-gene single-cell expression with the 2D VASC latent representation of the Petropoulos dataset, then performing Gene Ontology biological process enrichment of the top-ranked genes in DAVID 6.8.

## When to use

Use when a 2D latent embedding of single-cell RNA-seq data (e.g., VASC features of the Petropoulos pre-implantation embryo dataset) is available and the goal is to find and functionally annotate genes whose expression tracks embryo developmental dynamics.

## When NOT to use

- The input is raw scRNA-seq data with no reduced latent representation and the goal is dimension reduction/visualization itself rather than downstream marker discovery
- The dataset is not single-cell developmental transcriptomics (e.g., bulk RNA-seq or non-embryo cell population data) where the correlation-to-embryo-dynamics logic does not apply
- No web-based enrichment service (DAVID) is usable, since the enrichment step depends on submitting gene lists to DAVID 6.8

## Inputs

- VASC 2D latent feature matrix (2 columns) for the Petropoulos human pre-implantation embryo cells
- Single-cell gene expression matrix (genes x cells) matching the cells in the latent representation
- Gene identifier list for submission to DAVID 6.8

## Outputs

- Per-gene Spearman correlation and rank table for each of the two latent features
- Top-correlated gene list (rank <= 100 per feature)
- david_enrichment_results.csv containing enriched GO biological process terms with gene counts and q-values

## How to apply

Load the VASC 2D latent representation of the Petropoulos cells (produced via the VASC class in vasc.py) alongside the single-cell gene expression matrix, using the article's Python stack (Python 3.5+, numpy 1.12.1, h5py 2.7.0, sklearn 0.18.1 under Anaconda). Compute Spearman's rank correlation between every gene's expression profile and each of the two latent features. Rank genes by absolute correlation for each feature and retain those with rank <= 100 as the candidate marker set. Submit the resulting gene lists to DAVID 6.8 for Gene Ontology biological process enrichment, and record enriched terms with gene counts and q-values; the published run yielded carbohydrate derivative metabolic process (37 genes, q = 5.63E-05), oxidation-reduction process (32 genes, q = 4.87E-05), and lipid metabolic process (32 genes, q = 4.94E-03). Finally, write the enrichment table to david_enrichment_results.csv.

## Related tools

- **VASC** (Produces the 2D latent representation of the Petropoulos embryo cells via the VASC class in vasc.py; correlations are computed against these latent features) — https://github.com/wang-research/VASC
- **Python** (Analysis environment (Python 3.5+ via Anaconda) in which Spearman correlations between gene expression and latent features are computed)
- **numpy** (Numerical library (numpy 1.12.1) used for computing per-gene rank correlations with the latent features)
- **sklearn** (Machine-learning library (sklearn 0.18.1) used in the Python workflow for ranking genes by correlation)
- **h5py** (I/O library (h5py 2.7.0) for loading the single-cell expression data in the VASC workflow)

## Examples

```
python demo.py  # run VASC on the embryo scRNA-seq data (config.py sets parameters), then compute per-gene Spearman correlations against the two latent features and submit rank<=100 genes to DAVID 6.8
```

## Evaluation signals

- The retained candidate sets contain only genes with absolute-correlation rank <= 100 in at least one of the two VASC latent features
- The DAVID output reproduces the expected enriched terms with matching gene counts and q-values: carbohydrate derivative metabolic process (37 genes, q = 5.63E-05), oxidation-reduction process (32 genes, q = 4.87E-05), lipid metabolic process (32 genes, q = 4.94E-03)
- david_enrichment_results.csv exists and contains term names, gene counts, and q-values for the enriched GO biological process terms
- The latent features used for correlation are the 2D output of the VASC class from vasc.py, not raw PCA/t-SNE coordinates
- Repeated VASC runs should be checked for stability, since stochastic optimization introduces run-to-run variation in the latent representation

## Limitations

- Stochastic optimization in VASC model learning introduces run-to-run variation in the dimension reduction; repeated runs are recommended for a consensus representation
- VASC model learning can take several hours on a desktop-level computer with a single GPU card for datasets with several thousand cells
- The determination of the optimal latent dimension is tricky without prior knowledge, which affects which features are available for correlation

## Evidence

- [abstract] VASC successfully re-establishes the cell dynamics in pre-implantation embryos and identifies several candidate marker genes associated with early embryo development: "VASC successfully re-establishes the cell dynamics in pre-implantation embryos and identifies several candidate marker genes associated with early embryo development"
- [methods] vasc.py: contains a class VASC and a function vase: "vasc.py: contains a class VASC and a function vase"
- [discussion] Repeated runs recommended due to stochastic variation: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
- [readme] README: VASC re-establishes pre-implantation lineage and associated genes from 2D representation of human embryo scRNA-seq: "VASC successfully re-establishes the embryo pre-implantation cell lineage and its associated genes based on the 2D representation of a large-scale scRNA-seq from human embryos."
