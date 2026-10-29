---
name: artificial-doublet-simulation-from-cell-pairs
description: Use when building the real-artificial training set inside the Chord workflow
  after the overkill filter, i.e. when a prefiltered Seurat object of quality singlets
  must be converted into simulated doublet profiles to train the GBM/adaboost doublet
  classifier.
when_to_use_negative:
- When the input data has not passed the overkill filter — the simulation pool should
  consist of quality singlets after built-in-method doublets are removed
- When experimental doublet labelling is available (cell hashing, MULTI-seq, demuxlet)
  — the article notes these experimentally label doublets directly and no simulated
  training set is needed
- When the goal is to boost additional method scores on the existing training set
  rather than regenerate it — in that case evaluate overkilled.robj and add scores
  to simulated_data.scores.csv instead
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_6170
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Chord
  role: 'Hosts the artificial doublet simulation: overkillDB2 samples in-cluster cell
    pairs, averages N(1,0.1)-weighted expression profiles, and writes overkilled.robj'
  repo: github.com/13308204545/Chord
- name: DoubletFinder
  role: Installed dependency whose artificial-doublet pN convention (default 0.25)
    informs the proportion of simulated doublets in the merged training set
  repo: github.com/chris-mcginnis-ucsf/DoubletFinder
- name: scds
  role: Installed dependency providing cxds/bcds parameters used alongside the simulation
    in the Chord pipeline
  repo: github.com/kostkalab/scds
- name: Seurat
  role: R framework supplying NormalizeData, FindVariableFeatures, ScaleData, and
    the PCA embedding on which k-means clustering runs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:d10e391e2a1bca02862e603ba9f96fe6314462cdf3020be0d03c5fe0ce7411e1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chord
  - DoubletFinder
  - R
  - Seurat
  - scds
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_6170
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# artificial-doublet-simulation-from-cell-pairs

## Summary

Generates a labelled training set for Chord's GBM ensemble by simulating artificial doublets: after the overkill step removes likely doublets, remaining cells are k-means-clustered (k=20) on PC1-30 embeddings and in-cluster cell pairs are averaged with N(1,0.1) weighting to create artificial doublet profiles at the estimated doublet rate.

## When to use

Use when building the real-artificial training set inside the Chord workflow after the overkill filter, i.e. when a prefiltered Seurat object of quality singlets must be converted into simulated doublet profiles to train the GBM/adaboost doublet classifier.

## When NOT to use

- When the input data has not passed the overkill filter — the simulation pool should consist of quality singlets after built-in-method doublets are removed
- When experimental doublet labelling is available (cell hashing, MULTI-seq, demuxlet) — the article notes these experimentally label doublets directly and no simulated training set is needed
- When the goal is to boost additional method scores on the existing training set rather than regenerate it — in that case evaluate overkilled.robj and add scores to simulated_data.scores.csv instead

## Inputs

- Prefiltered (overkilled) Seurat object of quality singlet cells
- SingleCellExperiment object converted from the Seurat object (via creatSCE)
- Estimated doublet rate (doubletrate; ~0.9% per 1,000 cells for 10X as a guideline)
- Random seed; k-means parameter k (default k=20)
- Optional adddoublt parameter (doublet rate of cells to be simulated)

## Outputs

- overkilled.robj containing the merged labelled real-artificial training set
- Artificial doublet profiles appended to the real cells with 'doublet' labels
- Exportable list of doublet barcodes (CSV)

## How to apply

Take the overkilled Seurat object (likely doublets already removed by the built-in overkill methods) and normalise it with NormalizeData, identify highly variable genes with FindVariableFeatures, and scale with ScaleData. Run PCA retaining PC1-30, then partition cells into k=20 clusters by k-means on the PCA embedding. Within each cluster, sample cell pairs at a ratio of doubletrate/(1-doubletrate) relative to the real cells (e.g. pN default 0.25 of the merged data, per DoubletFinder convention), weighting each member's expression by an independent N(1,0.1) biological factor, and average the weighted profiles to make artificial doublets labelled as doublets. Append these to the remaining real cells to form the merged training set, saved via overkillDB2(seu, sce, doubletrate, seed, k, overkill, overkillrate, overkilllist, adddoublt) to overkilled.robj. The rationale is that the GBM ensemble needs labelled artificial doublets drawn from the cleaned, cluster-matched cell pool to learn doublet signatures without contamination from the coarse-filtered cells.

## Related tools

- **Chord** (Hosts the artificial doublet simulation: overkillDB2 samples in-cluster cell pairs, averages N(1,0.1)-weighted expression profiles, and writes overkilled.robj) — github.com/13308204545/Chord
- **DoubletFinder** (Installed dependency whose artificial-doublet pN convention (default 0.25) informs the proportion of simulated doublets in the merged training set) — github.com/chris-mcginnis-ucsf/DoubletFinder
- **scds** (Installed dependency providing cxds/bcds parameters used alongside the simulation in the Chord pipeline) — github.com/kostkalab/scds
- **Seurat** (R framework supplying NormalizeData, FindVariableFeatures, ScaleData, and the PCA embedding on which k-means clustering runs)

## Examples

```
R: overkillDB2(seu, sce, doubletrate, seed, k, overkill=TRUE, overkillrate, overkilllist, adddoublt)  # saves merged real-artificial training set to overkilled.robj
```

## Evaluation signals

- The merged real-artificial object contains artificial doublets at the proportion set by doubletrate (analogous to DoubletFinder pN, default 0.25 of the merged data)
- overkilled.robj exists and contains a labelled training set with real cells plus labelled artificial doublets
- Doublet barcodes can be exported to CSV from the merged object
- PCA retains PC1-30 and k-means partitions into exactly k=20 clusters before pair sampling
- Each artificial doublet is the average of two profiles each weighted by an independent N(1,0.1) factor

## Limitations

- Chord is not optimal in runtime because it must integrate results of multiple methods; refactoring was proposed but not implemented
- The doubletrate must be estimated by the user (~0.9% per 1,000 cells for 10X), and simulation quality depends on this estimate
- Other pre-processing, e.g. ambient mRNA removal for droplets, is not incorporated and may be needed for downstream accuracy
- The simulated doublets are heterotypic/homotypic pair averages from k=20 clusters; cluster choice (k) and seed affect the training set composition

## Evidence

- [discussion] Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells are used to generate a training set.: "Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells are used to generate a training set."
- [full_text] Within each cluster, sample cell pairs at a ratio of doubletrate/(1-doubletrate)... weighting each member's expression profile with an independent N(1,0.1) biological weighting factor.: [evidence span withheld — anchor status: not_found]
- [methods] overkillDB2(seu, sce, doubletrate, seed, k, overkill, overkillrate, overkilllist, adddoublt): "overkillDB2(seu, sce, doubletrate, seed, k, overkill, overkillrate, overkilllist, adddoublt)"
- [readme] The number of generated artificial doublets, expressed as a proportion of the merged real-artificial data. Default is set to 0.25: "The number of generated artificial doublets, expressed as a proportion of the merged real-artificial data. Default is set to 0.25"
- [readme] doubletrate = ~0.9% per 1,000 cells: "A:It depends on the number of cells in the sample. 10X can be referred：doubletrate = ~0.9% per 1,000 cells."
