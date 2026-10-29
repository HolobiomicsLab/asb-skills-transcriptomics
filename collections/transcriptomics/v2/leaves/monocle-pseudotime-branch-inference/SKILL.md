---
name: monocle-pseudotime-branch-inference
description: Use when single-cell scRNA-seq data along a developmental (bifurcating)
  trajectory needs pseudotime and branch structure inference, especially to check
  whether doublet removal preserved the lineage topology.
when_to_use_negative:
- Data are not along a developmental trajectory (e.g., dissociated tissue mixtures
  with no lineage ordering) — branch inference is not meaningful
- Doublets have not been scored or filtered and the goal is doublet detection itself
  rather than assessing its effect on trajectory topology
- Only clustering or differential expression is needed; Slingshot/monocle trajectory
  inference adds no value
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: monocle
  role: Reversed graph embedding method used to reconstruct the developmental trajectory
    and branch structure from filtered datasets
- name: Slingshot
  role: Parallel cluster-based minimum spanning tree lineage and pseudotime inference
    used alongside monocle for the branch-structure comparison
- name: Chord
  role: Ensemble doublet scorer whose top-20%-filtered cells are re-trajectoried with
    monocle to verify branch preservation
  repo: github.com/13308204545/Chord
- name: Scrublet
  role: Doublet scorer whose filtered cells are re-trajectoried with monocle for comparison
  repo: github.com/AllonKleinLab/scrublet
- name: scds
  role: Provides the bcds and cxds doublet scores integrated by Chord and evaluated
    independently in the trajectory comparison
  repo: github.com/kostkalab/scds
- name: doubletCells
  role: Scran doublet scorer whose filtered cells introduced spurious branches in
    monocle/Slingshot trajectories
- name: DoubletFinder
  role: Seurat-based doublet method integrated inside Chord's GBM boosting ensemble
  repo: github.com/chris-mcginnis-ucsf/DoubletFinder
- name: R
  role: Implementation environment for the full filter-and-re-trajectory workflow
    (Chord, scds, Slingshot, monocle are R packages)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:0485a501e079209e8a383df494c9402eba8db469d7811d1991828e06506d5f78
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# monocle-pseudotime-branch-inference

## Summary

Reconstruct developmental trajectories with monocle's reversed graph embedding to test whether doublet contamination or doublet filtering distorts the branch structure of a bifurcating single-cell trajectory, using extra spurious branches as the failure signal.

## When to use

Use when single-cell scRNA-seq data along a developmental (bifurcating) trajectory needs pseudotime and branch structure inference, especially to check whether doublet removal preserved the lineage topology.

## When NOT to use

- Data are not along a developmental trajectory (e.g., dissociated tissue mixtures with no lineage ordering) — branch inference is not meaningful
- Doublets have not been scored or filtered and the goal is doublet detection itself rather than assessing its effect on trajectory topology
- Only clustering or differential expression is needed; Slingshot/monocle trajectory inference adds no value

## Inputs

- scRNA-seq expression matrix with cells along a bifurcating trajectory (e.g., the synthetic DataPSE dataset, 600 cells, 20% labelled doublets)
- Doublet-filtered cell sets (per method: cells remaining after removing the top 20% by doublet score)
- Clean doublet-free reference trajectory for branch-structure comparison

## Outputs

- Monocle pseudotime trajectory with branch structure per filtered dataset
- Per-method comparison of branch structure against the clean-data trajectory (extra spurious branches: yes/no)
- Comparison figure (trajectory_comparison.png)

## How to apply

Load the single-cell dataset (e.g., a Seurat or SingleCellExperiment object) and run monocle's reversed graph embedding to infer pseudotime and branch structure. In the Chord validation workflow, monocle is applied to each doublet-filtered dataset — created by removing the top 20% of cells ranked by each method's doublet score (Chord, Scrublet, bcds, cxds, doubletCells) — matching the known 20% doublet proportion so all methods are compared at equal stringency. Run monocle in parallel with Slingshot (cluster-based minimum spanning tree lineage and pseudotime inference) and record whether the reconstructed trajectory matches the clean doublet-free reference or gains extra spurious branches. In the reported comparison, Chord- and Scrublet-filtered data yielded monocle trajectories with no new branches, while bcds-, cxds- and doubletCells-filtered data introduced spurious branches. Compile the per-method branch-structure comparison (saved as trajectory_comparison.png) as the deliverable.

