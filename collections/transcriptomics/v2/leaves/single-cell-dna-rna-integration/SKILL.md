---
name: single-cell-dna-rna-integration
description: Use when you have independent single-cell DNA copy-number and RNA expression
  matrices from the same sample (genes as rows, cells as columns) and need to associate
  RNA cells to DNA cells or infer cell-to-clone assignments.
when_to_use_negative:
- You need probabilistic confidence measures for assignments — MaCroDNA, unlike clonealign,
  does not provide them; use only the step number in cell2cell_step as a confidence
  proxy.
- Your sample is highly homogeneous — matching cells is distinctively challenging
  in homogeneous samples per the article's discussion.
- You have paired multi-omic measurements of the same cells (no cell association problem
  to solve) or no clone labels and only need cell-to-clone results.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
tools:
- name: MaCroDNA
  role: 'Core method: performs maximum weighted bipartite matching of per-gene read
    counts between scRNA-seq and scDNA-seq data for cell-to-cell and cell-to-clone
    assignment'
  repo: https://github.com/NakhlehLab/MaCroDNA
- name: Gurobi Optimizer
  role: Solves the binary integer program (via gurobipy) that implements the maximum
    weighted bipartite matching to optimality (tolerance 1.00e-04)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:036483eb811ef8a237b55c971e00d1739a7c0ff85c245f757193d1e653f3bfcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NakhlehLab/MaCroDNA
  tools:
  - Gurobi Optimizer
  - MaCroDNA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-dna-rna-integration

## Summary

Integrates scRNA-seq gene expression with scDNA-seq copy-number data by maximum weighted bipartite matching of per-gene Pearson correlations, using MaCroDNA (Gurobi MIP) to assign RNA cells to DNA cells or clones. Rationale: gene expression is assumed proportional to copy number, so correlated per-gene profiles identify the same cell across modalities.

## When to use

Use when you have independent single-cell DNA copy-number and RNA expression matrices from the same sample (genes as rows, cells as columns) and need to associate RNA cells to DNA cells or infer cell-to-clone assignments.

## When NOT to use

- You need probabilistic confidence measures for assignments — MaCroDNA, unlike clonealign, does not provide them; use only the step number in cell2cell_step as a confidence proxy.
- Your sample is highly homogeneous — matching cells is distinctively challenging in homogeneous samples per the article's discussion.
- You have paired multi-omic measurements of the same cells (no cell association problem to solve) or no clone labels and only need cell-to-clone results.

## Inputs

- dna_df: pandas dataframe of scDNA-seq copy-number values with genes as row indices and DNA cells as column indices
- rna_df: pandas dataframe of scRNA-seq gene expression values with genes as row indices and RNA cells as column indices
- dna_label (cell-to-clone mapping only): dataframe with columns 'clone' and 'cell' linking each DNA cell to its clone id

## Outputs

- cell2clone dataframe indexed by RNA cell ids with 'predict_cell' (matched DNA cell) and 'predict_clone' (predicted clone id) columns
- cell2cell dataframe indexed by RNA cell ids with a single 'predict_cell' column
- cell2cell_step dataframe with 'predict_cell' and 'step' columns (lower step = more confident assignment)

## How to apply

Load the scDNA copy-number table (dna_df), scRNA expression table (rna_df) with genes as row indices and cells as columns, and for clone-level analysis a dna_label table with 'clone' and 'cell' columns; MaCroDNA automatically selects the shared genes since gene sets may differ between modalities. Instantiate macrodna = MaCroDNA(rna_df, dna_df) for cell-to-cell mapping or MaCroDNA(rna_df, dna_df, dna_label) for cell-to-clone mapping. Run cell2cell_assignment() or cell2clone_assignment(), which internally maximizes the sum of per-gene Pearson correlation coefficients via maximum weighted bipartite matching solved as a binary integer program with Gurobi (MIP solved to optimality, tolerance 1.00e-04). For cell-to-cell mapping, the secondary output cell2cell_step reports a 'step' column where smaller step numbers indicate more confident assignments. Save the returned cell2clone dataframe (indexed by RNA cell ids, with 'predict_cell' and 'predict_clone' columns) as cell2clone_assignments.csv.

## Related tools

- **MaCroDNA** (Core method: performs maximum weighted bipartite matching of per-gene read counts between scRNA-seq and scDNA-seq data for cell-to-cell and cell-to-clone assignment) — https://github.com/NakhlehLab/MaCroDNA
- **Gurobi Optimizer** (Solves the binary integer program (via gurobipy) that implements the maximum weighted bipartite matching to optimality (tolerance 1.00e-04))

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df, dna_label); cell2clone = macrodna.cell2clone_assignment()
```

## Evaluation signals

- The underlying Gurobi MIP reports 'Optimal solution found (tolerance 1.00e-04)' with a 0.0000% gap between best objective and best bound.
- The number of associations in the correspondence matrix equals the expected RNA cell count (e.g. 4.0 for the 4-RNA-cell tiny test).
- For cell-to-clone mapping, every RNA cell id in the output index has non-null 'predict_cell' and 'predict_clone' values.
- The installation test (test_macrodna.tiny_test()) prints 'Finish Mapping / Test Success' with matching objective ~2.83001.
- Downstream validation: per-gene K* phylogenetic signal analysis should recover hundreds of genes with K* > 1 in HGD and EAC biopsies and identify ERBB2 as the highest-K* statistically significant COSMIC gene in two biopsies.

## Limitations

- MaCroDNA does not provide probabilistic confidence measures for the assignments (unlike statistical methods such as clonealign).
- Matching cells is distinctly challenging in homogeneous samples — a biologically driven issue for MaCroDNA.
- No ground truth correspondence exists for the BE dataset, so performance there was assessed only via resampling and stability analyses.
- Requires a Gurobi license (free academic license) and gurobipy; other dependencies (numpy, pandas, scipy, math) install automatically.

## Evidence

- [abstract] we present MaCroDNA, a method that uses maximum weighted bipartite matching of per-gene read counts from single-cell DNA and RNA-seq data.: "we present MaCroDNA, a method that uses maximum weighted bipartite matching of per-gene read counts from single-cell DNA and RNA-seq data."
- [discussion] MaCroDNA finds the correspondence between the scRNA-seq and scDNA-seq cells by maximizing the sum of the Pearson correlation coefficients: [evidence span withheld — anchor status: not_found]
- [readme] In these two dataframes, the row ids should be the genes and the column ids should be the cells.: "In these two dataframes, the row ids should be the genes and the column ids should be the cells."
- [readme] One is "predict_cell", which is the corresponding DNA cell for that RNA cell. The other column is "predict_clone", which is the predicted clone id for that RNA cell.: "One is "predict_cell", which is the corresponding DNA cell for that RNA cell. The other column is "predict_clone", which is the predicted clone id for that RNA cell."
- [discussion] unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments: "unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments"
