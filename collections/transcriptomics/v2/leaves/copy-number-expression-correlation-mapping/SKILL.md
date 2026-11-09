---
name: copy-number-expression-correlation-mapping
description: Use when you have paired single-cell DNA copy-number and RNA-seq count
  matrices (genes x cells) with no direct cell correspondence and need cell-to-cell
  or cell-to-clone assignment, e.g. to link genomic mutations to transcriptomic states.
when_to_use_negative:
- Samples are biologically homogeneous — the article notes 'matching cells is distinctively
  challenging in homogeneous samples'
- You need probabilistic confidence measures for assignments — 'unlike statistical
  methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures'
- Ground-truth correspondence between cells exists and direct pairing suffices; no
  cross-modality matching is needed
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: MaCroDNA
  role: Performs the correlation-based maximum weighted bipartite matching between
    scRNA-seq and scDNA-seq cells; source at src/MaCroDNA/macrodna.py
  repo: https://github.com/NakhlehLab/MaCroDNA
- name: Gurobi Optimizer
  role: Solves the binary integer program implementing the maximum weighted bipartite
    matching (MIP to optimality, tolerance 1.00e-04); requires gurobipy and an academic
    license
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:2c6bdd7b1d06035691e9fcc57a128756a1eba5f5906c47d82fad0cced2d659f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NakhlehLab/MaCroDNA
  tools:
  - Gurobi Optimizer
  - MaCroDNA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# copy-number-expression-correlation-mapping

## Summary

Map scRNA-seq cells to scDNA-seq cells or clones by maximizing summed per-gene Pearson correlation coefficients between RNA expression and DNA copy-number profiles via maximum weighted bipartite matching solved as a binary integer program with Gurobi. Used by MaCroDNA to resolve the cell association problem and downstream K* phylogenetic signal analyses in Barrett's esophagus progression.

## When to use

Use when you have paired single-cell DNA copy-number and RNA-seq count matrices (genes x cells) with no direct cell correspondence and need cell-to-cell or cell-to-clone assignment, e.g. to link genomic mutations to transcriptomic states.

## When NOT to use

- Samples are biologically homogeneous — the article notes 'matching cells is distinctively challenging in homogeneous samples'
- You need probabilistic confidence measures for assignments — 'unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures'
- Ground-truth correspondence between cells exists and direct pairing suffices; no cross-modality matching is needed

## Inputs

- dna_df: pandas dataframe of scDNA-seq copy-number values, rows = genes, columns = cells
- rna_df: pandas dataframe of scRNA-seq gene expression counts, rows = genes, columns = cells
- dna_label (clone mapping only): two-column dataframe 'clone' and 'cell' linking each DNA cell to its clone id

## Outputs

- cell2clone dataframe indexed by RNA cell ids with columns 'predict_cell' and 'predict_clone' (e.g. saved as cell2clone_assignments.csv)
- cell2cell and cell2cell_step dataframes (cell-to-cell mode) with 'predict_cell' and 'step' (lower step = more confident assignment)
- per-gene K* phylogenetic signal values for downstream BE progression analyses

## How to apply

Load dna_df and rna_df as pandas dataframes with genes as row indices and cells as column indices; gene sets may differ because MaCroDNA selects common genes itself. For clone-level mapping, also load dna_label with columns 'clone' and 'cell'. Instantiate MaCroDNA(rna_df, dna_df, dna_label) and run cell2clone_assignment(), which maximizes the sum of per-gene Pearson correlation coefficients via maximum weighted bipartite matching solved as a binary integer program in Gurobi (MIP to optimality, tolerance 1.00e-04). The returned cell2clone dataframe is indexed by RNA cell ids with 'predict_cell' (matched DNA cell) and 'predict_clone' (predicted clone id) columns; save as cell2clone_assignments.csv. Rationale: expression is assumed proportional to copy number, so per-gene Pearson correlation between modalities is an effective matching metric; for K* analyses, use the assignments to compute per-gene K* phylogenetic signal and identify genes with K* > 1 in HGD/EAC biopsies (e.g. ERBB2).

## Related tools

- **MaCroDNA** (Performs the correlation-based maximum weighted bipartite matching between scRNA-seq and scDNA-seq cells; source at src/MaCroDNA/macrodna.py) — https://github.com/NakhlehLab/MaCroDNA
- **Gurobi Optimizer** (Solves the binary integer program implementing the maximum weighted bipartite matching (MIP to optimality, tolerance 1.00e-04); requires gurobipy and an academic license)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df, dna_label); cell2clone = macrodna.cell2clone_assignment()
```

## Evaluation signals

- Underlying Gurobi MIP reports 'Optimal solution found (tolerance 1.00e-04)' with a 0.0000% optimality gap
- Number of associations in the correspondence matrix equals the expected count of RNA cells assigned (e.g. 4.0 for the 4-cell tiny test)
- cell2clone output is indexed by RNA cell ids and contains exactly the columns 'predict_cell' and 'predict_clone'
- tiny_test() installation check completes with 'Test Success' and exit code 0
- For BE analyses, significant genes show inferred K* > 1 (e.g. ERBB2 the highest-K* significant COSMIC gene in HGD/EAC biopsies, far fewer such genes in lower-grade/healthy tissue)

## Limitations

- No ground truth about cell correspondence exists for the BE dataset; validation relied on resampling and stability analyses
- MaCroDNA does not provide probabilistic confidence measures for the assignments (unlike clonealign)
- Matching is distinctly challenging in homogeneous samples — a biologically driven failure mode
- Knowledge of the DNA-to-RNA process is too limited to produce realistic simulation studies for evaluation

## Evidence

- [abstract] maximum weighted bipartite matching of per-gene read counts: "For this purpose, we present MaCroDNA, a method that uses maximum weighted bipartite matching of per-gene read counts from single-cell DNA and RNA-seq data."
- [readme] cell2clone assignment API: "macrodna = MaCroDNA(rna_df, dna_df, dna_label)
> cell2clone = macrodna.cell2clone_assignment()"
- [discussion] correlation objective: "Our method, named MaCroDNA, finds the correspondence between the scRNA-seq and scDNA-seq cells by maximizing the sum of the Pearson correlation coefficients"
- [readme] Gurobi MIP optimality: "Optimal solution found (tolerance 1.00e-04)
Best objective 2.830007718086e+00, best bound 2.830007718086e+00, gap 0.0000%"
- [readme] output schema: "One is "predict_cell", which is the corresponding DNA cell for that RNA cell. The other column is "predict_clone", which is the predicted clone id for that RNA cell."
