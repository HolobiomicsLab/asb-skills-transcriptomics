---
name: functional-enrichment-qvalue-interpretation
description: Use when a ranked or filtered gene list (e.g., top-correlated marker
  genes) needs functional interpretation via GO biological process enrichment with
  reported q-values, such as genes linked to latent features from dimension reduction.
when_to_use_negative:
- The gene list is not derived from a defined ranking or filtering criterion (e.g.,
  rank <= 100 in a latent feature), so enrichment results would be uninterpretable.
- The analysis goal is dimension reduction or visualization itself rather than functional
  interpretation of an already-derived candidate gene set.
- The organism or gene identifiers are not supported by DAVID 6.8's annotation databases.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: VASC
  role: Produces the 2D latent features that genes are correlated against to define
    candidate marker gene lists
  repo: github.com/wang-research/VASC
- name: Python
  role: Environment for computing Spearman rank correlations and handling the expression
    matrix
- name: numpy
  role: Numerical computation of gene-latent feature rank correlations
- name: sklearn
  role: Supports the correlation computation on the single-cell expression matrix
- name: h5py
  role: Loads the single-cell expression data used for correlation analysis
- name: Anaconda
  role: Recommended Python environment distribution for installing prerequisites
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:ff55c1c2ccf7e199d6c2df0386a8db2d7a63e4bf9fcf20c73c3da9dfc1ccaca6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Anaconda
  - Python
  - VASC
  - h5py
  - numpy
  - sklearn
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# functional-enrichment-qvalue-interpretation

## Summary

Interpret DAVID 6.8 Gene Ontology biological process enrichment results for a ranked candidate gene list — here the top Spearman-correlated genes from the VASC 2D latent features of the Petropoulos human pre-implantation embryo dataset — recording gene counts and q-values for each enriched term.

## When to use

Use when a ranked or filtered gene list (e.g., top-correlated marker genes) needs functional interpretation via GO biological process enrichment with reported q-values, such as genes linked to latent features from dimension reduction.

## When NOT to use

- The gene list is not derived from a defined ranking or filtering criterion (e.g., rank <= 100 in a latent feature), so enrichment results would be uninterpretable.
- The analysis goal is dimension reduction or visualization itself rather than functional interpretation of an already-derived candidate gene set.
- The organism or gene identifiers are not supported by DAVID 6.8's annotation databases.

## Inputs

- Single-cell gene expression matrix (Petropoulos human pre-implantation embryo dataset)
- VASC 2D latent feature representation of embryo cells (produced via the VASC class in vasc.py)
- Top-correlated gene lists (rank <= 100 in either VASC latent feature)

## Outputs

- david_enrichment_results.csv — enrichment table with GO biological process terms, gene counts, and q-values
- List of significantly enriched GO terms with gene counts and q-values (e.g., carbohydrate derivative metabolic process: 37 genes, q = 5.63E-05)

## How to apply

First compute Spearman's rank correlation between every gene's expression profile and each of the two VASC 2D latent features (from the VASC class in vasc.py) using numpy/sklearn, then retain genes with rank <= 100 in either feature. Submit these top-correlated gene lists to DAVID 6.8 for Gene Ontology biological process enrichment analysis. Record for each enriched term the gene count and q-value, e.g. carbohydrate derivative metabolic process (37 genes, q = 5.63E-05), oxidation-reduction process (32 genes, q = 4.87E-05), and lipid metabolic process (32 genes, q = 4.94E-03). Write the resulting enrichment table to david_enrichment_results.csv. The rationale is that genes whose expression tracks the embryo development trajectory captured by VASC's latent space should concentrate in metabolic processes active during pre-implantation development.

## Related tools

- **VASC** (Produces the 2D latent features that genes are correlated against to define candidate marker gene lists) — github.com/wang-research/VASC
- **Python** (Environment for computing Spearman rank correlations and handling the expression matrix)
- **numpy** (Numerical computation of gene-latent feature rank correlations)
- **sklearn** (Supports the correlation computation on the single-cell expression matrix)
- **h5py** (Loads the single-cell expression data used for correlation analysis)
- **Anaconda** (Recommended Python environment distribution for installing prerequisites)

## Examples

```
from vasc import VASC; import numpy as np; from scipy.stats import spearmanr; rho, _ = spearmanr(expr_matrix, latent_2d, axis=1); top = np.argsort(np.abs(rho))[:100]
```

## Evaluation signals

- The correlation step yields two gene lists whose members are ranked by absolute Spearman correlation, with the cutoff set at rank <= 100 in either VASC feature.
- The enrichment table contains, at minimum, the three metabolic process terms with the reported gene counts and q-values: carbohydrate derivative metabolic process (37 genes, q = 5.63E-05), oxidation-reduction process (32 genes, q = 4.87E-05), lipid metabolic process (32 genes, q = 4.94E-03).
- All reported q-values are small (on the order of 1E-03 or smaller), consistent with significant enrichment after multiple-testing correction in DAVID 6.8.
- The output file david_enrichment_results.csv exists and contains term names, gene counts, and q-value columns.

## Limitations

- The article reports only the three top metabolic process terms; other enriched terms or their thresholds are not documented in the surfaced context.
- VASC's stochastic optimization introduces run-to-run variation in the latent representation, so repeated VASC runs are recommended and may slightly alter the ranked gene lists feeding the enrichment.
- Determining the optimal latent dimension is difficult without prior knowledge, which affects which features are used for correlation.

## Evidence

- [other] Compute Spearman's rank correlation between every gene's expression profile and each of the two latent features in Python using numpy/sklearn.: [evidence span withheld — anchor status: not_found]
- [other] Submit the top-correlated gene lists to DAVID 6.8 for Gene Ontology biological process enrichment analysis.: [evidence span withheld — anchor status: not_found]
- [other] carbohydrate derivative metabolic process (37 genes, q = 5.63E-05 by DAVID 6.8), oxidation-reduction process (32 genes, q = 4.87E-05), and lipid metabolic process (32 genes, q = 4.94E-03): [evidence span withheld — anchor status: not_found]
- [methods] vasc.py: contains a class VASC and a function vase: "vasc.py: contains a class VASC and a function vase"
- [abstract] VASC successfully re-establishes the cell dynamics in pre-implantation embryos and identifies several candidate marker genes associated with early embryo development: "VASC successfully re-establishes the cell dynamics in pre-implantation embryos and identifies several candidate marker genes associated with early embryo development"