## Related tools

- **monocle** (Reversed graph embedding method used to reconstruct the developmental trajectory and branch structure from filtered datasets)
- **Slingshot** (Parallel cluster-based minimum spanning tree lineage and pseudotime inference used alongside monocle for the branch-structure comparison)
- **Chord** (Ensemble doublet scorer whose top-20%-filtered cells are re-trajectoried with monocle to verify branch preservation) — github.com/13308204545/Chord
- **Scrublet** (Doublet scorer whose filtered cells are re-trajectoried with monocle for comparison) — github.com/AllonKleinLab/scrublet
- **scds** (Provides the bcds and cxds doublet scores integrated by Chord and evaluated independently in the trajectory comparison) — github.com/kostkalab/scds
- **doubletCells** (Scran doublet scorer whose filtered cells introduced spurious branches in monocle/Slingshot trajectories)
- **DoubletFinder** (Seurat-based doublet method integrated inside Chord's GBM boosting ensemble) — github.com/chris-mcginnis-ucsf/DoubletFinder
- **R** (Implementation environment for the full filter-and-re-trajectory workflow (Chord, scds, Slingshot, monocle are R packages))

## Examples

```
R: run monocle reversed graph embedding on the filtered SingleCellExperiment, then compare branch structure vs the clean reference (trajectory_comparison.png); e.g. `monocle::reducedDimension(cds, method='DDRTree')` after removing the top 20% of cells by each doublet score
```

## Evaluation signals

- Trajectory from Chord- or Scrublet-filtered data shows no extra branches relative to the clean doublet-free reference trajectory
- Trajectories from bcds-, cxds- and doubletCells-filtered data show additional spurious branches in Slingshot and/or monocle (expected negative controls)
- Branch structure is consistent across both monocle (reversed graph embedding) and Slingshot (cluster-based MST) reconstructions
- All methods compared at equal stringency: exactly the top 20% of cells removed per method's doublet score, matching the 20% labelled doublet proportion
- Final deliverable figure (trajectory_comparison.png) reproduces the reported per-method pattern

## Limitations

- Heterotypic doublets have disproportionate impact on cell developmental trajectories, so residual doublets can create spurious branches even after partial filtering
- Performance of doublet detection methods varies greatly across datasets, so which methods preserve trajectory topology is dataset-dependent — no method is a gold standard for each scenario
- Comparison is against a synthetic bifurcating dataset (DataPSE) with known 20% doublet proportion; results may not transfer to real data without labelled doublets

## Evidence

- [intro] heterotypic doublets are considered to have more impact on downstream analyses, including dimensionality reduction, cell clustering, differential expression, and cell developmental trajectories: "heterotypic doublets are considered to have more impact on downstream analyses, including dimensionality reduction, cell clustering, differential expression, and cell developmental trajectories"
- [intro] the performance of these computational methods varies greatly ... no method can be considered as the gold standard for each scenario: [evidence span withheld — anchor status: not_found]
- [results] After filtering the top 20% scored cells and re-inferring trajectories, Chord and Scrublet produced trajectories similar to the clean data with no new branches: [evidence span withheld — anchor status: not_found]
- [methods] Reconstruct the developmental trajectory from each filtered dataset with Slingshot (cluster-based minimum spanning tree lineage and pseudotime inference) and with monocle (reversed graph embedding): [evidence span withheld — anchor status: not_found]
- [methods] Load the DataPSE synthetic scRNA-seq dataset (600 cells along a bifurcating trajectory with 20% labelled doublets) into R: [evidence span withheld — anchor status: not_found]
