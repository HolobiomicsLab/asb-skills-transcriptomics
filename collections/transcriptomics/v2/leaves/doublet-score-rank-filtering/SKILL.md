---
name: doublet-score-rank-filtering
description: Use when a droplet-based scRNA-seq dataset has an estimated doublet proportion
  and doublet scores from one or more tools (Chord, Scrublet, bcds, cxds, doubletCells),
  and you need to remove doublets by score cutoff and assess the effect on trajectory/branch
  inference.
when_to_use_negative:
- The data already carry experimental doublet labels (cell hashing, MULTI-seq, demuxlet)
  that only mark cross-sample doublets and cannot annotate within-sample ones — score-rank
  filtering is still needed, but do not treat demuxlet labels as the score being filtered
- The dataset is a merged multi-sample collection where cell-type proportions are
  not representative of any single sample — Scrublet's README warns it 'may perform
  poorly on merged datasets' and should be run per sample
edam_operation: http://edamontology.org/operation_3695
tools:
- name: Chord
  role: Ensemble doublet scorer (GBM/AdaBoost over built-in methods) whose ranked
    doublet calls are filtered and optionally refined with the overkill mechanism
  repo: github.com/13308204545/Chord
- name: Scrublet
  role: Python doublet scorer using simulated doublets and a k-NN classifier; its
    continuous score is ranked and thresholded
  repo: github.com/AllonKleinLab/scrublet
- name: scds
  role: Dependency providing the cxds and bcds base scores used by Chord and compared
    in the rank-filtering benchmark
  repo: github.com/kostkalab/scds
- name: DoubletFinder
  role: Base method whose artificial-doublet pANN scores feed the Chord ensemble and
    are ranked for filtering
  repo: github.com/chris-mcginnis-ucsf/DoubletFinder
- name: Slingshot
  role: Reconstructs cluster-based MST lineage/pseudotime trajectories on filtered
    data to validate doublet removal
- name: monocle
  role: Reconstructs reversed-graph-embedding trajectories on filtered data as a second
    validation view
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:df10c6629f8e9d2b3400438f5eb52642fb619a6ff0dba48a2e83e70720d84cd7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chord
  - DoubletFinder
  - R
  - Scrublet
  - Slingshot
  - adabag
  - doubletCells
  - gbm
  - monocle
  - scds
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# doublet-score-rank-filtering

## Summary

Rank cells by a per-method doublet score and remove the top-scoring fraction (matched to the expected doublet rate, e.g. the top 20% when 20% of cells are labelled doublets) so all detection methods can be compared at equal stringency, then verify the filtering by re-inferring downstream trajectories (Slingshot/monocle) against clean-data branch structure.

## When to use

Use when a droplet-based scRNA-seq dataset has an estimated doublet proportion and doublet scores from one or more tools (Chord, Scrublet, bcds, cxds, doubletCells), and you need to remove doublets by score cutoff and assess the effect on trajectory/branch inference.

## When NOT to use

- The data already carry experimental doublet labels (cell hashing, MULTI-seq, demuxlet) that only mark cross-sample doublets and cannot annotate within-sample ones — score-rank filtering is still needed, but do not treat demuxlet labels as the score being filtered
- The dataset is a merged multi-sample collection where cell-type proportions are not representative of any single sample — Scrublet's README warns it 'may perform poorly on merged datasets' and should be run per sample

## Inputs

- Raw or normalized scRNA-seq count matrix (cells x genes), e.g. a Seurat object or SingleCellExperiment for R methods
- Per-cell doublet scores from each method (Chord, Scrublet, bcds, cxds, doubletCells)
- Estimated or known doublet rate (e.g. ~0.9% per 1,000 cells for 10X, or 0.20 for the DataPSE synthetic dataset)

## Outputs

- Filtered cell/country barcode list (removed top-ranked cells), e.g. outname_doublets.csv from Chord
- Filtered expression matrix with the top 20% scored cells removed
- Reconstructed Slingshot/monocle trajectories and branch-structure comparison against the clean reference (trajectory_comparison.png)

