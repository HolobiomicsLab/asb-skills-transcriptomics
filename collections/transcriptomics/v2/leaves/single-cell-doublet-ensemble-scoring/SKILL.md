---
name: single-cell-doublet-ensemble-scoring
description: Use when droplet-based scRNA-seq data may contain heterotypic doublets
  that destabilize downstream clustering or trajectory inference, and single-method
  doublet scores vary across datasets — an ensemble (GBM-boosted) score is needed
  before filtering.
when_to_use_negative:
- The dataset was already doublet-identified experimentally (cell hashing, MULTI-seq,
  demuxlet) and only inter-sample doublets need labelling — those techniques label
  doublets directly without computational scoring
- Ambient mRNA contamination is the dominant artifact rather than doublets — Chord
  does not incorporate ambient RNA removal steps and other pre-processing would be
  needed
- Input is not single-cell droplet data (e.g. a bulk or aggregated feature table)
  — simulating doublets from remaining cells is meaningless
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Chord
  role: 'Ensemble driver: integrates per-method doublet scores via GBM/AdaBoost boosting,
    applies the ''overkill'' step, and emits outname_doublets.csv'
  repo: github.com/13308204545/Chord
- name: scds
  role: Built-in scorer providing cxds and bcds doublet scores (cxds.ntop=500, bcds.ntop=500)
    used inside the ensemble
  repo: github.com/kostkalab/scds
- name: DoubletFinder
  role: Built-in scorer generating artificial doublets and pANN-based scores (dbf.pN=0.25,
    dbf.pK tuned per dataset, dbf.PCs=1:10)
  repo: github.com/chris-mcginnis-ucsf/DoubletFinder
- name: gbm
  role: Gradient boosting machine used to integrate the results of different doublet
    detection methods
- name: adabag
  role: AdaBoost alternative boosting backend (mfinal=40 iterations) selectable via
    method="adaboost"
- name: Scrublet
  role: External method whose doublet scores can be added to the ensemble via addmethods1/addmethods2;
    also a comparison baseline in the trajectory evaluation
  repo: github.com/AllonKleinLab/scrublet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:f573d587001b36e2b2fdf504dac4dc60600e15d6f231907bc520e431e58b0391
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chord
  - DoubletFinder
  - R
  - Scrublet
  - adabag
  - doubletCells
  - gbm
  - scds
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-doublet-ensemble-scoring

## Summary

Scores every cell in a droplet scRNA-seq dataset for doublets by integrating multiple detection methods (scds cxds/bcds, DoubletFinder, optionally Scrublet) with a GBM/AdaBoost ensemble, then removes the top-ranked cells (matching the estimated doublet rate, e.g. 20%) to produce a clean single-cell matrix.

## When to use

Use when droplet-based scRNA-seq data may contain heterotypic doublets that destabilize downstream clustering or trajectory inference, and single-method doublet scores vary across datasets — an ensemble (GBM-boosted) score is needed before filtering.

## When NOT to use

- The dataset was already doublet-identified experimentally (cell hashing, MULTI-seq, demuxlet) and only inter-sample doublets need labelling — those techniques label doublets directly without computational scoring
- Ambient mRNA contamination is the dominant artifact rather than doublets — Chord does not incorporate ambient RNA removal steps and other pre-processing would be needed
- Input is not single-cell droplet data (e.g. a bulk or aggregated feature table) — simulating doublets from remaining cells is meaningless

## Inputs

- Raw (unnormalized) droplet scRNA-seq UMI counts matrix with cells as rows and genes as columns (Seurat object 'seu' and/or SingleCellExperiment 'sce')
- Estimated doublet rate (e.g. ~0.9% per 1,000 cells for 10X chemistry, or a known labelled doublet proportion such as 20% in DataPSE)

## Outputs

- Per-cell ensemble doublet scores (simulated_data.scores.csv / real_data.scores.csv)
- Doublet barcodes file 'outname_doublets.csv' listing cells to remove
- Filtered single-cell dataset (removed top 20% of cells by doublet score) suitable for clustering, differential expression, and trajectory re-inference with Slingshot or monocle

## How to apply

