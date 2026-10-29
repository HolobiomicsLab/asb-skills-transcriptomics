---
name: prevalence-scatter-plot-visualization
description: Use when benchmarking scRNA-to-scDNA clone-assignment methods against
  ground-truth clone labels and you need to compare predicted vs. true clonal prevalence
  concordance across methods.
when_to_use_negative:
- No ground-truth clone labels exist for the dataset (e.g., the BE dataset), so true
  prevalences cannot be paired with predicted ones — use resampling/stability analyses
  instead
- Evaluating cell-to-cell (rather than cell-to-clone) assignments, where clonal prevalence
  is not defined
- Only a single method with no baseline or competitor is being assessed — a scatter
  against the diagonal adds little without comparison context
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_2640
tools:
- name: MaCroDNA
  role: Produces the predicted cell-to-clone assignments whose per-clone prevalence
    fractions form the predicted axis of the scatter plot
  repo: github.com/NakhlehLab/MaCroDNA
- name: clonealign
  role: Benchmark method whose prevalence points are plotted to show its divergence
    from the diagonal (misassignment behavior)
- name: Seurat
  role: Benchmark method whose prevalence points are plotted alongside MaCroDNA and
    clonealign for comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:5ef0531cb799b0aed1edcc9c91ada051b3a85c64eb786dbc9aed71a5327adfc2
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
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# prevalence-scatter-plot-visualization

## Summary

Visualizes cell-to-clone integration quality by plotting true clonal prevalences against clonal prevalences predicted by each integration method (e.g., MaCroDNA, Seurat, clonealign) as per-method scatter points with a diagonal ideal-correlation reference line.

## When to use

Use when benchmarking scRNA-to-scDNA clone-assignment methods against ground-truth clone labels and you need to compare predicted vs. true clonal prevalence concordance across methods.

## When NOT to use

- No ground-truth clone labels exist for the dataset (e.g., the BE dataset), so true prevalences cannot be paired with predicted ones — use resampling/stability analyses instead
- Evaluating cell-to-cell (rather than cell-to-clone) assignments, where clonal prevalence is not defined
- Only a single method with no baseline or competitor is being assessed — a scatter against the diagonal adds little without comparison context

## Inputs

- cell2clone dataframe mapping each RNA cell to its predicted DNA cell and predicted clone id (e.g., MaCroDNA cell2clone output with 'predict_cell' and 'predict_clone' columns)
- ground-truth clone labels (dna_label with 'clone' and 'cell' columns) for computing true clonal prevalences
- cell-to-clone assignments from benchmark methods (clonealign, Seurat) and a random-assignment baseline

## Outputs

- true-vs-predicted clonal prevalence scatter plot with one point series per method and a diagonal reference line
- per-clone predicted prevalence fractions per method (including random baseline)

## How to apply

After running cell-to-clone assignment (e.g., MaCroDNA's cell2clone_assignment() on the CRC scTrio-seq data), compute each clone's predicted prevalence as the fraction of RNA cells assigned to it under each method, plus a random-assignment baseline. Pair each predicted prevalence with the ground-truth prevalence derived from cells with joint RNA and DNA measurements, and plot one scatter series per method with true prevalence on one axis and predicted on the other. Add a diagonal (y=x) reference line: points lying close to the diagonal indicate faithful prevalence recovery, while systematic deviation indicates misassignment. Compare methods by visual proximity to the diagonal — in the CRC evaluation, MaCroDNA's points aligned most closely with the diagonal, while clonealign's points showed the strongest deviation, consistent with clonealign showing the poorest performance among the three methods.

## Related tools

- **MaCroDNA** (Produces the predicted cell-to-clone assignments whose per-clone prevalence fractions form the predicted axis of the scatter plot) — github.com/NakhlehLab/MaCroDNA
- **clonealign** (Benchmark method whose prevalence points are plotted to show its divergence from the diagonal (misassignment behavior))
- **Seurat** (Benchmark method whose prevalence points are plotted alongside MaCroDNA and clonealign for comparison)

## Examples

```
macrodna = MaCroDNA(rna_df, dna_df, dna_label); cell2clone = macrodna.cell2clone_assignment()
```

## Evaluation signals

- MaCroDNA's true-vs-predicted prevalence points lie closest to the diagonal reference line among the compared methods
- clonealign's points show the strongest deviation from the diagonal (misassignment), consistent with it showing the poorest performance among the three methods
- Random-baseline points fall clearly away from the diagonal, confirming the plot discriminates signal from chance assignments
- Predicted prevalences sum to 1 per method (each is the fraction of RNA cells assigned across clones), matching the ground-truth prevalence normalization

## Limitations

- Requires ground-truth clonal prevalences derived from cells with joint RNA and DNA measurements; unavailable for datasets like BE where no cell correspondence ground truth exists
- Visual proximity to the diagonal is a qualitative benchmark; MaCroDNA provides no probabilistic confidence measures for assignments to accompany the points
- Matching is distinctively challenging in homogeneous samples, which can distort prevalence patterns

## Evidence

- [discussion] MaCroDNA outperformed clonealign and Seurat in cell-to-clone assignment and clonal prevalence prediction: "MaCroDNA outperformed clonealign and Seurat in both tasks"
- [discussion] clonealign showed the poorest performance among the three methods: "Among the three methods, clonealign showed the poorest performance"
- [discussion] Two evaluated tasks: cell-to-clone assignment and prevalence prediction: "We evaluated the accuracy of the methods for two inference tasks, namely the cell-to-clone assignment and prediction of the clonal prevalences"
- [readme] README: cell2clone output columns used to compute predicted prevalences: "The other column is "predict_clone", which is the predicted clone id for that RNA cell."
