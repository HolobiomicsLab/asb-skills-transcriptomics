---
name: intnmf-copy-number-clustering
description: Use when scDNA-seq copy-number data must be partitioned into clones (clone
  labels for DNA cells) before evaluating or running cell-to-clone assignment of scRNA-seq
  cells, and multiple clustering algorithms/transformations should be compared.
when_to_use_negative:
- Ground-truth clone labels already exist for the DNA cells — clustering would only
  add a confound; use the known labels directly as dna_label.
- The goal is cell-to-cell (not cell-to-clone) mapping, which needs only dna_df and
  rna_df and no clone partition.
- The sample is biologically homogeneous — the article notes matching cells is distinctively
  challenging in homogeneous samples.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3673
tools:
- name: intNMF
  role: Clusters DNA cells by copy-number values to define clones under one of the
    four clustering/transformation configurations
- name: MaCroDNA
  role: Consumes the intNMF-derived clone labels via cell2clone_assignment(), performing
    multi-step maximum weighted bipartite matching of Pearson correlation coefficients
    with the Gurobi MIP solver
  repo: github.com/NakhlehLab/MaCroDNA
- name: clonealign
  role: Comparator that maps RNA cells to the intNMF-defined DNA clones under its
    variational (ELBO-based) probabilistic model
- name: Seurat
  role: Comparator representing commonly used single-cell multi-omic integration approaches
    applied to the same clustered clones
- name: Gurobi
  role: MIP solver used by MaCroDNA's maximum weighted bipartite matching during cell-to-clone
    assignment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:9bf4b6311259a1bdddccccd518d03162a916272b040af0662f978f2dba74a047
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
  edam_topics:
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# intnmf-copy-number-clustering

## Summary

Cluster scDNA-seq copy-number profiles with intNMF (and agglomerative clustering as a comparator) to define DNA clones, then use the resulting clone labels as inputs for cell-to-clone assignment benchmarking of MaCroDNA, clonealign, and Seurat on the Bian et al. CRC scTrio-seq dataset.

## When to use

Use when scDNA-seq copy-number data must be partitioned into clones (clone labels for DNA cells) before evaluating or running cell-to-clone assignment of scRNA-seq cells, and multiple clustering algorithms/transformations should be compared.

## When NOT to use

- Ground-truth clone labels already exist for the DNA cells — clustering would only add a confound; use the known labels directly as dna_label.
- The goal is cell-to-cell (not cell-to-clone) mapping, which needs only dna_df and rna_df and no clone partition.
- The sample is biologically homogeneous — the article notes matching cells is distinctively challenging in homogeneous samples.

## Inputs

- scDNA-seq copy-number dataframe dna_df (rows = genes, columns = cells) for patients CRC04, CRC10, CRC11 from the Bian et al. scTrio-seq2 CRC dataset
- scRNA-seq gene-expression dataframe rna_df (rows = genes, columns = cells)
- Choice of clustering algorithm (intNMF or agglomerative) and copy-number transformation (original or log-transformed)
- Partially known ground-truth joint RNA/DNA clone labels (evaluation only)

## Outputs

- DNA clone labels per patient per configuration (dna_label-style dataframe with 'clone' and 'cell' columns)
- Cell-to-clone assignment predictions per method (e.g., MaCroDNA's cell2clone dataframe with 'predict_cell' and 'predict_clone' columns)
- Per-patient, per-method/parameterization cell-to-clone assignment accuracy distributions written to cell2clone_accuracy.csv

## How to apply

For each patient (CRC04, CRC10, CRC11) in the Bian et al. CRC scTrio-seq2 dataset, take the copy-number dataframe dna_df (genes x cells) and cluster the DNA cells with intNMF; repeat with agglomerative clustering as a second algorithm. For each clustering algorithm run the assignment twice — once on the original copy-number values and once on log-transformed copy-number values — yielding four clustering/transformation configurations per method. Feed the resulting clone labels (as a dna_label dataframe with 'clone' and 'cell' columns, per the MaCroDNA README) into MaCroDNA's cell2clone_assignment(), clonealign's variational (ELBO-based) model, and Seurat as the common multi-omic integration representative. Rationale: the partially known ground-truth clone labels are used solely for evaluation, so clustering-derived clones let you test how robust each method's cell-to-clone assignment accuracy is to how the DNA clones were defined.

## Related tools

- **intNMF** (Clusters DNA cells by copy-number values to define clones under one of the four clustering/transformation configurations)
- **MaCroDNA** (Consumes the intNMF-derived clone labels via cell2clone_assignment(), performing multi-step maximum weighted bipartite matching of Pearson correlation coefficients with the Gurobi MIP solver) — github.com/NakhlehLab/MaCroDNA
- **clonealign** (Comparator that maps RNA cells to the intNMF-defined DNA clones under its variational (ELBO-based) probabilistic model)
- **Seurat** (Comparator representing commonly used single-cell multi-omic integration approaches applied to the same clustered clones)
- **Gurobi** (MIP solver used by MaCroDNA's maximum weighted bipartite matching during cell-to-clone assignment)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df, dna_label); cell2clone = macrodna.cell2clone_assignment()
```

## Evaluation signals

- Four configurations are produced per method: intNMF and agglomerative clustering, each with original and log-transformed copy-number values.
- MaCroDNA shows the lowest variance and highest median cell-to-clone assignment accuracy among MaCroDNA, clonealign, and Seurat across all four configurations.
- Per-patient accuracy distributions reproduce the ordering reported in Fig. 2 of the paper (MaCroDNA > clonealign, with clonealign the poorest).
- clonealign shows the poorest performance among the three methods, consistent with the paper's discussion.
- Output accuracy distributions are written to cell2clone_accuracy.csv with one accuracy per patient and method/parameterization.

## Limitations

- Ground-truth joint clone labels exist only for the CRC dataset; for datasets like Barrett's esophagus (BE) there is no ground truth for cell correspondence, so only resampling/stability analyses are possible.
- Lack of a detailed description of clonealign's ELBO hindered determining the root cause of its behavior.
- Limited knowledge of the DNA-to-RNA process prevents realistic simulation studies for evaluation.
- MaCroDNA does not provide probabilistic confidence measures for the assignments, unlike statistical methods such as clonealign.

## Evidence

- [other] workflow step 5 - clustering configs: [evidence span withheld — anchor status: not_found]
- [readme] readme - clone labels format: "`dna_label` has two columns "clone" and "cell", where "cell" is the DNA cells and "clone" is the corresponding clone id for that DNA cell."
- [readme] readme - cell2clone API: "> macrodna = MaCroDNA(rna_df, dna_df, dna_label)
> cell2clone = macrodna.cell2clone_assignment()"
- [discussion] discussion - clonealign poorest: "Among the three methods, clonealign showed the poorest performance"
- [discussion] discussion - MaCroDNA outperformance: [evidence span withheld — anchor status: not_found]
