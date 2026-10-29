---
name: overkill-simulated-doublet-training-set-construction
description: Use when running Chord with overkill=T on a droplet-based scRNA-seq sample
  (e.g. a Seurat object) and you need the overkilled training data — doublets filtered
  out and simulated doublets generated from remaining cells — before GBM integration
  of method scores.
when_to_use_negative:
- Time-critical analyses — Chord requires integration of multiple methods' results,
  so it is not optimal in terms of time efficiency
- Data where doublets are already experimentally labeled (e.g. cell hashing, MULTI-seq,
  demuxlet), making computational overkill unnecessary
- Existing scRNA-seq datasets where you only want to run one lightweight detector
  rather than retrain an ensemble on simulated doublets
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Chord
  role: 'Implements the overkill step: overkillDB2 removes filtered doublets and simulates
    doublets from remaining cells to build the GBM training set'
  repo: github.com/13308204545/Chord
- name: scds
  role: Built-in method (cxds/bcds) used to evaluate the data and identify doublets
    for removal in the overkill stage
  repo: kostkalab/scds
- name: DoubletFinder
  role: Built-in method used to evaluate the data during overkill and generate initial
    doublet calls before simulation
  repo: chris-mcginnis-ucsf/DoubletFinder
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:5ecf77b93cdfe6de155765e1defc28a53380892de63701a76fb9c15f49938ffe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chord
  - DoubletFinder
  - R
  - adabag
  - gbm
  - remotes
  - scds
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# overkill-simulated-doublet-training-set-construction

## Summary

The Chord 'overkill' step: built-in methods (scds, DoubletFinder) first evaluate the real data, the top fraction of doublet-like cells is removed, and artificial doublets are then simulated from the remaining cells to form the training set for the GBM/AdaBoost ensemble classifier.

## When to use

Use when running Chord with overkill=T on a droplet-based scRNA-seq sample (e.g. a Seurat object) and you need the overkilled training data — doublets filtered out and simulated doublets generated from remaining cells — before GBM integration of method scores.

## When NOT to use

- Time-critical analyses — Chord requires integration of multiple methods' results, so it is not optimal in terms of time efficiency
- Data where doublets are already experimentally labeled (e.g. cell hashing, MULTI-seq, demuxlet), making computational overkill unnecessary
- Existing scRNA-seq datasets where you only want to run one lightweight detector rather than retrain an ensemble on simulated doublets

## Inputs

- Seurat object (seu) for one droplet scRNA-seq sample
- SingleCellExperiment object (sce) produced by creatSCE(seu)
- Estimated doublet rate (doubletrate; ~0.9% per 1,000 cells for 10X data)
- overkillrate — top fraction (0-1) of doublet-like cells to remove across methods' results
- Optional overkilllist — user-supplied vector of cells to remove in overkill
- adddoublt — doublet rate for cells to simulate

## Outputs

- 'overkilled.robj' — dataset with filtered doublets removed, used to generate simulated doublets
- 'seu.robj' and 'sce.robj' — serialized input objects for re-running with added methods
- 'simulated_data.scores.csv' — per-cell scores from built-in methods on simulated doublets
- 'real_data.scores.csv' — per-cell scores on real cells
- 'outname_doublets.csv' — barcodes of predicted doublets (final Chord output after boosting)

## How to apply

Convert the input Seurat object with creatSCE(seu) into a SingleCellExperiment, then run overkillDB2(seu, sce, doubletrate, seed, k, overkill, overkillrate, overkilllist, adddoublt) via chord(..., overkill=T). Set doubletrate using the 10X rule of thumb of ~0.9% per 1,000 cells; set overkillrate to the top fraction (0-1) of doublet-like cells to remove — these are cells called doublet by any of the built-in methods. The rationale is that filtered doublets are removed first so that simulated doublets are generated only from remaining singlet-like cells, yielding a cleaner training set for the GBM (adabag/gbm) classifier. Custom combinations of overkill methods can be supplied via the overkilllist parameter (a vector of cells to remove), and adddoublt controls the doublet rate for simulated cells. The step writes 'overkilled.robj', 'seu.robj' and score tables ('real_data.scores.csv', 'simulated_data.scores.csv') that downstream boosting and add-methods steps consume.

## Related tools

- **Chord** (Implements the overkill step: overkillDB2 removes filtered doublets and simulates doublets from remaining cells to build the GBM training set) — github.com/13308204545/Chord
- **scds** (Built-in method (cxds/bcds) used to evaluate the data and identify doublets for removal in the overkill stage) — kostkalab/scds
- **DoubletFinder** (Built-in method used to evaluate the data during overkill and generate initial doublet calls before simulation) — chris-mcginnis-ucsf/DoubletFinder

## Examples

```
chord(seu="input seurat object",doubletrate="estimated doubletrate",overkill=T,outname="the name you want")
```

## Evaluation signals

- The 'overkilled.robj' dataset exists in the working directory and contains no cells from the overkilllist / top overkillrate fraction
- 'simulated_data.scores.csv' and 'real_data.scores.csv' contain per-cell scores from the built-in methods (scds, DoubletFinder)
- doubletrate was set per the 10X rule of thumb (~0.9% per 1,000 cells) appropriate to the sample's cell count
- After boosting, per-sample doublet rates fall within plausible ranges — e.g. the NSCLC validation gave 3.81–4.74% across samples 11, 13, 17, 18, 22
- Re-running chord with addmethods1/addmethods2 score tables produces refined calls in 'outname2_doublets.csv' consistent with the initial overkill removals

## Limitations

- Chord is not optimal in runtime because it must integrate results of multiple methods; the overkill training-set construction adds to this cost
- The 'overkill' step might improve performance of other methods, but this claim was unverified in the article
- overkillrate must be tuned per dataset — removing the top fraction of doublet-like cells is heuristic and no default is validated for all tissues

## Evidence

- [discussion] Overkill step removes doublets before simulation: "filtered doublets identified with various methods are removed, and doublets simulated based on the remaining cells are used to generate a training set"
- [readme] overkilllist parameter selects the method combination: "You can choose any combination of methods for overkill by setting the overkilllist parameter (a vector of cells to be remove in overkill)"
- [readme] overkillrate defines the top doublet-like fraction removed: "overkillrate  an integer,remove the top ?% doublet-liked cells of any methods' results.(0-1)"
- [methods] Chord invoked with overkill enabled: "chord（seu="input seurat object",doubletrate="estimated doubletrate",overkill=T,outname="the name you want"）"
- [methods] overkillDB2 signature includes doubletrate, overkillrate and adddoublt: "overkillDB2(seu, sce, doubletrate, seed, k, overkill, overkillrate, overkilllist, adddoublt)"
