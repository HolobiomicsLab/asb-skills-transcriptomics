---
name: random-marker-set-sampling
description: Use when benchmarking a marker-selection method (e.g., scGeneFit on CBMC
  scRNA-seq) and a size-matched random-gene baseline is needed to show that reported
  classification error reflects informed marker choice rather than panel size.
when_to_use_negative:
- The goal is to select genuinely informative markers for cell type discrimination
  — a random panel has no optimization rationale and high error by design
- Labels are not categorical cell types (e.g., trajectory time points or spatial locations),
  since the kNN label-classification baseline assumes a flat label partition
- Input is already a reduced feature table or a fixed curated panel — there is no
  gene universe from which to sample a matched-size random set
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGeneFit
  role: Defines the optimized marker panel whose size k the random set must match
    and against which the random baseline's kNN error (30%) is compared (8.05% scGeneFit
    vs 9.90% curated)
  repo: https://github.com/solevillar/scGeneFit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:ec335cafd3498704f1427e5dd2d9249ac22047eaa12d4ab2153d7505b1c95d4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/solevillar/scGeneFit
  tools:
  - scGeneFit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# random-marker-set-sampling

## Summary

Baseline procedure from the scGeneFit paper (Nature Communications, 2021) that samples a marker set of random genes matched in size to the method-selected panel, so that discriminatory power (via k-nearest-neighbor label classification in the reduced marker space) can be compared against jointly optimized and curated marker sets.

## When to use

Use when benchmarking a marker-selection method (e.g., scGeneFit on CBMC scRNA-seq) and a size-matched random-gene baseline is needed to show that reported classification error reflects informed marker choice rather than panel size.

## When NOT to use

- The goal is to select genuinely informative markers for cell type discrimination — a random panel has no optimization rationale and high error by design
- Labels are not categorical cell types (e.g., trajectory time points or spatial locations), since the kNN label-classification baseline assumes a flat label partition
- Input is already a reduced feature table or a fixed curated panel — there is no gene universe from which to sample a matched-size random set

## Inputs

- Normalized scRNA-seq expression matrix (e.g., CBMC CITE-seq from GSE100866, log2(1+X) transformed)
- Vector of cell-type labels for the same cells
- Reference marker set size k (the size of the scGeneFit-selected panel)

## Outputs

- A randomly sampled gene-marker list of exactly k genes
- Held-out kNN label classification error rate for the random panel (expected ~30% on CBMC)
- CSV table row comparing error rates for scGeneFit, curated, and random marker sets

## How to apply

First run the method under test (e.g., scGeneFit on log2(1+X)-normalized CBMC CITE-seq data with cell-type labels) to obtain its selected panel, then record the panel size k. Sample k genes uniformly at random from the same full gene set with a fixed seed, ensuring the count matches the optimized panel exactly. Process the random set through the identical evaluation pipeline as the other panels: split cells 70% training / 30% test, train a k-nearest-neighbor classifier on the 70% training split restricted to the random markers, and evaluate label classification error on the held-out 30%. In the CBMC benchmark the random panel yielded ~30% error versus 8.05% (scGeneFit) and 9.90% (curated), so the random baseline should be the worst performer; write one error entry per marker set to a CSV table.

## Related tools

- **scGeneFit** (Defines the optimized marker panel whose size k the random set must match and against which the random baseline's kNN error (30%) is compared (8.05% scGeneFit vs 9.90% curated)) — https://github.com/solevillar/scGeneFit

## Examples

```
import numpy as np; random_markers = np.random.RandomState(0).choice(gene_names, size=len(scgenefit_markers), replace=False)
```

## Evaluation signals

- The random marker list has exactly the same number of genes as the scGeneFit-selected panel
- The random-set classification error is the highest of the three sets, matching the reported ordering (8.05% scGeneFit < 9.90% curated < 30% random on CBMC)
- The error is computed on the held-out 30% test split with the same kNN classifier settings used for the other marker sets
- The output CSV contains exactly one error entry per marker set (scGeneFit, curated, random)

## Limitations

- The random baseline presumes the linear-separability assumption shared by marker-selection methods holds; the paper notes this 'is a reasonable assumption in high dimensions, but may not' hold in lower dimensions
- Random sampling gives no robustness against redundant or uninformative genes, so results are noisy and seed-dependent; a single random draw is a weak statistical baseline
- Only applicable to categorical labeling tasks — the paper notes no support yet for placing unlabeled cells on trajectories or spatial assays

## Evidence

- [results] scGeneFit achieved 8.05% label classification error versus 9.90% for the curated marker set, and substantially outperformed a random marker set, which had 30% error.: [evidence span withheld — anchor status: not_found]
- [other] Random set matched in size to scGeneFit panel, evaluated identically.: [evidence span withheld — anchor status: not_found]
- [abstract] Joint optimization rationale for marker selection.: "Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification"
- [discussion] Linear separability caveat.: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
