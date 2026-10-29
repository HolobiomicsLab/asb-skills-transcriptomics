---
name: log-transformation-of-copy-number-data
description: Use when benchmarking cell-to-clone assignment accuracy across clustering
  algorithms and preprocessing choices, where both original and log-transformed copy-number
  values must be fed to intNMF or agglomerative clustering before running MaCroDNA.
when_to_use_negative:
- When the goal is cell-to-cell mapping rather than clone assignment — clone labels
  (dna_label) are not required and cell2cell_assignment() should be used instead
- When a downstream method models copy numbers directly (e.g. clonealign's variational
  ELBO-based probabilistic model) rather than consuming pre-clustered clone labels
  — the clustering/transformation parameterization applies to the MaCroDNA matching
  workflow
- When working with the Barrett's esophagus dataset, where no ground truth correspondence
  between cells exists and resampling/stability analyses are used instead of accuracy
  evaluation
edam_operation: http://edamontology.org/operation_3432
tools:
- name: MaCroDNA
  role: Receives cluster-derived clone labels (dna_label) built from raw or log-transformed
    copy numbers and performs multi-step maximum weighted bipartite matching of Pearson
    correlation coefficients to assign RNA cells to DNA clones
  repo: github.com/NakhlehLab/MaCroDNA
- name: intNMF
  role: Clustering algorithm applied to the copy-number matrix in both original and
    log-transformed form to generate the clone configurations being compared
- name: Gurobi
  role: MIP solver used by MaCroDNA's bipartite matching on each clustering/transformation
    configuration
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:b2c9d4d537657e093ed54ef8b074ce856a084be2fdc82e4d51d2da4480ba6ce9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gurobi
  - MaCroDNA
  - Seurat
  - clonealign
  - intNMF
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# log-transformation-of-copy-number-data

## Summary

Applies a log transformation to scDNA-seq copy-number values before clustering (intNMF or agglomerative), as one of four clustering/transformation parameterizations used to test how sensitive cell-to-clone assignment accuracy is to copy-number preprocessing in the CRC scTrio-seq benchmark.

## When to use

Use when benchmarking cell-to-clone assignment accuracy across clustering algorithms and preprocessing choices, where both original and log-transformed copy-number values must be fed to intNMF or agglomerative clustering before running MaCroDNA.

## When NOT to use

- When the goal is cell-to-cell mapping rather than clone assignment — clone labels (dna_label) are not required and cell2cell_assignment() should be used instead
- When a downstream method models copy numbers directly (e.g. clonealign's variational ELBO-based probabilistic model) rather than consuming pre-clustered clone labels — the clustering/transformation parameterization applies to the MaCroDNA matching workflow
- When working with the Barrett's esophagus dataset, where no ground truth correspondence between cells exists and resampling/stability analyses are used instead of accuracy evaluation

## Inputs

- scDNA-seq copy-number dataframe (dna_df: genes as rows, cells as columns) for patients CRC04, CRC10, CRC11 from the Bian et al. scTrio-seq2 dataset
- scRNA-seq gene-expression dataframe (rna_df: genes as rows, cells as columns)
- DNA clone labels dataframe (dna_label with 'clone' and 'cell' columns)
- Partially known ground-truth joint RNA/DNA clone labels (evaluation only)

## Outputs

- Accuracy distributions per patient and per method/parameterization written to cell2clone_accuracy.csv
- cell2clone dataframe from macrodna.cell2clone_assignment() with 'predict_cell' and 'predict_clone' columns indexed by RNA cell ids

## How to apply

Take the per-patient scDNA-seq copy-number matrix (dna_df, rows=genes, columns=cells) from the Bian et al. CRC scTrio-seq2 dataset and generate two variants per clustering method: one with the original copy-number values and one with log-transformed values. Cluster each variant with intNMF and with agglomerative clustering of the copy numbers, yielding four clustering/transformation configurations per patient (CRC04, CRC10, CRC11). Feed the resulting clone assignments as the dna_label dataframe (columns 'clone' and 'cell') into macrodna.cell2clone_assignment(), which performs multi-step maximum weighted bipartite matching of Pearson correlation coefficients between RNA expression and DNA copy-number profiles via the Gurobi MIP solver. Compute per-patient, per-configuration assignment accuracy against the partially known ground-truth joint RNA/DNA clone labels (used only for evaluation) and write the distributions to cell2clone_accuracy.csv. The rationale is to verify that MaCroDNA's reported advantage — lowest variance and highest median accuracy — holds regardless of whether copy numbers are raw or log-scaled before clustering.

## Related tools

- **MaCroDNA** (Receives cluster-derived clone labels (dna_label) built from raw or log-transformed copy numbers and performs multi-step maximum weighted bipartite matching of Pearson correlation coefficients to assign RNA cells to DNA clones) — github.com/NakhlehLab/MaCroDNA
- **intNMF** (Clustering algorithm applied to the copy-number matrix in both original and log-transformed form to generate the clone configurations being compared)
- **Gurobi** (MIP solver used by MaCroDNA's bipartite matching on each clustering/transformation configuration)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df, dna_label); cell2clone = macrodna.cell2clone_assignment()
```

## Evaluation signals

- The per-patient accuracy distributions reproduce the ordering reported in Fig. 2 of the paper, with MaCroDNA outperforming clonealign and Seurat on the CRC ground truth
- MaCroDNA shows the lowest variance and highest median accuracy among all methods across all four clustering/transformation configurations
- Both the original and log-transformed copy-number variants are represented for each clustering algorithm (intNMF and agglomerative), i.e. four configurations per patient are present in cell2clone_accuracy.csv
- Clone labels passed as dna_label must contain exactly two columns, 'clone' and 'cell', matching the README-specified input schema
- The assignment step completes with the Gurobi MIP reporting an optimal solution (gap 0) for each configuration

## Limitations

- Ground-truth clone labels are only partially known in the CRC scTrio-seq data, so accuracy is computed against a partial reference
- MaCroDNA does not provide probabilistic confidence measures for the assignments, unlike statistical methods such as clonealign
- Limited knowledge of the DNA-to-RNA process prevents realistic simulation studies for evaluating the transformation configurations
- Matching cells is distinctively challenging in homogeneous samples, a biologically driven issue that may affect assignment quality regardless of preprocessing

## Evidence

- [other] task card workflow: [evidence span withheld — anchor status: not_found]
- [other] task card validation: [evidence span withheld — anchor status: not_found]
- [readme] README usage: "For mapping RNA cells to DNA clones, the input needs three dataframes `dna_df`, `rna_df` and `dna_label`."
- [discussion] article discussion: "We evaluated the accuracy of the methods for two inference tasks, namely the cell-to-clone assignment and prediction of the clonal prevalences"
