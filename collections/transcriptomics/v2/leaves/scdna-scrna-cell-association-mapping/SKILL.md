---
name: scdna-scrna-cell-association-mapping
description: Use when you have paired scDNA-seq copy-number and scRNA-seq gene-expression
  matrices (genes as rows, cells as columns, optionally clone labels per DNA cell)
  and need to assign RNA cells to DNA cells or clones, e.g. for intratumor heterogeneity
  analysis on scTrio-seq data.
when_to_use_negative:
- You need probabilistic confidence measures for assignments — unlike statistical
  methods such as clonealign, MaCroDNA does not provide them; use clonealign instead.
- The sample is biologically homogeneous — the article notes matching cells is distinctively
  challenging in homogeneous samples, a biologically driven issue for MaCroDNA.
- Your goal is integrative clustering of a single modality or you lack per-cell DNA
  copy-number data; MaCroDNA requires both per-gene DNA and RNA read counts for correlation
  matching.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
tools:
- name: MaCroDNA
  role: Performs the cross-domain mapping via multi-step maximum weighted bipartite
    matching of Pearson correlations using Gurobi MIP; provides cell2cell_assignment()
    and cell2clone_assignment()
  repo: github.com/NakhlehLab/MaCroDNA
- name: gurobipy
  role: MIP solver backend that solves the maximum weighted bipartite matching optimization
    inside MaCroDNA
- name: clonealign
  role: Variational (ELBO-based) benchmark method for cell-to-clone assignment used
    for comparison
- name: Seurat
  role: Representative of commonly used single-cell multi-omic integration approaches
    used as a benchmark
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:4ccb3d32c0105fd296c644fa23dca84db98abf0689d76637b0b4b8fd2f7c454f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MaCroDNA
  - Seurat
  - clonealign
  - gurobipy
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scdna-scrna-cell-association-mapping

## Summary

Maps cells measured across single-cell DNA-seq (copy number) and single-cell RNA-seq (gene expression) modalities to a common domain by maximum weighted bipartite matching of Pearson correlation coefficients, solving the 'cell association problem' as implemented in MaCroDNA. It supports both cell-to-cell mapping and cell-to-clone assignment grounded in scDNA clone labels.

## When to use

Use when you have paired scDNA-seq copy-number and scRNA-seq gene-expression matrices (genes as rows, cells as columns, optionally clone labels per DNA cell) and need to assign RNA cells to DNA cells or clones, e.g. for intratumor heterogeneity analysis on scTrio-seq data.

## When NOT to use

- You need probabilistic confidence measures for assignments — unlike statistical methods such as clonealign, MaCroDNA does not provide them; use clonealign instead.
- The sample is biologically homogeneous — the article notes matching cells is distinctively challenging in homogeneous samples, a biologically driven issue for MaCroDNA.
- Your goal is integrative clustering of a single modality or you lack per-cell DNA copy-number data; MaCroDNA requires both per-gene DNA and RNA read counts for correlation matching.

## Inputs

- dna_df: pandas dataframe of scDNA-seq copy-number values, rows=genes, columns=DNA cells
- rna_df: pandas dataframe of scRNA-seq gene-expression values, rows=genes, columns=RNA cells
- dna_label: dataframe with columns 'clone' and 'cell' mapping each DNA cell to its clone id (required only for cell-to-clone mapping)

## Outputs

- cell2cell dataframe indexed by RNA cell ids with column 'predict_cell' (assigned DNA cell), plus cell2cell_step dataframe with 'step' indicating inference step / confidence
- cell2clone dataframe with columns 'predict_cell' and 'predict_clone' assigning each RNA cell to a DNA cell and clone

## How to apply

