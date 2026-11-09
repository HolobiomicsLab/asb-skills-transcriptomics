---
name: benchmark-comparison-across-integration-methods
description: Use when you have paired scDNA-seq copy-number and scRNA-seq expression
  data with partially known clone labels and need to compare integration methods'
  cell-to-clone assignment accuracy.
when_to_use_negative:
- No partially known ground-truth clone labels exist (e.g. the Barrett's esophagus
  dataset has no cell correspondence ground truth, so accuracy benchmarking is impossible;
  use resampling/stability analyses instead)
- You need probabilistic confidence measures for assignments — MaCroDNA does not provide
  these, unlike clonealign
- The sample is biologically homogeneous, where matching cells is distinctively challenging
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3173
tools:
- name: MaCroDNA
  role: Primary method under test; assigns RNA cells to DNA clones via multi-step
    maximum weighted bipartite matching of Pearson correlation coefficients (Gurobi
    MIP solver)
  repo: https://github.com/NakhlehLab/MaCroDNA
- name: clonealign
  role: Baseline comparator; maps RNA cells to DNA clones under its variational (ELBO-based)
    probabilistic model
- name: Seurat
  role: Baseline comparator representing commonly used single-cell multi-omic integration
    approaches
- name: Gurobi
  role: MIP solver used by MaCroDNA for the maximum weighted bipartite matching optimization
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:7f8068db72319d466a681cb174e838e7664c59cfbe4a93871defe3b51acae01d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NakhlehLab/MaCroDNA
  tools:
  - Gurobi
  - MaCroDNA
  - Seurat
  - clonealign
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# benchmark-comparison-across-integration-methods

## Summary

Benchmark cell-to-clone assignment accuracy of MaCroDNA against clonealign and Seurat on the Bian et al. colorectal cancer scTrio-seq dataset, evaluating each method under intNMF and agglomerative clustering with both original and log-transformed copy-number values.

## When to use

Use when you have paired scDNA-seq copy-number and scRNA-seq expression data with partially known clone labels and need to compare integration methods' cell-to-clone assignment accuracy.

## When NOT to use

- No partially known ground-truth clone labels exist (e.g. the Barrett's esophagus dataset has no cell correspondence ground truth, so accuracy benchmarking is impossible; use resampling/stability analyses instead)
- You need probabilistic confidence measures for assignments — MaCroDNA does not provide these, unlike clonealign
- The sample is biologically homogeneous, where matching cells is distinctively challenging

## Inputs

- dna_df: scDNA-seq copy-number dataframe (rows=genes, columns=cells) for patients CRC04, CRC10, CRC11 (Bian et al. scTrio-seq2)
- rna_df: scRNA-seq gene-expression dataframe (rows=genes, columns=cells) for the same patients
- dna_label: clone-label dataframe with 'clone' and 'cell' columns mapping DNA cells to clone ids (used for evaluation and cell-to-clone mapping)

## Outputs

- Per-patient, per-method cell-to-clone assignment accuracy distributions (e.g. cell2clone_accuracy.csv)
- Accuracy comparison across all four clustering/transformation configurations (intNMF vs. agglomerative; original vs. log-transformed copy numbers)

## How to apply

Load the CRC scTrio-seq2 copy-number (dna_df), gene-expression (rna_df), and clone-label (dna_label with 'clone' and 'cell' columns) dataframes for patients CRC04, CRC10, and CRC11 from the Bian et al. dataset. Run MaCroDNA via macrodna.cell2clone_assignment(), which performs multi-step maximum weighted bipartite matching of Pearson correlation coefficients between RNA expression and DNA copy-number profiles using the Gurobi MIP solver; run clonealign under its variational (ELBO-based) probabilistic model; and run Seurat as the representative of common single-cell multi-omic integration approaches. Repeat each method under intNMF and agglomerative clustering of the copy numbers, each with both original and log-transformed copy-number values (four configurations). Compute per-patient cell-to-clone assignment accuracy using the partially known ground-truth joint RNA/DNA clone labels solely for evaluation, and write accuracy distributions (e.g. to cell2clone_accuracy.csv). The benchmark reproduces the ordering in Fig. 2 of the paper: MaCroDNA achieves the highest and most stable accuracy (lowest variance, highest median), clonealign performs worst, and Seurat falls between.

## Related tools

- **MaCroDNA** (Primary method under test; assigns RNA cells to DNA clones via multi-step maximum weighted bipartite matching of Pearson correlation coefficients (Gurobi MIP solver)) — https://github.com/NakhlehLab/MaCroDNA
- **clonealign** (Baseline comparator; maps RNA cells to DNA clones under its variational (ELBO-based) probabilistic model)
- **Seurat** (Baseline comparator representing commonly used single-cell multi-omic integration approaches)
- **Gurobi** (MIP solver used by MaCroDNA for the maximum weighted bipartite matching optimization)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df, dna_label); cell2clone = macrodna.cell2clone_assignment()
```

## Evaluation signals

- Per-patient accuracy ordering matches Fig. 2 of the paper: MaCroDNA outperforms clonealign and Seurat in cell-to-clone assignment on the CRC ground truth
- MaCroDNA shows the lowest variance and highest median accuracy among all methods across all four clustering/transformation configurations
- The README tiny_test runs successfully and reports 'Test Success' after the Gurobi model is solved to optimality (gap 0.0000%)
- clonealign shows the poorest performance among the three methods, as reported in the discussion
- Accuracy is computed only on cells with partially known ground-truth joint RNA/DNA clone labels, never for training

## Limitations

- clonealign's ELBO is not described in sufficient detail to determine the root cause of its poorest performance
- MaCroDNA does not provide probabilistic confidence measures for assignments, unlike statistical methods such as clonealign
- Matching cells is distinctively challenging in homogeneous samples — a biologically driven failure mode
- Limited knowledge of the DNA-to-RNA process prevents realistic simulation-based evaluation

## Evidence

- [abstract] Using ground truth information from colorectal cancer data, we demonstrate the advantage of MaCroDNA over existing methods in accuracy and speed.: "Using ground truth information from colorectal cancer data, we demonstrate the advantage of MaCroDNA over existing methods in accuracy and speed."
- [discussion] MaCroDNA outperformed clonealign and Seurat in cell-to-clone assignment and clonal prevalence prediction: "MaCroDNA outperformed clonealign and Seurat in both tasks"
- [discussion] clonealign showed the poorest performance among the three methods: "Among the three methods, clonealign showed the poorest performance"
- [readme] cell2clone_assignment usage: "> macrodna = MaCroDNA(rna_df, dna_df, dna_label)
> cell2clone = macrodna.cell2clone_assignment()"
- [readme] dna_label schema: "`dna_label` has two columns "clone" and "cell", where "cell" is the DNA cells and "clone" is the corresponding clone id"