## How to apply

Compute a doublet score for every cell with each method being compared (e.g. Chord via its GBM ensemble, Scrublet via simulated-doublet KNN scoring, plus bcds, cxds and doubletCells). Rank cells by score and remove the top fraction equal to the known or estimated doublet rate — in the DataPSE benchmark with 600 cells and 20% labelled doublets, this meant removing the top 20% for every method so comparisons are made at equal stringency. Retrieve the barcodes to drop from the tool's output (for Chord, the 'outname_doublets.csv' file, or use the overkill mechanism where overkillrate removes 'the top ?% doublet-liked cells'). Re-run the downstream analysis on the filtered matrix — here, reconstruct bifurcating trajectories with Slingshot (cluster-based MST pseudotime) and monocle (reversed graph embedding). Judge success by comparing branch structure to the clean doublet-free reference: good filters (Chord, Scrublet) preserve the original bifurcation with no extra branches, while poor ones (bcds, cxds, doubletCells) introduce spurious branches.

## Related tools

- **Chord** (Ensemble doublet scorer (GBM/AdaBoost over built-in methods) whose ranked doublet calls are filtered and optionally refined with the overkill mechanism) — github.com/13308204545/Chord
- **Scrublet** (Python doublet scorer using simulated doublets and a k-NN classifier; its continuous score is ranked and thresholded) — github.com/AllonKleinLab/scrublet
- **scds** (Dependency providing the cxds and bcds base scores used by Chord and compared in the rank-filtering benchmark) — github.com/kostkalab/scds
- **DoubletFinder** (Base method whose artificial-doublet pANN scores feed the Chord ensemble and are ranked for filtering) — github.com/chris-mcginnis-ucsf/DoubletFinder
- **Slingshot** (Reconstructs cluster-based MST lineage/pseudotime trajectories on filtered data to validate doublet removal)
- **monocle** (Reconstructs reversed-graph-embedding trajectories on filtered data as a second validation view)

## Examples

```
chord(seu = seu, doubletrate = 0.2, overkill = TRUE, outname = "DataPSE")  # then read DataPSE_doublets.csv and drop those barcodes before Slingshot/monocle
```

## Evaluation signals

- After removing the top 20% of cells by each method's score, Slingshot and monocle trajectories from Chord- and Scrublet-filtered data show no extra branches relative to the clean doublet-free reference
- bcds-, cxds- and doubletCells-filtered data introduce spurious additional branches in Slingshot and/or monocle — expected failure signature used as a negative control
- The fraction removed exactly matches the estimated doublet rate (e.g. 0.20 for DataPSE's 600-cell dataset), ensuring equal stringency across methods
- Chord's doublet barcodes are emitted to 'outname_doublets.csv'; verify the removed set matches this file
- All methods should be evaluated on the same input object and the same number of cells (600) before and after filtering

## Limitations

- Chord is 'not optimal in terms of time efficiency' because it must integrate the results of multiple methods
- Score-rank filtering at a fixed fraction assumes the known doublet rate is correct; over-aggressive removal can distort the trajectory
- Computational doublet detection performance 'varies greatly' across datasets and 'no method can be considered as the gold standard for each scenario'
- Heterotypic doublets most impact downstream dimensionality reduction, clustering, differential expression and developmental trajectories, so residual heterotypic doublets below the cutoff can still create spurious branches

## Evidence

- [readme] overkillrate discussion: "overkillrate  an integer,remove the top ?% doublet-liked cells of any methods' results.(0-1)"
- [readme] doublets csv: "The doublets' barcodes are in the file "outname_doublets.csv""
- [abstract] accuracy claim: "Chord had higher accuracy and stability than the individual approaches on different datasets containing real and synthetic data."
- [intro] doublet impact on trajectories: "heterotypic doublets are considered to have more impact on downstream analyses, including dimensionality reduction, cell clustering, differential expression, and cell developmental trajectories"
- [discussion] variable performance: "the performance of these computational methods varies greatly"
