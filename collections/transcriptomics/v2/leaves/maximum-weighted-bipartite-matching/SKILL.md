---
name: maximum-weighted-bipartite-matching
description: Use when two single-cell modalities (scRNA-seq expression and scDNA-seq
  copy number) must be matched cell-to-cell or cell-to-clone with no ground-truth
  correspondence, i.e., the cell association problem.
when_to_use_negative:
- The sample is biologically homogeneous (e.g., healthy/NDBE BE biopsies), where the
  article shows matching cells is distinctively challenging and assignments show lower
  stability and definiteness
- Probabilistic confidence measures are required, since unlike clonealign, MaCroDNA
  does not provide them
- Ground-truth RNA–DNA correspondence already exists, making optimization unnecessary
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0199
tools:
- name: MaCroDNA
  role: Implements the per-gene Pearson-correlation-weighted bipartite matching in
    src/MaCroDNA/macrodna.py, exposing cell2cell_assignment() and cell2clone_assignment()
  repo: github.com/NakhlehLab/MaCroDNA
- name: Gurobi Optimizer
  role: Solves the assignment as a binary integer program via maximum weighted bipartite
    matching (MIP solved to optimality, tolerance 1.00e-04)
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:8c1683cd6db18acdbcdfbdac60e89bab67b0337fa8aaf9747d77fa498540a8e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gurobi Optimizer
  - MaCroDNA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# maximum-weighted-bipartite-matching

## Summary

Maximum weighted bipartite matching of per-gene read counts is the core optimization used by MaCroDNA to associate scRNA-seq cells with scDNA-seq cells or clones. Edge weights are per-gene Pearson correlation coefficients, and the assignment is solved as a binary integer program with Gurobi.

## When to use

Use when two single-cell modalities (scRNA-seq expression and scDNA-seq copy number) must be matched cell-to-cell or cell-to-clone with no ground-truth correspondence, i.e., the cell association problem.

## When NOT to use

- The sample is biologically homogeneous (e.g., healthy/NDBE BE biopsies), where the article shows matching cells is distinctively challenging and assignments show lower stability and definiteness
- Probabilistic confidence measures are required, since unlike clonealign, MaCroDNA does not provide them
- Ground-truth RNA–DNA correspondence already exists, making optimization unnecessary

## Inputs

- rna_df: pandas dataframe of single-cell RNA-seq gene expression, genes as rows, cells as columns
- dna_df: pandas dataframe of single-cell DNA copy number, genes as rows, cells as columns
- dna_label: pandas dataframe with columns 'clone' and 'cell' linking each DNA cell to its clone id (cell-to-clone mode only)

## Outputs

- cell2cell: dataframe indexed by RNA cell ids with column 'predict_cell' (matched DNA cell)
- cell2cell_step: dataframe with columns 'predict_cell' and 'step' encoding assignment confidence (smaller step = more confident)
- cell2clone: dataframe indexed by RNA cell ids with columns 'predict_cell' and 'predict_clone'

## How to apply

Represent each modality as a pandas dataframe with genes as row indices and cells as column indices (dna_df, rna_df; gene sets may differ — MaCroDNA selects the shared genes itself). Compute per-gene Pearson correlation coefficients between RNA expression and DNA copy-number profiles, motivated by the assumption that gene expression is proportional to copy number, and maximize the sum of these coefficients over RNA-cell–DNA-cell assignments via maximum weighted bipartite matching. Solve the assignment as a binary integer program with Gurobi (MIP solved to optimality, tolerance 1.00e-04), restricting the number of scRNA-seq cells assigned to each scDNA-seq cell — a key design choice behind MaCroDNA's accuracy advantage over clonealign and Seurat. For cell-to-cell mapping call macrodna.cell2cell_assignment(), which also returns cell2cell_step where a smaller 'step' value indicates higher confidence; for cell-to-clone mapping pass a third dataframe dna_label with 'clone' and 'cell' columns and call macrodna.cell2clone_assignment(). Confirm the Gurobi log reports 'Optimal solution found' with a 0.0000% gap and that the number of associations in the correspondence matrix equals the expected RNA-cell count (4.0 in the tiny test).

## Related tools

- **MaCroDNA** (Implements the per-gene Pearson-correlation-weighted bipartite matching in src/MaCroDNA/macrodna.py, exposing cell2cell_assignment() and cell2clone_assignment()) — github.com/NakhlehLab/MaCroDNA
- **Gurobi Optimizer** (Solves the assignment as a binary integer program via maximum weighted bipartite matching (MIP solved to optimality, tolerance 1.00e-04))

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df, dna_label); cell2clone = macrodna.cell2clone_assignment()
```

## Evaluation signals

- Gurobi log reports 'Optimal solution found' with best objective equal to best bound (gap 0.0000%, tolerance 1.00e-04)
- The number of associations in the correspondence matrix equals the expected RNA-cell count (4.0 in the tiny installation test with 4 RNA cells)
- Assignment quality is statistically significant against a null distribution of randomly permuted RNA-to-DNA assignments
- In resampling stability analyses, stability/definiteness is greater in heterogeneous HGD/EAC biopsies than in homogeneous healthy/NDBE biopsies, matching the reported finding
- In the tiny test, matched RNA–DNA pairs show concordant expression–copy-number patterns (e.g., high g5/g6 expression matches the DNA cell with elevated g5/g6 copy number)

## Limitations

- Matching cells is distinctively challenging in biologically homogeneous samples, which can undermine assignment stability
- MaCroDNA does not provide probabilistic confidence measures for assignments (unlike clonealign); only the 'step' field proxies confidence
- Limited knowledge of the DNA-to-RNA process prevents realistic simulation-based evaluation of the matching
- Gurobi requires installation of gurobipy and a license (academic license free for non-commercial use; requires Python >= 3.7, numpy, pandas, scipy, math)

## Evidence

- [abstract] we present MaCroDNA, a method that uses maximum weighted bipartite matching of per-gene read counts from single-cell DNA and RNA-seq data: "we present MaCroDNA, a method that uses maximum weighted bipartite matching of per-gene read counts from single-cell DNA and RNA-seq data"
- [discussion] finds the correspondence between the scRNA-seq and scDNA-seq cells by maximizing the sum of the Pearson correlation coefficients: "finds the correspondence between the scRNA-seq and scDNA-seq cells by maximizing the sum of the Pearson correlation coefficients"
- [readme] "step" indicates at which step this result is inferred. And the smaller the step number is, the more confident the result is: ""step" indicates at which step this result is inferred. And the smaller the step number is, the more confident the result is"
- [readme] The genes in RNA data and those in DNA data can be different because MaCroDNA can select the same genes from the data by itself: "The genes in RNA data and those in DNA data can be different because MaCroDNA can select the same genes from the data by itself"
- [discussion] Matching cells is distinctively challenging in homogeneous samples and highlights a biologically driven issue: "matching cells is distinctively challenging in homogeneous samples and highlights a biologically driven issue that may arise when utilizing MaCroDNA in such cases"
