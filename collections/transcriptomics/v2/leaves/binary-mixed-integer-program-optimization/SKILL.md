---
name: binary-mixed-integer-program-optimization
description: Use when cells measured only by scRNA-seq (expression) must be associated
  one-to-one with cells measured only by scDNA-seq (CNA), and the matching can be
  cast as a binary MIP over Pearson-correlation edge weights when |RNA cells| ≤ |DNA
  cells|.
when_to_use_negative:
- When probabilistic confidence measures for assignments are required — MaCroDNA,
  unlike statistical methods such as clonealign, provides none (only the step number
  as a coarse confidence proxy)
- When the sample is biologically homogeneous — the article notes matching cells is
  distinctively challenging in homogeneous samples
- When pairing RNA cells to DNA clones rather than individual cells — use the cell2clone_assignment
  path with an additional dna_label DataFrame ('clone' and 'cell' columns) instead
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: MaCroDNA
  role: Provides the cell2cell_assignment routine that builds the Pearson-correlation
    bipartite graph, formulates the MIP, and returns cell2cell and cell2cell_step;
    located at src/MaCroDNA/macrodna.py
  repo: github.com/NakhlehLab/MaCroDNA
- name: Gurobi (gurobipy)
  role: Solves the weighted maximum bipartite matching as a binary mixed-integer program
    (binary variables, ~1e-04 tolerance); academic license required and must be installed
    manually before MaCroDNA
- name: pandas
  role: Carries the gene-by-cell input DataFrames (dna_df, rna_df) and the returned
    correspondence DataFrames
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:37b2c1c3c3620dad80af6f6d8cb5644cde452f8b16f39b1f44f03ebab7d7204d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gurobi (gurobipy)
  - MaCroDNA
  - pandas
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# binary-mixed-integer-program-optimization

## Summary

Formulates MaCroDNA's maximum weighted bipartite matching between scRNA-seq cells (gene expression) and scDNA-seq cells (copy-number profiles) as a binary mixed-integer program, edge-weighted by per-gene Pearson correlation coefficients, and solves it with Gurobi to recover the one-to-one cell correspondence.

## When to use

Use when cells measured only by scRNA-seq (expression) must be associated one-to-one with cells measured only by scDNA-seq (CNA), and the matching can be cast as a binary MIP over Pearson-correlation edge weights when |RNA cells| ≤ |DNA cells|.

## When NOT to use

- When probabilistic confidence measures for assignments are required — MaCroDNA, unlike statistical methods such as clonealign, provides none (only the step number as a coarse confidence proxy)
- When the sample is biologically homogeneous — the article notes matching cells is distinctively challenging in homogeneous samples
- When pairing RNA cells to DNA clones rather than individual cells — use the cell2clone_assignment path with an additional dna_label DataFrame ('clone' and 'cell' columns) instead

## Inputs

- dna_df: pandas DataFrame of scDNA-seq CNA values, genes as rows, DNA cells as columns
- rna_df: pandas DataFrame of scRNA-seq expression counts, genes as rows, RNA cells as columns (may contain extra genes; MaCroDNA selects the shared gene set)
- Pearson correlation weight matrix between each scRNA-seq cell and each scDNA-seq cell

## Outputs

- cell2cell: DataFrame indexed by RNA cell ids with a single 'predict_cell' column giving the assigned DNA cell
- cell2cell_step: DataFrame with 'predict_cell' and 'step' columns recording at which sequential step each RNA cell was assigned (smaller step = more confident)
- binary one-to-one correspondence matrix with the number of associations equal to the smaller modality's cell count
- Gurobi optimal objective value and optimality gap

## How to apply

Load dna_df and rna_df with genes as rows and cells as columns, restricting both to the shared gene set (MaCroDNA selects common genes itself, so differing gene lists are acceptable). Compute per-gene Pearson correlation coefficients between each RNA cell's expression profile and each DNA cell's CNA profile to weight the bipartite edges. When the number of RNA cells is at most the number of DNA cells, encode the maximum weighted bipartite matching as a binary MIP (e.g. a model of 9 rows, 16 binary columns on the 4-cell test) and solve it with Gurobi (version 9.5.1, default MIPFocus: 0, 1.00e-04 optimality tolerance), reading out the one-to-one binary correspondence matrix. When RNA cells outnumber DNA cells, switch to the iterative constraint-relaxed heuristic, solving a sequence of matching problems and recording the step at which each RNA cell is assigned — a smaller step number means a more confident assignment. Rationale: maximizing summed Pearson correlations rests on the assumption that gene expression is proportional to copy number, which makes correlation the effective cell-association metric.

## Related tools

- **MaCroDNA** (Provides the cell2cell_assignment routine that builds the Pearson-correlation bipartite graph, formulates the MIP, and returns cell2cell and cell2cell_step; located at src/MaCroDNA/macrodna.py) — github.com/NakhlehLab/MaCroDNA
- **Gurobi (gurobipy)** (Solves the weighted maximum bipartite matching as a binary mixed-integer program (binary variables, ~1e-04 tolerance); academic license required and must be installed manually before MaCroDNA)
- **pandas** (Carries the gene-by-cell input DataFrames (dna_df, rna_df) and the returned correspondence DataFrames)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df); cell2cell, cell2cell_step = macrodna.cell2cell_assignment()
```

## Evaluation signals

- Run the built-in tiny_test() on the 4-cell synthetic DNA/RNA matrices: it must report 'Test Success' with 4 associations in the correspondence matrix
- The Gurobi solve must reach best objective 2.830007718086e+00, best bound 2.830007718086e+00, gap 0.0000% (tolerance 1.00e-04)
- The correspondence matrix must contain exactly min(|RNA cells|, |DNA cells|) associations (4.0 in the test case)
- Each RNA cell must map to exactly one DNA cell and no DNA cell may receive more than one RNA cell when |RNA| ≤ |DNA| (one-to-one bipartite constraint)
- On a CRC patient sample (Bian et al.), assignments must match the reference implementation's output

## Limitations

- MaCroDNA does not provide probabilistic confidence measures for the assignments; confidence is only proxied by the step number of the sequential heuristic
- Matching cells is distinctively challenging in biologically homogeneous samples
- When RNA cells outnumber DNA cells, the assignment is a heuristic decomposition of sequential MIPs, so later-step assignments are less confident
- Limited knowledge of the DNA-to-RNA process prevents realistic simulation studies for evaluation
- gurobipy must be installed manually (academic license is free but expires)

## Evidence

- [intro] We hypothesize that the Pearson correlation coefficients between the gene expression measurements of scRNA-seq cells and the CNA profiles of scDNA-seq cells: "we hypothesize that the Pearson correlation coefficients between the gene expression measurements of scRNA-seq cells and the CNA profiles of scDNA-seq cells could be an effective metric for the cell"
- [discussion] MaCroDNA finds correspondence by maximizing summed Pearson correlations: "Our method, named MaCroDNA, finds the correspondence between the scRNA-seq and scDNA-seq cells by maximizing the sum of the Pearson correlation coefficients"
- [readme] Tiny test expected objective and gap: "Best objective 2.830007718086e+00, best bound 2.830007718086e+00, gap 0.0000%"
- [readme] Step number encodes confidence: ""step" indicates at which step this result is inferred. And the smaller the step number is, the more confident the result is."
- [discussion] No probabilistic confidence: "unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments"
- [readme] Shared-gene selection: "The genes in RNA data and those in DNA data can be different because MaCroDNA can select the same genes
 from the data by itself."
