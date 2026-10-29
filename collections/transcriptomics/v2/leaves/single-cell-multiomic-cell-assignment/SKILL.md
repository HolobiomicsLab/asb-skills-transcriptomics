---
name: single-cell-multiomic-cell-assignment
description: Use when you have two independent single-cell datasets from the same
  sample — scDNA-seq copy-number profiles and scRNA-seq expression profiles — restricted
  to a shared gene set, and need a one-to-one cell correspondence or RNA-cell-to-DNA-clone
  mapping.
when_to_use_negative:
- You need probabilistic confidence measures for assignments — unlike statistical
  methods such as clonealign, MaCroDNA does not provide them
- The sample is biologically homogeneous — the paper notes matching cells is distinctively
  challenging in homogeneous samples, a biologically driven issue
- You have paired multi-omic measurements of the same cells (e.g. scTrio-seq with
  known correspondence) — no computational association is needed
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
tools:
- name: MaCroDNA
  role: 'Core solver: computes per-gene Pearson correlations between scRNA-seq and
    scDNA-seq profiles and infers cell correspondence via maximum weighted bipartite
    matching'
  repo: github.com/NakhlehLab/MaCroDNA
- name: Gurobi
  role: Solves the weighted maximum bipartite matching as a binary mixed-integer program
    (gurobipy; requires free academic license)
- name: pandas
  role: Provides the input dataframe format (dna_df, rna_df, dna_label) and returns
    assignment results as dataframes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:47815c5429d1781f49a8bb97c0592849d3e1aa8e5553415bb6609de150d70258
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gurobi
  - MaCroDNA
  - pandas
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# single-cell-multiomic-cell-assignment

## Summary

Infers cell-to-cell and cell-to-clone correspondence between scRNA-seq (gene expression) and scDNA-seq (copy number) data using MaCroDNA's maximum weighted bipartite matching over per-gene Pearson correlation coefficients, solved as a Gurobi binary mixed-integer program (with a sequential heuristic when RNA cells outnumber DNA cells).

## When to use

Use when you have two independent single-cell datasets from the same sample — scDNA-seq copy-number profiles and scRNA-seq expression profiles — restricted to a shared gene set, and need a one-to-one cell correspondence or RNA-cell-to-DNA-clone mapping.

## When NOT to use

- You need probabilistic confidence measures for assignments — unlike statistical methods such as clonealign, MaCroDNA does not provide them
- The sample is biologically homogeneous — the paper notes matching cells is distinctively challenging in homogeneous samples, a biologically driven issue
- You have paired multi-omic measurements of the same cells (e.g. scTrio-seq with known correspondence) — no computational association is needed

## Inputs

- dna_df: pandas dataframe of scDNA-seq CNA values, genes as rows, cells as columns
- rna_df: pandas dataframe of scRNA-seq gene expression values, genes as rows, cells as columns
- dna_label: pandas dataframe with columns 'clone' and 'cell' (only for cell-to-clone mapping)

## Outputs

- cell2cell: dataframe indexed by RNA cell ids with column 'predict_cell' (assigned DNA cell)
- cell2cell_step: dataframe with columns 'predict_cell' and 'step', where smaller step numbers indicate more confident assignments
- cell2clone: dataframe indexed by RNA cell ids with columns 'predict_cell' and 'predict_clone'

## How to apply

Load dna_df and rna_df as pandas dataframes with genes as rows and cells as columns; MaCroDNA restricts both to the shared gene set itself, so the gene lists need not match. Compute per-gene Pearson correlation coefficients between each RNA cell's expression profile and each DNA cell's CNA profile to weight the bipartite matching edges. When the number of RNA cells is at most the number of DNA cells, solve the weighted maximum bipartite matching as a binary mixed-integer program with Gurobi to obtain the one-to-one correspondence matrix. When RNA cells exceed DNA cells, MaCroDNA applies an iterative constraint-relaxed heuristic that decomposes the assignment into a series of maximum weighted bipartite matching problems solved sequentially, and records at which step each RNA cell was assigned (smaller step = more confident). For cell-to-clone mapping, additionally supply dna_label with 'clone' and 'cell' columns and call cell2clone_assignment().

## Related tools

- **MaCroDNA** (Core solver: computes per-gene Pearson correlations between scRNA-seq and scDNA-seq profiles and infers cell correspondence via maximum weighted bipartite matching) — github.com/NakhlehLab/MaCroDNA
- **Gurobi** (Solves the weighted maximum bipartite matching as a binary mixed-integer program (gurobipy; requires free academic license))
- **pandas** (Provides the input dataframe format (dna_df, rna_df, dna_label) and returns assignment results as dataframes)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df); cell2cell, cell2cell_step = macrodna.cell2cell_assignment()
```

## Evaluation signals

- Run tiny_test(): the Gurobi best objective must equal 2.830007718086e+00 with a 0.0000% optimality gap
- tiny_test() must report 4.0 associations in the correspondence matrix and end with 'Test Success'
- On real data, the produced cell2cell matrix must match the reference implementation's assignments on a CRC patient sample
- cell2cell_step values should be consistent: every RNA cell in cell2cell has a matching 'predict_cell' and a recorded step, with smaller steps denoting more confident assignments
- Gurobi log should show 'Optimal solution found (tolerance 1.00e-04)' for the mixed-integer model (9 rows, 16 columns, 48 nonzeros on the tiny test)

## Limitations

- MaCroDNA does not provide probabilistic confidence measures for the assignments, unlike statistical methods such as clonealign
- Matching cells is distinctively challenging in homogeneous samples — a biologically driven issue when utilizing MaCroDNA in such cases
- Lack of ground-truth correspondence (e.g. for the BE dataset) forces indirect validation via resampling and stability analyses
- Realistic simulation-based evaluation is difficult because knowledge of the DNA-to-RNA process is too limited to produce realistic simulations

## Evidence

- [discussion] MaCroDNA finds the correspondence between the scRNA-seq and scDNA-seq cells by maximizing the sum of the Pearson correlation coefficients: "Our method, named MaCroDNA, finds the correspondence between the scRNA-seq and scDNA-seq cells by maximizing the sum of the Pearson correlation coefficients"
- [abstract] a method that uses maximum weighted bipartite matching of per-gene read counts from single-cell DNA and RNA-seq data: "we present MaCroDNA, a method that uses maximum weighted bipartite matching of per-gene read counts from single-cell DNA and RNA-seq data"
- [readme] The genes in RNA data and those in DNA data can be different because MaCroDNA can select the same genes from the data by itself: "The genes in RNA data and those in DNA data can be different because MaCroDNA can select the same genes
 from the data by itself."
- [readme] "step" indicates at which step this result is inferred. And the smaller the step number is, the more confident the result is: ""step" indicates at which step this result is inferred. And the smaller the step number is, the more confident the result is."
- [discussion] unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments: "unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments"
