---
name: cell-to-clone-assignment
description: Use when you have matched scDNA-seq copy-number data, scRNA-seq gene-expression
  data, and a clone-label table mapping each DNA cell to a clone id, and you need
  to predict which clone each RNA cell belongs to.
when_to_use_negative:
- You only have DNA and RNA data without clone labels and want cell-to-cell rather
  than cell-to-clone mapping — use MaCroDNA's cell2cell_assignment() instead
- The sample is biologically homogeneous (e.g., healthy or low-grade tissue), where
  matching cells is distinctively challenging and assignments are unreliable
- You require probabilistic confidence measures on assignments — MaCroDNA does not
  provide them, unlike variational methods such as clonealign
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3520
tools:
- name: MaCroDNA
  role: Performs the cell-to-clone assignment by maximizing summed per-gene Pearson
    correlation coefficients between scRNA-seq expression and scDNA-seq copy number
    via weighted bipartite matching
  repo: github.com/NakhlehLab/MaCroDNA
- name: Gurobi Optimizer
  role: Solves the binary integer program representing the maximum weighted bipartite
    matching to optimality (tolerance 1.00e-04)
- name: clonealign
  role: Baseline variational (ELBO-based) method for comparison of cell-to-clone assignment
    accuracy
- name: Seurat
  role: Baseline representing common single-cell multi-omic integration approaches
    for accuracy comparison
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:65e184b334125eb1f40bcd78a9a3d16b9478a7dbb2ea2b33bd8b841243bfdf2d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gurobi Optimizer
  - MaCroDNA
  - Seurat
  - clonealign
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cell-to-clone-assignment

## Summary

Assign scRNA-seq cells to scDNA-seq clones by running MaCroDNA's cell2clone_assignment(), which maximizes the sum of per-gene Pearson correlation coefficients between RNA expression and DNA copy-number profiles via a Gurobi-solved weighted bipartite matching.

## When to use

Use when you have matched scDNA-seq copy-number data, scRNA-seq gene-expression data, and a clone-label table mapping each DNA cell to a clone id, and you need to predict which clone each RNA cell belongs to.

## When NOT to use

- You only have DNA and RNA data without clone labels and want cell-to-cell rather than cell-to-clone mapping — use MaCroDNA's cell2cell_assignment() instead
- The sample is biologically homogeneous (e.g., healthy or low-grade tissue), where matching cells is distinctively challenging and assignments are unreliable
- You require probabilistic confidence measures on assignments — MaCroDNA does not provide them, unlike variational methods such as clonealign

## Inputs

- dna_df: pandas dataframe of single-cell DNA copy-number values, rows=genes, columns=DNA cells
- rna_df: pandas dataframe of single-cell RNA gene-expression values, rows=genes, columns=RNA cells (gene sets may differ from dna_df)
- dna_label: two-column dataframe ('clone', 'cell') linking each DNA cell to its clone id

## Outputs

- cell2clone: dataframe indexed by RNA cell ids with 'predict_cell' (assigned DNA cell) and 'predict_clone' (predicted clone id) columns, e.g. saved as cell2clone_assignments.csv
- Gurobi MIP solver log reporting optimal solution, objective value, and optimality gap

## How to apply

Load dna_df and rna_df as pandas dataframes with genes as row indices and cells as columns (gene sets may differ between modalities; MaCroDNA intersects them itself), plus dna_label with 'clone' and 'cell' columns. Instantiate the model with all three dataframes and call cell2clone_assignment(): it internally builds a weighted bipartite graph of per-gene Pearson correlation coefficients and solves a maximum weighted bipartite matching as a binary integer program with Gurobi (MIP solved to optimality, tolerance 1.00e-04). The returned cell2clone dataframe is indexed by RNA cell ids with 'predict_cell' (matched DNA cell) and 'predict_clone' (predicted clone id) columns; save it, e.g., as cell2clone_assignments.csv. Restricting the number of scRNA-seq cells assigned to each scDNA-seq cell is hypothesized to be the key reason MaCroDNA outperforms clonealign and Seurat on CRC scTrio-seq ground truth.

## Related tools

- **MaCroDNA** (Performs the cell-to-clone assignment by maximizing summed per-gene Pearson correlation coefficients between scRNA-seq expression and scDNA-seq copy number via weighted bipartite matching) — github.com/NakhlehLab/MaCroDNA
- **Gurobi Optimizer** (Solves the binary integer program representing the maximum weighted bipartite matching to optimality (tolerance 1.00e-04))
- **clonealign** (Baseline variational (ELBO-based) method for comparison of cell-to-clone assignment accuracy)
- **Seurat** (Baseline representing common single-cell multi-omic integration approaches for accuracy comparison)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df, dna_label); cell2clone = macrodna.cell2clone_assignment()
```

## Evaluation signals

- Gurobi log reports 'Optimal solution found' with a zero optimality gap (e.g., 'best bound ... gap 0.0000%')
- The number of associations in the correspondence matrix equals the expected count of assigned RNA cells (in the README installation test, 4 RNA cells yield 4.0 associations)
- On CRC patients (CRC04, CRC10, CRC11) with partially known ground truth, per-patient assignment accuracy reproduces the Fig. 2 ordering: MaCroDNA has the highest median and lowest variance accuracy versus clonealign and Seurat across the four clustering (intNMF, agglomerative) / transformation (original, log-transformed) configurations
- Output schema check: cell2clone is indexed by RNA cell ids and contains exactly the 'predict_cell' and 'predict_clone' columns
- On BE biopsies, downstream K* phylogenetic-signal analysis recovers hundreds of genes with K* > 1 in HGD and EAC biopsies (e.g., ERBB2 among top significant COSMIC genes), far fewer in lower-grade/healthy tissue

## Limitations

- MaCroDNA does not provide probabilistic confidence measures for the assignments, unlike statistical methods such as clonealign
- Matching cells is distinctively challenging in homogeneous samples, a biologically driven issue that can degrade performance
- Performance evaluation relies on partially known ground truth; the BE dataset lacks ground-truth cell correspondence, so only resampling/stability analyses are possible
- Limited knowledge of the DNA-to-RNA process prevents realistic simulation-based evaluation
- Requires a Gurobi license (free academic license) and gurobipy installed before use

## Evidence

- [readme] cell2clone_assignment with Gurobi MIP: [evidence span withheld — anchor status: not_found]
- [readme] Optimal MIP solution: "Optimal solution found (tolerance 1.00e-04) Best objective 2.830007718086e+00, best bound 2.830007718086e+00, gap 0.0000%"
- [abstract] Method core: "we present MaCroDNA, a method that uses maximum weighted bipartite matching of per-gene read counts from single-cell DNA and RNA-seq data."
- [discussion] Performance vs baselines: "MaCroDNA outperformed clonealign and Seurat in both tasks"
- [discussion] Mechanistic hypothesis: "We hypothesize that the key to the better performance of MaCroDNA is restricting the number of the scRNA-seq cells being assigned to each scDNA-seq cell"
- [discussion] No confidence measures: "unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments"
