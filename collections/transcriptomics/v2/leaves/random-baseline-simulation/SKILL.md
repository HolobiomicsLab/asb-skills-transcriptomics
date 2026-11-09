---
name: random-baseline-simulation
description: Use when evaluating cell-to-clone assignment or clonal prevalence prediction
  against a ground truth (e.g. scTrio-seq data with joint RNA+DNA measurements) and
  you need a chance-level reference to show a method's points deviate from random
  assignment.
when_to_use_negative:
- No ground-truth clone labels exist (e.g. the BE dataset, where the article notes
  the absence of ground truth about cell correspondence) — resampling/stability analyses
  are used instead
- The task is cell-to-cell assignment confidence rather than clonal prevalence comparison,
  since MaCroDNA, unlike clonealign, does not provide probabilistic confidence measures
  for its assignments
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3173
tools:
- name: MaCroDNA
  role: Produces predicted clone ids per RNA cell via cell2clone_assignment(), whose
    prevalences are compared to ground truth and the random baseline
  repo: github.com/NakhlehLab/MaCroDNA
- name: clonealign
  role: Benchmark method whose clonal prevalences are plotted against ground truth;
    expected to show the strongest deviation from the diagonal (misassignment behavior)
- name: Seurat
  role: Benchmark integration method providing cell-to-clone assignments whose clonal
    prevalences are included in the baseline comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:fc862917229cea8d5241994979b9e20407d148a39ba7d37262f2acd5aa53e26d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MaCroDNA
  - Seurat
  - clonealign
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# random-baseline-simulation

## Summary

Constructs a random cell-to-clone assignment baseline and plots true-vs-predicted clonal prevalences per method (MaCroDNA, clonealign, Seurat) against the diagonal, to judge whether an integration method beats chance on the CRC scTrio-seq benchmark.

## When to use

Use when evaluating cell-to-clone assignment or clonal prevalence prediction against a ground truth (e.g. scTrio-seq data with joint RNA+DNA measurements) and you need a chance-level reference to show a method's points deviate from random assignment.

## When NOT to use

- No ground-truth clone labels exist (e.g. the BE dataset, where the article notes the absence of ground truth about cell correspondence) — resampling/stability analyses are used instead
- The task is cell-to-cell assignment confidence rather than clonal prevalence comparison, since MaCroDNA, unlike clonealign, does not provide probabilistic confidence measures for its assignments

## Inputs

- dna_df: scDNA-seq CNA matrix (genes as rows, cells as columns) from CRC scTrio-seq data
- rna_df: scRNA-seq gene-expression matrix (genes as rows, cells as columns)
- dna_label: dataframe with 'clone' and 'cell' columns giving ground-truth clone ids for DNA cells
- cell-to-clone assignment outputs (e.g. cell2clone dataframe with 'predict_cell' and 'predict_clone' columns) from MaCroDNA, clonealign, and Seurat

## Outputs

- Clonal prevalence table: fraction of RNA cells assigned to each clone per method, per random baseline, and per ground truth
- True-vs-predicted prevalence scatter plot with one point-series per method, a diagonal reference line, and highlighted random-baseline points

## How to apply

After running each integration method (MaCroDNA via MaCroDNA(rna_df, dna_df, dna_label).cell2clone_assignment(), plus clonealign and Seurat) on the CRC scTrio-seq CNA and expression matrices, compute clonal prevalences as the fraction of RNA cells assigned to each clone under each method. Compute the ground-truth prevalence from dna_label for cells with joint RNA and DNA measurements, and generate a random-assignment baseline by assigning RNA cells to clones uniformly at random and computing the same prevalence fractions. Pair each predicted prevalence with its ground-truth prevalence and plot scatter points per method with a diagonal (y=x) reference line plus the random-baseline points; a method whose points lie closest to the diagonal (MaCroDNA) outperforms chance, while points far off the diagonal (clonealign's misassignment behavior) indicate poor performance.

## Related tools

- **MaCroDNA** (Produces predicted clone ids per RNA cell via cell2clone_assignment(), whose prevalences are compared to ground truth and the random baseline) — github.com/NakhlehLab/MaCroDNA
- **clonealign** (Benchmark method whose clonal prevalences are plotted against ground truth; expected to show the strongest deviation from the diagonal (misassignment behavior))
- **Seurat** (Benchmark integration method providing cell-to-clone assignments whose clonal prevalences are included in the baseline comparison)

## Examples

```
macrodna = MaCroDNA(rna_df, dna_df, dna_label); cell2clone = macrodna.cell2clone_assignment()
```

## Evaluation signals

- MaCroDNA's true-vs-predicted prevalence points lie closest to the diagonal across all configurations
- clonealign's prevalence points show the strongest deviation from the diagonal (misassignment), consistent with it showing the poorest performance among the three methods
- Random-baseline points fall visibly farther from the diagonal than MaCroDNA's points, confirming the methods beat chance
- Each predicted prevalence pairs 1:1 with a ground-truth prevalence computed from cells with joint RNA and DNA measurements
- Prevalences under each method and the random baseline each sum to 1 across clones

## Limitations

- The baseline is only interpretable when partial ground truth (joint RNA+DNA scTrio-seq measurements) is available; it cannot be computed for datasets without cell correspondence labels
- Limited knowledge of the DNA-to-RNA process prevents realistic simulation studies, so the random baseline is a coarse chance reference rather than a biologically informed null
- Matching cells is distinctively challenging in homogeneous samples, which can compress prevalence differences and weaken baseline contrast

## Evidence

- [discussion] We evaluated the accuracy of the methods for two inference tasks, namely the cell-to-clone assignment and prediction of the clonal prevalences: "We evaluated the accuracy of the methods for two inference tasks, namely the cell-to-clone assignment and prediction of the clonal prevalences"
- [discussion] clonealign poorest performance: "Among the three methods, clonealign showed the poorest performance"
- [readme] README usage: "> macrodna = MaCroDNA(rna_df, dna_df, dna_label)
> cell2clone = macrodna.cell2clone_assignment()"
- [readme] README output schema: "One is "predict_cell", which is the corresponding DNA cell for that RNA cell.
The other column is "predict_clone", which is the predicted clone id for that RNA cell."
- [abstract] abstract accuracy claim: "Using ground truth information from colorectal cancer data, we demonstrate the advantage of MaCroDNA over existing methods in accuracy and speed."