Load the cell-by-gene counts as a Seurat object (seu) and SingleCellExperiment (sce), and estimate the doublet rate from cell count (~0.9% per 1,000 cells for 10X; use the known proportion, e.g. 20% labelled doublets in the DataPSE synthetic dataset, when available). Run chord() with overkill=T: built-in methods (cxds, bcds from scds; DoubletFinder with dbf.pN=0.25 and dataset-tuned dbf.pK) first score the data, filtered doublets are removed, and doublets simulated from the remaining cells generate the training set; the gbm (or adaboost) algorithm then integrates all method scores into one ensemble score. Optionally boost additional methods such as Scrublet by scoring 'overkilled.robj' and 'seu.robj' and appending scores to simulated_data.scores.csv before re-running chord with addmethods1/addmethods2. Retrieve doublet barcodes from outname_doublets.csv and remove the top-ranked cells — for method comparison at equal stringency, remove the top 20% of cells by each method's score. Validate by confirming Chord- and Scrublet-filtered data support Slingshot/monocle bifurcating trajectories matching clean reference data without spurious extra branches.

## Related tools

- **Chord** (Ensemble driver: integrates per-method doublet scores via GBM/AdaBoost boosting, applies the 'overkill' step, and emits outname_doublets.csv) — github.com/13308204545/Chord
- **scds** (Built-in scorer providing cxds and bcds doublet scores (cxds.ntop=500, bcds.ntop=500) used inside the ensemble) — github.com/kostkalab/scds
- **DoubletFinder** (Built-in scorer generating artificial doublets and pANN-based scores (dbf.pN=0.25, dbf.pK tuned per dataset, dbf.PCs=1:10)) — github.com/chris-mcginnis-ucsf/DoubletFinder
- **gbm** (Gradient boosting machine used to integrate the results of different doublet detection methods)
- **adabag** (AdaBoost alternative boosting backend (mfinal=40 iterations) selectable via method="adaboost")
- **Scrublet** (External method whose doublet scores can be added to the ensemble via addmethods1/addmethods2; also a comparison baseline in the trajectory evaluation) — github.com/AllonKleinLab/scrublet

## Examples

```
chord(seu=seu, doubletrate=0.2, overkill=T, outname="datapse")  # then read outname_doublets.csv and remove the top 20% scored cells
```

## Evaluation signals

- Filtered data (top 20% scored cells removed) re-clustered with Slingshot/monocle reproduces the clean-data bifurcating trajectory with no extra spurious branches (as achieved by Chord and Scrublet on DataPSE)
- 'outname_doublets.csv' contains barcodes consistent with the estimated doubletrate (e.g. ~20% of 600 cells), not an arbitrary count
- Simulated doublet scores are recorded in 'simulated_data.scores.csv' for both 'overkilled.robj' and 'seu.robj' before re-running chord with addmethods1/addmethods2
- Ensemble calls are more stable than any single constituent method (cxds, bcds, DoubletFinder) across datasets — bcds-, cxds- and doubletCells-only filtering introduced spurious Slingshot/monocle branches while the ensemble did not

## Limitations

- Chord is not optimal in time efficiency because it must integrate the results of multiple methods; refactoring the integrated methods was proposed but not implemented
- Performance of constituent computational doublet detection methods varies greatly across datasets; no single method is a gold standard for every scenario
- The claim that the 'overkill' step might also improve the performance of other methods is unverified in the paper
- Other pre-processing such as ambient mRNA removal for droplets is not incorporated, though the paper notes it should be considered for downstream accuracy

## Evidence

- [abstract] Chord integrates multiple doublet detection methods using a machine learning algorithm: "We here propose Chord which implements a machine learning algorithm that integrates multiple doublet detection methods to address these issues."
- [discussion] GBM integration of method results: "To solve this problem, Chord integrates the results of different methods through the GBM algorithm."
- [discussion] Overkill training-set construction: "First, built-in methods are used to evaluate the data. Then, filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells"
- [readme] Output doublet barcodes file: "The doublets' barcodes are in the file "outname_doublets.csv""
- [intro] Heterotypic doublets impact trajectories: "heterotypic doublets are considered to have more impact on downstream analyses, including dimensionality reduction, cell clustering"
- [readme] doubletrate estimation guidance: "doubletrate = ~0.9% per 1,000 cells"
