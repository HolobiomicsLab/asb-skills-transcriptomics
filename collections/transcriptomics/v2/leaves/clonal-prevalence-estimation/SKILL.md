---
name: clonal-prevalence-estimation
description: Use when paired scDNA-seq CNA and scRNA-seq expression matrices with
  ground-truth DNA clone labels are available and the goal is to assess whether a
  cell-to-clone integration method's predicted clonal prevalences correlate with ground
  truth.
when_to_use_negative:
- No ground-truth clone labels exist for the DNA cells (e.g. the BE dataset in this
  paper lacked correspondence ground truth) — use resampling/stability analyses instead
  of prevalence-correlation validation.
- The task is cell-to-cell mapping rather than cell-to-clone assignment — use MaCroDNA's
  cell2cell_assignment() instead, which does not require dna_label.
- Probabilistic confidence measures per assignment are required — MaCroDNA, unlike
  clonealign, does not provide probabilistic confidence measures for its assignments.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3173
tools:
- name: MaCroDNA
  role: 'Primary method: performs cell-to-clone assignment via maximum weighted bipartite
    matching of per-gene read counts; its cell2clone_assignment() output yields predicted
    clone ids used to compute clonal prevalences'
  repo: github.com/NakhlehLab/MaCroDNA
- name: clonealign
  role: Benchmark method run on the same paired data for comparison of predicted clonal
    prevalences; showed the poorest performance among the three methods
- name: Seurat
  role: Benchmark method representing commonly used single-cell multi-omic integration,
    run on the same CRC data for prevalence-prediction comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:c2ce02041f380d8321e6ae5f0a2d1b1eba61c1a7ce1a1769d68938170b54e8e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MaCroDNA
  - Seurat
  - clonealign
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# clonal-prevalence-estimation

## Summary

Benchmark skill for evaluating how well scRNA-seq-to-scDNA-seq clone assignment methods (MaCroDNA, clonealign, Seurat) recover ground-truth clonal prevalences, by comparing predicted versus true per-clone cell fractions on paired scTrio-seq data such as the CRC dataset.

## When to use

Use when paired scDNA-seq CNA and scRNA-seq expression matrices with ground-truth DNA clone labels are available and the goal is to assess whether a cell-to-clone integration method's predicted clonal prevalences correlate with ground truth.

## When NOT to use

- No ground-truth clone labels exist for the DNA cells (e.g. the BE dataset in this paper lacked correspondence ground truth) — use resampling/stability analyses instead of prevalence-correlation validation.
- The task is cell-to-cell mapping rather than cell-to-clone assignment — use MaCroDNA's cell2cell_assignment() instead, which does not require dna_label.
- Probabilistic confidence measures per assignment are required — MaCroDNA, unlike clonealign, does not provide probabilistic confidence measures for its assignments.

## Inputs

- dna_df: pandas DataFrame of scDNA-seq CNA values with genes as rows and DNA cells as columns
- rna_df: pandas DataFrame of scRNA-seq gene expression values with genes as rows and RNA cells as columns
- dna_label: two-column DataFrame ('clone', 'cell') giving the ground-truth clone id for each DNA cell

## Outputs

- cell2clone DataFrame indexed by RNA cell id with columns 'predict_cell' (matched DNA cell) and 'predict_clone' (predicted clone id) per method
- Per-clone predicted prevalences (fraction of RNA cells per clone) for each method and for a random-assignment baseline
- True-vs-predicted clonal prevalence scatter plot per method with a diagonal reference line

## How to apply

Load the CRC scTrio-seq CNA matrix (dna_df), gene-expression matrix (rna_df), and ground-truth clone labels (dna_label) with pandas, with genes as rows and cells as columns; dna_label is a two-column frame ('clone' and 'cell'). Instantiate MaCroDNA(rna_df, dna_df, dna_label) and call cell2clone_assignment() to obtain a 'predict_cell' and 'predict_clone' id for every RNA cell, then run clonealign and Seurat on the same data for benchmarking, plus a random-assignment baseline. Compute clonal prevalences as the fraction of RNA cells assigned to each clone under each method and pair each predicted prevalence with its ground-truth prevalence. Plot true-vs-predicted prevalence as scatter points per method with a diagonal reference line; the method whose points lie closest to the diagonal best recovers ground-truth clonal composition. The paper's validation criterion is that MaCroDNA's prevalence points align most closely with the diagonal while clonealign's points deviate most strongly (misassignment).

## Related tools

- **MaCroDNA** (Primary method: performs cell-to-clone assignment via maximum weighted bipartite matching of per-gene read counts; its cell2clone_assignment() output yields predicted clone ids used to compute clonal prevalences) — github.com/NakhlehLab/MaCroDNA
- **clonealign** (Benchmark method run on the same paired data for comparison of predicted clonal prevalences; showed the poorest performance among the three methods)
- **Seurat** (Benchmark method representing commonly used single-cell multi-omic integration, run on the same CRC data for prevalence-prediction comparison)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df, dna_label); cell2clone = macrodna.cell2clone_assignment()
```

## Evaluation signals

- MaCroDNA's true-vs-predicted prevalence scatter points lie closest to the diagonal (ideal correlation) across all configurations.
- clonealign's prevalence points show the strongest deviation from the diagonal, consistent with its reported misassignment behavior.
- Predicted prevalences for each method are computed as fractions of RNA cells per clone, so per-method prevalence vectors sum to 1 across clones.
- A random-assignment baseline should produce points scattered well away from the diagonal, providing a lower-bound sanity check.
- MaCroDNA's cell2clone output must contain both 'predict_cell' and 'predict_clone' columns with every RNA cell id in the index.

## Limitations

- MaCroDNA does not provide probabilistic confidence measures for the assignments, unlike statistical methods such as clonealign.
- Matching cells is distinctively challenging in homogeneous samples — a biologically driven issue when clone prevalence estimation relies on MaCroDNA assignments.
- Restricting the number of scRNA-seq cells assigned to each scDNA-seq cell is hypothesized to be key to MaCroDNA's better performance; relaxing this constraint may degrade prevalence estimates.
- Lack of a detailed description of the ELBO for clonealign's variational approximation hindered determining the root cause of its poor behavior.

## Evidence

- [discussion] MaCroDNA outperformed clonealign and Seurat in cell-to-clone assignment and clonal prevalence prediction: "MaCroDNA outperformed clonealign and Seurat in both tasks"
- [discussion] Evaluation of clonal prevalence prediction: "We evaluated the accuracy of the methods for two inference tasks, namely the cell-to-clone assignment and prediction of the clonal prevalences"
- [discussion] clonealign poorest performance: "Among the three methods, clonealign showed the poorest performance"
- [discussion] CRC benchmark dataset: "and Seurat19—as the representative of the commonly used methods in the single-cell multi-omic studies—through their application to a CRC dataset31"
- [readme] cell2clone_assignment API: "> macrodna = MaCroDNA(rna_df, dna_df, dna_label)
> cell2clone = macrodna.cell2clone_assignment()"
- [readme] cell2clone output schema: "It has two columns. One is "predict_cell", which is the corresponding DNA cell for that RNA cell.
The other column is "predict_clone", which is the predicted clone id for that RNA cell."
- [discussion] No probabilistic confidence measures: "unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments"