Instantiate MaCroDNA with pandas dataframes dna_df, rna_df (and dna_label for clone mapping), where row ids are genes and column ids are cells; the tool selects the shared genes between RNA and DNA data itself. For cell-to-cell mapping call macrodna.cell2cell_assignment(), which performs multi-step maximum weighted bipartite matching of Pearson correlation coefficients between RNA expression and DNA copy-number profiles using the Gurobi MIP solver (gurobipy); the 'step' column indicates confidence, with smaller step numbers being more confident. For cell-to-clone mapping supply dna_label with 'clone' and 'cell' columns and call macrodna.cell2clone_assignment() to obtain predict_cell and predict_clone per RNA cell. Restricting the number of scRNA-seq cells assigned to each scDNA-seq cell is hypothesized to be the key to MaCroDNA's accuracy advantage over clonealign and Seurat, so keep this constraint in place rather than relaxing it.

## Related tools

- **MaCroDNA** (Performs the cross-domain mapping via multi-step maximum weighted bipartite matching of Pearson correlations using Gurobi MIP; provides cell2cell_assignment() and cell2clone_assignment()) — github.com/NakhlehLab/MaCroDNA
- **gurobipy** (MIP solver backend that solves the maximum weighted bipartite matching optimization inside MaCroDNA)
- **clonealign** (Variational (ELBO-based) benchmark method for cell-to-clone assignment used for comparison)
- **Seurat** (Representative of commonly used single-cell multi-omic integration approaches used as a benchmark)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df, dna_label); cell2clone = macrodna.cell2clone_assignment()
```

## Evaluation signals

- On the CRC scTrio-seq patients (CRC04, CRC10, CRC11), cell-to-clone assignment accuracy should reproduce the Fig. 2 ordering: MaCroDNA has the highest median accuracy and lowest variance across intNMF and agglomerative clustering with original and log-transformed copy numbers.
- Predicted clonal prevalences (fraction of RNA cells per clone) should lie closest to the true-vs-predicted diagonal relative to clonealign and Seurat; clonealign should show the strongest deviation.
- Installation test: from MaCroDNA import MaCroDNA; test_macrodna.tiny_test() must end with 'Test Success' and an optimal Gurobi solution (IsMIP: 1, gap 0.0000%).
- The correspondence matrix after mapping should contain the expected number of associations (e.g. 4.0 associations for the 4-cell tiny test).
- cell2cell_step should partition every RNA cell with a step number; smaller step values indicate higher-confidence assignments.

## Limitations

- MaCroDNA does not provide probabilistic confidence measures for the assignments, unlike statistical methods such as clonealign.
- Matching cells is distinctively challenging in homogeneous samples — a biologically driven issue for MaCroDNA.
- Limited knowledge of the DNA-to-RNA process prevents realistic simulation-based evaluation of the method.
- The lack of a detailed description of the ELBO for clonealign's variational approximation hindered determining the root cause of clonealign's misassignment behavior.
- Requires Gurobi (academic license, free for non-commercial use) and Python >= 3.7 with numpy, pandas, scipy, math.

## Evidence

- [abstract] we present MaCroDNA, a method that uses maximum weighted bipartite matching of per-gene read counts from single-cell DNA and RNA-seq data.: "we present MaCroDNA, a method that uses maximum weighted bipartite matching of per-gene read counts from single-cell DNA and RNA-seq data."
- [readme] For mapping RNA cells to DNA cells, the input should be two dataframes `dna_df` and `rna_df`.: "For mapping RNA cells to DNA cells, the input should be two dataframes `dna_df` and `rna_df`."
- [readme] For mapping RNA cells to DNA clones, the input needs three dataframes `dna_df`, `rna_df` and `dna_label`.: "For mapping RNA cells to DNA clones, the input needs three dataframes `dna_df`,
`rna_df` and `dna_label`."
- [readme] The second output `cell2cell_step` is also a dataframe. It has two columns "predict_cell" and "step".: "It has two columns "predict_cell" and "step". 
"predict_cell" is the same as that in `cell2cell`.
"step" indicates at which step this result is inferred."
- [discussion] unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments: "unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments"
