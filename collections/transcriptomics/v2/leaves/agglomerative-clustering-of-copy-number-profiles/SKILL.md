---
name: agglomerative-clustering-of-copy-number-profiles
description: Use when scDNA-seq cells have copy-number (CNA) profiles but no prior
  clone labels, and a clone partition is needed as input to RNA-to-clone mapping (e.g.,
  MaCroDNA cell2clone_assignment) or when benchmarking clone definitions across clustering/transformation
  configurations.
when_to_use_negative:
- Ground-truth clone labels are already available (e.g., the dna_label 'clone'/'cell'
  columns from scTrio-seq joint genotyping) — skip clustering and use the labels directly
- The goal is to evaluate a method on the BE dataset, where no ground truth cell correspondence
  exists and only resampling/stability analyses apply
- Comparing against probabilistic assignment methods when confidence measures are
  required — agglomerative clustering plus MaCroDNA does not provide probabilistic
  confidence for assignments
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MaCroDNA
  role: Consumes the agglomerative clone partition as dna_label and performs multi-step
    maximum weighted bipartite matching of Pearson correlation coefficients (Gurobi
    MIP) for cell-to-clone assignment
  repo: github.com/NakhlehLab/MaCroDNA
- name: Gurobi
  role: MIP solver used inside macrodna.cell2clone_assignment() for the weighted bipartite
    matching of RNA cells to DNA clones
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:a3d023676e25995f17e342eb053535a99c4cfef0e26057412e6b48e31d8c3a9c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gurobi
  - MaCroDNA
  - Seurat
  - clonealign
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# agglomerative-clustering-of-copy-number-profiles

## Summary

Agglomerative (hierarchical) clustering of scDNA-seq copy-number profiles into clones, applied in the MaCroDNA CRC scTrio-seq benchmark as one of two clone-partitioning strategies (with intNMF) evaluated under both original and log-transformed copy-number values before cell-to-clone assignment.

## When to use

Use when scDNA-seq cells have copy-number (CNA) profiles but no prior clone labels, and a clone partition is needed as input to RNA-to-clone mapping (e.g., MaCroDNA cell2clone_assignment) or when benchmarking clone definitions across clustering/transformation configurations.

## When NOT to use

- Ground-truth clone labels are already available (e.g., the dna_label 'clone'/'cell' columns from scTrio-seq joint genotyping) — skip clustering and use the labels directly
- The goal is to evaluate a method on the BE dataset, where no ground truth cell correspondence exists and only resampling/stability analyses apply
- Comparing against probabilistic assignment methods when confidence measures are required — agglomerative clustering plus MaCroDNA does not provide probabilistic confidence for assignments

## Inputs

- scDNA-seq copy-number dataframe (dna_df: genes as row ids, DNA cells as columns), e.g. from the Bian et al. CRC scTrio-seq2 dataset for patients CRC04/CRC10/CRC11
- scRNA-seq gene-expression dataframe (rna_df: genes as rows, RNA cells as columns) for downstream cell2clone mapping
- Choice of copy-number transformation: original vs. log-transformed values
- Partially known ground-truth joint RNA/DNA clone labels (used only for evaluation, not clustering)

## Outputs

- A clone partition of DNA cells, formatted as a dna_label dataframe with 'clone' and 'cell' columns
- cell2clone dataframe (index = RNA cell ids; columns 'predict_cell' and 'predict_clone')
- Per-patient/method/configuration cell-to-clone assignment accuracy (e.g., written to cell2clone_accuracy.csv)

## How to apply

Start from the copy-number dataframe (dna_df: genes as row ids, DNA cells as columns) for a patient such as CRC04, CRC10, or CRC11 from the Bian et al. scTrio-seq dataset. Run agglomerative clustering on the copy-number profiles to partition DNA cells into clones, once with the original copy-number values and once with log-transformed values, giving four clustering/transformation configurations when paired with intNMF. Convert the resulting clone partition into the dna_label format required by MaCroDNA (two columns 'clone' and 'cell'), then call macrodna.cell2clone_assignment(), which performs multi-step maximum weighted bipartite matching of Pearson correlation coefficients between RNA expression and DNA copy-number profiles via the Gurobi MIP solver. Evaluate with partially known ground-truth joint RNA/DNA clone labels only after assignment, computing per-patient accuracy to compare against clonealign and Seurat, reproducing the ordering in Fig. 2 of the paper.

## Related tools

- **MaCroDNA** (Consumes the agglomerative clone partition as dna_label and performs multi-step maximum weighted bipartite matching of Pearson correlation coefficients (Gurobi MIP) for cell-to-clone assignment) — github.com/NakhlehLab/MaCroDNA
- **Gurobi** (MIP solver used inside macrodna.cell2clone_assignment() for the weighted bipartite matching of RNA cells to DNA clones)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df, dna_label); cell2clone = macrodna.cell2clone_assignment()
```

## Evaluation signals

- Per-patient accuracy ordering matches Fig. 2 of the paper: MaCroDNA shows the lowest variance and highest median accuracy across all four clustering/transformation configurations (intNMF and agglomerative clustering, each with original and log-transformed copy numbers)
- dna_label output has exactly two columns, 'clone' and 'cell', with every DNA cell assigned to one clone id
- cell2clone output contains 'predict_cell' and 'predict_clone' columns for all RNA cell ids
- Gurobi reports 'Optimal solution found' (MIP gap at tolerance 1.00e-04) during each matching step, and the README tiny_test prints 'Test Success'
- Accuracy is computed only against partially known ground-truth joint RNA/DNA labels, never used during fitting

## Limitations

- Matching cells is distinctively challenging in homogeneous samples, a biologically driven issue for MaCroDNA-based cell-to-clone pipelines
- Unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments
- Limited knowledge of the DNA-to-RNA process prevents realistic simulation studies for evaluating clone-partition choices

## Evidence

- [other] agglomerative clustering as alternative clone definition: [evidence span withheld — anchor status: not_found]
- [readme] dna_label input format for cell2clone: "For mapping RNA cells to DNA clones, the input needs three dataframes `dna_df`, `rna_df` and `dna_label`."
- [readme] dna_label schema: "`dna_label` has two columns "clone" and "cell", where "cell" is the DNA cells and "clone" is the corresponding clone id for that DNA cell."
- [other] MaCroDNA assignment mechanism: [evidence span withheld — anchor status: not_found]
- [other] ground-truth ordering expected: [evidence span withheld — anchor status: not_found]
