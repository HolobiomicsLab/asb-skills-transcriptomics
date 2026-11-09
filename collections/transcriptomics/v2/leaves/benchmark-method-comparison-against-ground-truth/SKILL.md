---
name: benchmark-method-comparison-against-ground-truth
description: Use when paired scDNA-seq (CNA) and scRNA-seq (gene expression) data
  from the same cells provide partial ground-truth clone labels, and multiple integration
  methods must be ranked on cell-to-clone assignment accuracy and clonal prevalence
  prediction.
when_to_use_negative:
- No ground truth exists for the cell correspondence (e.g., the BE dataset) — use
  resampling and stability analyses instead of accuracy benchmarking
- The goal is probabilistic confidence per assignment — MaCroDNA, unlike clonealign,
  does not provide probabilistic confidence measures for the assignments
- Input samples are biologically homogeneous, where matching cells is distinctly challenging
  and benchmark results may not generalize
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
tools:
- name: MaCroDNA
  role: Primary method under benchmark; performs cell-to-clone assignment via maximum
    weighted bipartite matching of per-gene read counts (Pearson correlation coefficients)
    using gurobipy
  repo: github.com/NakhlehLab/MaCroDNA
- name: clonealign
  role: Benchmarked baseline (state-of-the-art variational method); showed the poorest
    performance and strongest deviation from ground-truth clonal prevalences
- name: Seurat
  role: Benchmarked baseline representing commonly used single-cell multi-omic integration
    methods
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:dfc17b60721db847154699b5093c3517c752e03fc4f3f8a08f699b2129200655
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MaCroDNA
  - Seurat
  - clonealign
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Benchmark method comparison against ground truth

## Summary

Benchmark scRNA-seq–to–scDNA-seq clone-assignment methods (MaCroDNA, clonealign, Seurat) on the CRC scTrio-seq dataset by comparing predicted cell-to-clone assignments and clonal prevalences against ground-truth labels from jointly profiled cells, using true-vs-predicted prevalence scatter plots with a diagonal reference.

## When to use

Use when paired scDNA-seq (CNA) and scRNA-seq (gene expression) data from the same cells provide partial ground-truth clone labels, and multiple integration methods must be ranked on cell-to-clone assignment accuracy and clonal prevalence prediction.

## When NOT to use

- No ground truth exists for the cell correspondence (e.g., the BE dataset) — use resampling and stability analyses instead of accuracy benchmarking
- The goal is probabilistic confidence per assignment — MaCroDNA, unlike clonealign, does not provide probabilistic confidence measures for the assignments
- Input samples are biologically homogeneous, where matching cells is distinctly challenging and benchmark results may not generalize

## Inputs

- dna_df: scDNA-seq CNA matrix (genes as rows, cells as columns, pandas DataFrame)
- rna_df: scRNA-seq gene-expression matrix (genes as rows, cells as columns, pandas DataFrame)
- dna_label: ground-truth clone labels for DNA cells (DataFrame with columns 'clone' and 'cell')

## Outputs

- cell2clone dataframe per method: predicted DNA cell ('predict_cell') and predicted clone id ('predict_clone') for every RNA cell
- Per-method clonal prevalences (fraction of RNA cells assigned to each clone) and a random-assignment baseline
- True-vs-predicted clonal prevalence scatter plot per method with diagonal reference line

## How to apply

Load the CRC scTrio-seq CNA and gene-expression matrices (dna_df, rna_df) and the ground-truth clone labels (dna_label, with columns 'clone' and 'cell') via pandas, with genes as rows and cells as columns. Run MaCroDNA's cell-to-clone assignment by instantiating MaCroDNA(rna_df, dna_df, dna_label) and calling cell2clone_assignment(), which returns a dataframe with 'predict_cell' and 'predict_clone' columns indexed by RNA cell ids. Run clonealign and Seurat on the same paired data for benchmarking, plus a random-assignment baseline. Compute clonal prevalences as the fraction of RNA cells assigned to each clone under each method and pair each predicted prevalence with its ground-truth prevalence; plot true-vs-predicted prevalence as scatter points per method with a diagonal reference line. Accept the reproduction when MaCroDNA's points lie closest to the diagonal and clonealign's points show the strongest deviation (misassignment), consistent with the reported ranking.

## Related tools

- **MaCroDNA** (Primary method under benchmark; performs cell-to-clone assignment via maximum weighted bipartite matching of per-gene read counts (Pearson correlation coefficients) using gurobipy) — github.com/NakhlehLab/MaCroDNA
- **clonealign** (Benchmarked baseline (state-of-the-art variational method); showed the poorest performance and strongest deviation from ground-truth clonal prevalences)
- **Seurat** (Benchmarked baseline representing commonly used single-cell multi-omic integration methods)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df, dna_label); cell2clone = macrodna.cell2clone_assignment()
```

## Evaluation signals

- MaCroDNA's true-vs-predicted prevalence points lie closest to the diagonal (ideal correlation) across all configurations
- clonealign's points show the strongest deviation from the diagonal (misassignment), ranking last among the three methods
- cell2clone output schema is correct: DataFrame indexed by RNA cell ids with 'predict_cell' and 'predict_clone' columns
- Number of associations in the correspondence matrix matches the number of RNA cells mapped (as in the tiny test where 4.0 associations were found)
- Random-baseline prevalence points fall clearly off the diagonal, confirming the plot discriminates real performance from chance

## Limitations

- Matching cells is distinctively challenging in homogeneous samples — a biologically driven issue that may degrade benchmark validity
- MaCroDNA does not provide probabilistic confidence measures for assignments, unlike statistical methods such as clonealign
- Limited knowledge of the DNA-to-RNA process prevents realistic simulation studies for evaluation; benchmarks must rely on partially known ground truth
- Lack of a detailed description of clonealign's ELBO hindered determining the root cause of its poor behavior

## Evidence

- [readme] For mapping RNA cells to DNA clones, the input needs three dataframes `dna_df`, `rna_df` and `dna_label`.: "For mapping RNA cells to DNA clones, the input needs three dataframes `dna_df`, `rna_df` and `dna_label`."
- [methods] cell2clone assignment invocation: "> cell2clone = macrodna.cell2clone_assignment()"
- [discussion] Evaluation finding: "MaCroDNA outperformed clonealign and Seurat in both tasks"
- [abstract] Ground truth usage in abstract: "Using ground truth information from colorectal cancer data, we demonstrate the advantage of MaCroDNA over existing methods in accuracy and speed."
- [discussion] Benchmark task definition: "We evaluated the accuracy of the methods for two inference tasks, namely the cell-to-clone assignment and prediction of the clonal prevalences"
