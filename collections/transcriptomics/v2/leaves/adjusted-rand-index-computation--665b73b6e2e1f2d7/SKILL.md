---
name: adjusted-rand-index-computation
description: Use when evaluating a cell annotation/clustering result against known
  ground-truth labels on scRNA-seq data, especially when comparing methods in Monte
  Carlo splits and needing a chance-adjusted agreement metric beyond raw accuracy.
when_to_use_negative:
- The new sample contains unknown/rare cell types not present in the training labels
  — the ground-truth label set differs from the prediction set and NeuCA/exhaustive
  assignment, and by extension this label-agreement evaluation, is not designed for
  it
- Only one clustering/labeling result exists with no reference labels, so no pairwise
  label agreement against truth can be computed
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NeuCA
  role: Produces the predicted cell labels whose quality is scored by ARI and accurately
    assigned rate in the benchmark
  repo: https://bioconductor.org/packages/NeuCA/
- name: scmap
  role: Competing supervised label-assignment method benchmarked with ARI on identical
    splits (scmap-cluster was the best existing method)
- name: CHETAH
  role: Competing supervised method benchmarked with ARI; known to produce unassigned/intermediate
    labels that ARI penalizes
- name: Seurat
  role: Competing unsupervised clustering method evaluated with ARI on the same PBMC
    splits
- name: SC3
  role: Competing unsupervised clustering method evaluated with ARI on the same PBMC
    splits
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:43b56137edb67bc336a795bd206f986e1b431ad7e34268e8edafc2d9adf3aa12
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - NeuCA
  - R
  - SC3
  - Seurat
  - UniverSC
  - clues
  - scmap
  - stats
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# adjusted-rand-index-computation

## Summary

Compute the Adjusted Rand Index (ARI) to compare predicted single-cell labels against known reference labels, used alongside the accurately assigned rate to benchmark cell annotation methods such as NeuCA against Seurat, SC3, scmap, and CHETAH across Monte Carlo simulation splits.

## When to use

Use when evaluating a cell annotation/clustering result against known ground-truth labels on scRNA-seq data, especially when comparing methods in Monte Carlo splits and needing a chance-adjusted agreement metric beyond raw accuracy.

## When NOT to use

- The new sample contains unknown/rare cell types not present in the training labels — the ground-truth label set differs from the prediction set and NeuCA/exhaustive assignment, and by extension this label-agreement evaluation, is not designed for it
- Only one clustering/labeling result exists with no reference labels, so no pairwise label agreement against truth can be computed

## Inputs

- Vector of predicted cell type labels for each testing cell (e.g., predicted.label from NeuCA)
- Vector of true/reference cell type labels for the same cells (e.g., Seg_true_cell_label)
- Benchmark results per split from methods run on identical Monte Carlo train/test splits

## Outputs

- ARI value (range approximately -1 to 1, chance-adjusted to ~0) per method per split
- Aggregated ARI comparisons across the 160 Monte Carlo simulations, suitable for benchmark_results.csv

## How to apply

After each method (e.g., NeuCA at small/medium/big model sizes, scmap-cluster, scmap-cell, CHETAH, Seurat, SC3) predicts labels for a test split (e.g., 800, 1600, or 4000 testing cells drawn from the FACS-sorted 10X PBMC data), pair the predicted labels with the true cell type labels. ARI is computed as a chance-adjusted pairwise-agreement measure, making it robust to label permutation and differing numbers of predicted classes, which matters when methods emit unassigned/intermediate labels (as CHETAH can). Compute ARI for every method on every split, together with the accurately assigned rate, then aggregate across the 160 Monte Carlo simulations (training fractions p = 10%, 20%, 50%, 80%) to rank methods. The article's benchmark checks that NeuCA-med exceeds scmap-cluster by about 10 percentage points in accurately assigned rate while also ranking top in ARI, so ARI should be reported jointly with accuracy rather than alone.

## Related tools

- **NeuCA** (Produces the predicted cell labels whose quality is scored by ARI and accurately assigned rate in the benchmark) — https://bioconductor.org/packages/NeuCA/
- **scmap** (Competing supervised label-assignment method benchmarked with ARI on identical splits (scmap-cluster was the best existing method))
- **CHETAH** (Competing supervised method benchmarked with ARI; known to produce unassigned/intermediate labels that ARI penalizes)
- **Seurat** (Competing unsupervised clustering method evaluated with ARI on the same PBMC splits)
- **SC3** (Competing unsupervised clustering method evaluated with ARI on the same PBMC splits)

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE); table(predicted.label, Seg_true_cell_label)
```

## Evaluation signals

- ARI equals exactly 1.0 when predicted labels match true labels up to permutation (e.g., in a perfect confusion matrix like the README's Baron/Seg example)
- ARI is near 0 when agreement is at chance level; negative values indicate worse-than-chance agreement
- ARI is invariant to relabeling/permutation of predicted class names, so a sanity check is that permuting predicted.label does not change the ARI
- On this benchmark, aggregated ARI for NeuCA-med, NeuCA-med's accuracy margin over scmap-cluster (~10 percentage points) and NeuCA-small (>=5 points) should be consistent — all three NeuCA versions rank top in both accurately assigned rate and ARI
- Cross-check ARI against the confusion matrix computed via table(predicted.label, Seg_true_cell_label): off-diagonal counts should predict reduced ARI

## Limitations

- CHETAH can produce an unignorable amount of unassigned/intermediate cell labels even when no unknown types are expected, which depresses label-agreement metrics and must be handled consistently across methods
- Benchmark conclusions are specific to settings where all cell types in the new sample exist in the reference data; ARI against a reference label set presumes an exhaustive label universe
- The paper reports ARI alongside accurately assigned rate; neither metric alone captures both absolute accuracy and chance-adjusted agreement

## Evidence

- [abstract] NeuCA can achieve more accurate cell annotation results compared with existing methods: "NeuCA can achieve more accurate cell annotation results compared with existing methods"
- [methods] Compute the accurately assigned rate and the Adjusted Rand Index (ARI) for each method on every split.: [evidence span withheld — anchor status: not_found]
- [readme] Evaluate prediction with a confusion matrix using true labels: "If you have the true cell type labels for the testing dataset, you may evaluate the predictive performance by a confusion matrix"
- [intro] CHETAH unassigned labels complicate label-agreement evaluation: "CHETAH still produces an unignorable amount of unassigned/intermediate cell labels"
