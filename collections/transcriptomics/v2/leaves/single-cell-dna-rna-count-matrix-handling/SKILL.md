---
name: single-cell-dna-rna-count-matrix-handling
description: Use when paired scDNA-seq copy-number and scRNA-seq expression matrices
  (e.g. the CRC dataset from Bian et al.) must be prepared as gene-by-cell dataframes
  for MaCroDNA cell-association or runtime/memory benchmarking.
when_to_use_negative:
- The RNA and DNA cells are already matched (e.g. paired single-cell multi-omic measurements
  of the same cells) — no cell-association step is needed
- Input is raw sequencing reads or feature tables rather than per-gene read/copy-number
  matrices indexed by genes and cells
- Probabilistic confidence measures on assignments are required — MaCroDNA does not
  provide them, unlike clonealign
edam_operation: http://edamontology.org/operation_2409
tools:
- name: MaCroDNA
  role: Consumes the gene-by-cell dna_df/rna_df dataframes; provides tiny_test() installation
    check, cell2cell_assignment() and cell2clone_assignment()
  repo: github.com/NakhlehLab/MaCroDNA
- name: pandas
  role: Holds the scDNA copy-number and scRNA expression count matrices as dataframes
    with genes as row ids and cells as columns
- name: gurobipy
  role: Mixed-integer optimizer backend invoked by cell2cell_assignment(); must be
    installed manually with an academic (non-commercial) license
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:99946ef602656061543184963613dd4bff52ee7ba6f8aa574068c09934d2b451
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MaCroDNA
  - gurobipy
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-dna-rna-count-matrix-handling

## Summary

Loads and shapes paired single-cell DNA (copy-number) and RNA (gene expression) count data into pandas dataframes with genes as row indices and cells as column ids, in the exact orientation MaCroDNA expects for cell2cell and cell2clone assignment.

## When to use

Use when paired scDNA-seq copy-number and scRNA-seq expression matrices (e.g. the CRC dataset from Bian et al.) must be prepared as gene-by-cell dataframes for MaCroDNA cell-association or runtime/memory benchmarking.

## When NOT to use

- The RNA and DNA cells are already matched (e.g. paired single-cell multi-omic measurements of the same cells) — no cell-association step is needed
- Input is raw sequencing reads or feature tables rather than per-gene read/copy-number matrices indexed by genes and cells
- Probabilistic confidence measures on assignments are required — MaCroDNA does not provide them, unlike clonealign

## Inputs

- scDNA-seq copy-number count matrix (pandas dataframe, genes as rows, cells as columns)
- scRNA-seq gene expression count matrix (pandas dataframe, genes as rows, cells as columns)
- dna_label dataframe with 'clone' and 'cell' columns (only for cell-to-clone mapping)

## Outputs

- dna_df and rna_df gene-by-cell dataframes ready for MaCroDNA(rna_df, dna_df) instantiation
- cell2cell dataframe (index = RNA cell ids, column 'predict_cell') after assignment
- cell2clone dataframe (columns 'predict_cell' and 'predict_clone') for clone-level mapping
- subsetted per-patient matrices at each sampling rate lambda for scaling benchmarks

## How to apply

Load the CRC patient scDNA copy-number and scRNA expression count matrices into two pandas dataframes, dna_df and rna_df, with genes as row ids and cells as column ids; do not pre-intersect the gene sets, since MaCroDNA selects the same genes from the two dataframes itself. For cell-to-clone work, additionally prepare a dna_label dataframe with 'clone' and 'cell' columns mapping each DNA cell to its clone id. Verify package requirements (Python >= 3.7, numpy, pandas, scipy, math, gurobipy) before instantiation; only gurobipy needs manual installation via pip. Sanity-check the data path with the built-in tiny_test(), which runs a 6-gene DNA and 7-gene RNA toy matrix through the Gurobi MIP (binary-integer model, tolerance 1.00e-04) and should end with 'Test Success'. For benchmarking use, subset RNA and DNA cells within each patient at sampling rates lambda in [0.5, 0.6, 0.7, 0.8, 0.9, 1.0] before instantiating MaCroDNA(rna_df, dna_df).

## Related tools

- **MaCroDNA** (Consumes the gene-by-cell dna_df/rna_df dataframes; provides tiny_test() installation check, cell2cell_assignment() and cell2clone_assignment()) — github.com/NakhlehLab/MaCroDNA
- **pandas** (Holds the scDNA copy-number and scRNA expression count matrices as dataframes with genes as row ids and cells as columns)
- **gurobipy** (Mixed-integer optimizer backend invoked by cell2cell_assignment(); must be installed manually with an academic (non-commercial) license)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df); cell2cell, cell2cell_step = macrodna.cell2cell_assignment()
```

## Evaluation signals

- tiny_test() output must show a solved Gurobi model with 'Optimal solution found (tolerance 1.00e-04)', gap 0.0000%, and end with 'Test Success'
- tiny_test reports 'number of genes in dna data 6' and 'number of genes in rna data 6' even though the toy RNA matrix has 7 gene rows — confirming MaCroDNA itself selects the common genes
- Cell- and gene-count log lines ('number of cells in dna data', 'number of cells in rna data') must match the dataframe shapes passed in
- For benchmarking, sampled RNA/DNA cell counts per patient should equal lambda x the original patient cell counts for lambda in [0.5, ..., 1.0]
- cell2cell output must contain a single 'predict_cell' column indexed by RNA cell ids, with a companion 'step' column whose smaller values denote more confident assignments

## Limitations

- Each cell2cell_assignment() call invokes the Gurobi mixed-integer optimizer, whose solve time dominates runtime (measured ~10 min extrapolated for 20,000 CRC cells; 0.18 GB peak memory for 835 cells), and Gurobi requires a license (free academic, non-commercial only, with an expiry date)
- Matching is distinctively challenging in homogeneous samples — a biologically driven failure mode of MaCroDNA
- MaCroDNA does not provide probabilistic confidence measures for assignments (unlike clonealign); the 'step' column in cell2cell_step is the only relative-confidence signal

## Evidence

- [readme] dataframe orientation: "In these two dataframes, the row ids should be the genes and the column ids should be the cells. The genes in RNA data and those in DNA data can be different because MaCroDNA can select the same"
- [readme] dna_label schema: [evidence span withheld — anchor status: not_found]
- [readme] usage API: "> from MaCroDNA import MaCroDNA
> macrodna = MaCroDNA(rna_df, dna_df)
> cell2cell, cell2cell_step = macrodna.cell2cell_assignment()"
- [methods] input format in methods: "For mapping RNA cells to DNA cells, the input should be two dataframes `dna_df` and `rna_df`"
- [methods] install test: "You can test the installation using the following code in Python

`from MaCroDNA import MaCroDNA` ... `test_macrodna.tiny_test()`"
