---
name: iterative-constraint-relaxed-assignment-heuristic
description: Use when associating single-cell RNA-seq expression profiles to single-cell
  DNA-seq CNA profiles one-to-one and the number of RNA cells exceeds the number of
  DNA cells, so a single bipartite matching cannot assign every RNA cell.
when_to_use_negative:
- The sample is biologically homogeneous — the article notes that 'matching cells
  is distinctively challenging in homogeneous samples' and this is a biologically
  driven failure mode
- You need probabilistic confidence measures for assignments — MaCroDNA does not provide
  these, unlike statistical methods such as clonealign
- You require probabilistic clone-assignment or per-cell posteriors rather than a
  hard one-to-one cell correspondence — clonealign may be more appropriate
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: MaCroDNA
  role: Implements the Pearson-correlation-based sequential maximum weighted bipartite
    matching; source at src/MaCroDNA/macrodna.py
  repo: https://github.com/NakhlehLab/MaCroDNA
- name: Gurobi (gurobipy)
  role: Solves the maximum weighted bipartite matching as a binary mixed-integer program
    at each sequential step
- name: pandas
  role: Handles the dna_df/rna_df input dataframes and the cell2cell / cell2cell_step
    output dataframes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:79c83d3cec4a63863b941898e7730b324de55c14e116f327184a5cab2453c2e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NakhlehLab/MaCroDNA
  tools:
  - Gurobi (gurobipy)
  - MaCroDNA
  - pandas
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# iterative-constraint-relaxed-assignment-heuristic

## Summary

MaCroDNA's sequential maximum-weighted-bipartite-matching heuristic for the scRNA-seq-to-scDNA-seq cell association problem: when RNA cells outnumber DNA cells, the assignment is decomposed into a series of Gurobi-solved matching problems run step by step, with the step number recording assignment confidence.

## When to use

Use when associating single-cell RNA-seq expression profiles to single-cell DNA-seq CNA profiles one-to-one and the number of RNA cells exceeds the number of DNA cells, so a single bipartite matching cannot assign every RNA cell.

## When NOT to use

- The sample is biologically homogeneous — the article notes that 'matching cells is distinctively challenging in homogeneous samples' and this is a biologically driven failure mode
- You need probabilistic confidence measures for assignments — MaCroDNA does not provide these, unlike statistical methods such as clonealign
- You require probabilistic clone-assignment or per-cell posteriors rather than a hard one-to-one cell correspondence — clonealign may be more appropriate

## Inputs

- dna_df: pandas dataframe of scDNA-seq copy number values, genes as row ids, cells as column ids
- rna_df: pandas dataframe of scRNA-seq gene expression values, genes as row ids, cells as column ids
- dna_label (optional, for cell-to-clone mode): dataframe with 'clone' and 'cell' columns mapping DNA cells to clone ids

## Outputs

- cell2cell: dataframe indexed by RNA cell ids with a single 'predict_cell' column giving the assigned DNA cell
- cell2cell_step: dataframe with 'predict_cell' and 'step' columns, where the step number indicates at which matching iteration the assignment was inferred (smaller = more confident)
- cell2clone (clone mode): dataframe indexed by RNA cell ids with 'predict_cell' and 'predict_clone' columns

## How to apply

Load dna_df and rna_df dataframes with genes as rows and cells as columns, and restrict both to the shared gene set (MaCroDNA selects common genes itself). Compute per-gene Pearson correlation coefficients between each scRNA-seq cell's expression profile and each scDNA-seq cell's CNA profile to weight the bipartite matching edges — the hypothesis is that gene expression is proportional to copy number. If the number of RNA cells is at most the number of DNA cells, solve the weighted maximum bipartite matching once as a binary mixed-integer program with gurobipy. If RNA cells exceed DNA cells, apply the iterative constraint-relaxed heuristic: solve a series of maximum weighted bipartite matching problems sequentially, one after another, recording at which step each RNA cell was assigned (smaller step = more confident). Return cell2cell (RNA cell id → predicted DNA cell) and cell2cell_step (predicted cell plus step number) from macrodna.cell2cell_assignment().

## Related tools

- **MaCroDNA** (Implements the Pearson-correlation-based sequential maximum weighted bipartite matching; source at src/MaCroDNA/macrodna.py) — https://github.com/NakhlehLab/MaCroDNA
- **Gurobi (gurobipy)** (Solves the maximum weighted bipartite matching as a binary mixed-integer program at each sequential step)
- **pandas** (Handles the dna_df/rna_df input dataframes and the cell2cell / cell2cell_step output dataframes)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df); cell2cell, cell2cell_step = macrodna.cell2cell_assignment()
```

## Evaluation signals

- tiny_test() on the built-in 4-cell synthetic DNA/RNA matrices must report 'Best objective 2.830007718086e+00' with a 0.0000% optimality gap
- tiny_test() must report 'the number of associations in the correspondence matrix 4.0' and finish with 'Test Success'
- The number of associations in the correspondence matrix must equal the number of RNA cells (each RNA cell assigned exactly one DNA cell)
- cell2cell_step step values are positive integers; smaller step numbers should correspond to the most confident assignments
- The correspondence matrix on the CRC patient sample must match the reference implementation's assignments

## Limitations

- MaCroDNA does not provide probabilistic confidence measures for the assignments, unlike statistical methods such as clonealign
- Matching cells is distinctively challenging in homogeneous samples — a biologically driven issue when utilizing MaCroDNA in such cases
- Limited knowledge of the DNA-to-RNA process prevents realistic simulation studies for evaluation
- Absence of ground truth for some datasets (e.g., the BE dataset) forces reliance on resampling and stability analyses instead of accuracy metrics

## Evidence

- [discussion] MaCroDNA finds the correspondence between the scRNA-seq and scDNA-seq cells by maximizing the sum of the Pearson correlation coefficients: "Our method, named MaCroDNA, finds the correspondence between the scRNA-seq and scDNA-seq cells by maximizing the sum of the Pearson correlation coefficients"
- [intro] we hypothesize that the Pearson correlation coefficients between the gene expression measurements of scRNA-seq cells and the CNA profiles of scDNA-seq cells could be an effective metric: "we hypothesize that the Pearson correlation coefficients between the gene expression measurements of scRNA-seq cells and the CNA profiles of scDNA-seq cells could be an effective metric for the cell"
- [readme] For mapping RNA cells to DNA cells, the input should be two dataframes `dna_df` and `rna_df`. In these two dataframes, the row ids should be the genes and the column ids should be the cells.: "For mapping RNA cells to DNA cells, the input should be two dataframes `dna_df` and `rna_df`. In these two dataframes, the row ids should be the genes and the column ids "
- [readme] step indicates at which step this result is inferred. And the smaller the step number is, the more confident the result is.: ""step" indicates at which step this result is inferred. And the smaller the step number is, the more confident the result is."
- [readme] Best objective 2.830007718086e+00, best bound 2.830007718086e+00, gap 0.0000%: "Best objective 2.830007718086e+00, best bound 2.830007718086e+00, gap 0.0000%"
- [discussion] unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments: "unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments"
