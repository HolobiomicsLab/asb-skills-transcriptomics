---
name: geometric-versus-spherical-sketch-benchmarking
description: Use when you need to compare cell-subsampling (sketching) strategies
  on scRNA-seq data by quantifying how faithfully a sketch of 1%-10% of cells represents
  the full transcriptomic space or preserves cell-type separability.
when_to_use_negative:
- The input is not a cell-by-feature matrix (e.g., raw FASTQ or unnormalized counts)
  — Sphetcher expects features such as genes, transcripts, or principal components
  with rows as samples
- The goal is only to obtain one sketch without method comparison — running Sphetcher
  alone suffices and the benchmarking overhead (baseline method, 10 trials, distance
  computation) is unnecessary
- The downstream algorithm's assumptions are incompatible with Sphetcher's model —
  the article notes the model 'might be less compatible with assumptions made by other
  algorithms' beyond Louvain and Monocle 2
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Sphetcher
  role: Computes the spherical sketch via thresholding, selecting representative cells
    within spheres of a fixed size; run with Pearson correlation as distance metric
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Clones the Sphetcher source repository for compilation
  repo: https://github.com/canzarlab/Sphetcher
- name: Louvain
  role: Community detection on a k-nearest-neighbor graph built from each sketch to
    assess cell-type separation (ARI) of sketched cells
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:641e6a8e17d753def25ade529b40290eb5b2c018d392a8a23f95185bba11bb41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Louvain
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# geometric-versus-spherical-sketch-benchmarking

## Summary

Benchmark Sphetcher's spherical sketching (spheres via a thresholding technique) against Hie et al.'s geometric sketching (boxes) on single-cell transcriptomic datasets by measuring robust Hausdorff distances to the full data and downstream Louvain clustering quality (ARI).

## When to use

Use when you need to compare cell-subsampling (sketching) strategies on scRNA-seq data by quantifying how faithfully a sketch of 1%-10% of cells represents the full transcriptomic space or preserves cell-type separability.

## When NOT to use

- The input is not a cell-by-feature matrix (e.g., raw FASTQ or unnormalized counts) — Sphetcher expects features such as genes, transcripts, or principal components with rows as samples
- The goal is only to obtain one sketch without method comparison — running Sphetcher alone suffices and the benchmarking overhead (baseline method, 10 trials, distance computation) is unnecessary
- The downstream algorithm's assumptions are incompatible with Sphetcher's model — the article notes the model 'might be less compatible with assumptions made by other algorithms' beyond Louvain and Monocle 2

## Inputs

- Expression matrix CSV (rows = cells, columns = features/genes/PCs) for each dataset, e.g. muraro, klein, zeisel, chen, zeiselCNS, saunders
- Sketch size specification (1%, 4%, 10% of cells)
- Ground-truth cell-type annotations for ARI computation
- Geometric sketch indicator vectors from Hie et al. method as baseline

## Outputs

- sketch_indicator_output.csv: indicator vector of n samples with sketch_size 1's
- hausdorff_results.csv: robust Hausdorff distances per dataset, sketch size, and method over 10 trials
- ari_results.csv: ARI values per dataset, sketching method, and sketch size
- Comparison plots reproducing Figures 2/S1 (Hausdorff distances) and Figure 3/S4 (grouped ARI bar chart)

## How to apply

Clone and compile Sphetcher from github.com/canzarlab/Sphetcher with a C++11 compiler (cd src; make). Prepare each dataset's expression matrix as CSV with rows as cells and columns as features/PCs, then run 'sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv' at 1%, 4%, and 10% sketch sizes, using Pearson correlation as the distance metric as in the paper's experiments. Run Hie et al. geometric sketching on the same matrices at the same sizes as the baseline, repeating both procedures over 10 random trials per dataset and size. For each trial compute the robust Hausdorff distance between sketch and full dataset to test the paper's finding that the 1% spherical sketch even outperforms the 10% geometric sketch. For the downstream axis, extract sketched cells from the indicator vectors, build a k-nearest-neighbor graph per sketch, run Louvain community detection, propagate labels to all cells with a kNN classifier, and compute the Adjusted Rand Index against ground-truth cell-type annotations; aggregate results per dataset, size, and method into comparison tables and figures.

## Related tools

- **Sphetcher** (Computes the spherical sketch via thresholding, selecting representative cells within spheres of a fixed size; run with Pearson correlation as distance metric) — https://github.com/canzarlab/Sphetcher
- **git** (Clones the Sphetcher source repository for compilation) — https://github.com/canzarlab/Sphetcher
- **Louvain** (Community detection on a k-nearest-neighbor graph built from each sketch to assess cell-type separation (ARI) of sketched cells)

## Examples

```
git clone https://github.com/canzarlab/Sphetcher && cd src && make && sphetcher zeisel_pca.csv 1000 sketch_indicator_output.csv
```

## Evaluation signals

- Each sketch_indicator_output.csv contains exactly sketch_size 1's in a length-n vector
- Robust Hausdorff distances of spherical sketches should be consistently smaller than geometric sketches across all six datasets and sizes 1%-10%, and the 1% spherical sketch should outperform the 10% geometric sketch
- Louvain clustering of spherical sketches should yield better or comparable median ARI than geometric sketches, with geometric sketching winning in at most ~3 of 36 dataset/size instances
- Relative ARI trends across sketch sizes and methods should match the paper's Figure 3/S4
- Results should be stable over 10 random trials per dataset and sketch size

## Limitations

- The interplay of sketching distance metric with internal preprocessing routines of downstream scRNA-seq algorithms was not systematically addressed in the study
- No software version numbers are stated for Sphetcher or the downstream tools used
- Fairness-aware sketching (prior class labels with l_min representatives per class) is only demonstrated at proof-of-principle level

## Evidence

- [other] Sphetcher's sphere-based sampling consistently produces sketches with smaller Hausdorff distances to the full dataset than geometric sketching: [evidence span withheld — anchor status: not_found]
- [other] Louvain clustering ARI finding: [evidence span withheld — anchor status: not_found]
- [discussion] Sphetcher uses Pearson correlation: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric"
- [readme] Compilation and invocation: "sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv"
- [readme] Output format spec: "an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)"
