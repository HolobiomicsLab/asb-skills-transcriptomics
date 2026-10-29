---
name: doublet-removal-effect-on-trajectory-branching-comparison
description: Use when you need to benchmark doublet detection methods by their downstream
  effect on developmental trajectory inference, especially on synthetic data with
  known doublet labels and a bifurcating trajectory.
when_to_use_negative:
- You lack a reference trajectory (clean data or known ground truth) against which
  to compare branch structure — the evaluation depends on it
- The dataset has no doublet contamination or the doublet proportion is unknown, so
  the fixed top-20% removal fraction cannot be justified
- Your analysis goal is doublet classification accuracy (precision/recall vs labels),
  not downstream trajectory robustness
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3611
tools:
- name: Chord
  role: Ensemble doublet scorer (GBM boosting of scds cxds/bcds and DoubletFinder)
    whose filtered data should preserve trajectory branching
  repo: github.com/13308204545/Chord
- name: Scrublet
  role: k-NN doublet scorer used as an independent filtering baseline for trajectory
    re-inference
  repo: github.com/AllonKleinLab/scrublet
- name: scds
  role: Provides the cxds and bcds individual doublet scores evaluated in the comparison
  repo: github.com/kostkalab/scds
- name: DoubletFinder
  role: Artificial-doublet-based scorer integrated into the Chord ensemble
  repo: github.com/chris-mcginnis-ucsf/DoubletFinder
- name: doubletCells
  role: scran-based doublet scorer whose filtering introduced spurious branches in
    the benchmark
- name: Slingshot
  role: Cluster-based MST lineage and pseudotime inference used to re-derive trajectories
    from filtered data
- name: monocle
  role: Reversed graph embedding trajectory inference used as a second trajectory
    method
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:02bdf8f9d544e0a7078118f9bb42b9bc357d235e70d4700c94219bf0673e9d1c
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3611
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# doublet-removal-effect-on-trajectory-branching-comparison

## Summary

A benchmarking procedure that scores cells for doublets with multiple detection methods (Chord, Scrublet, bcds, cxds, doubletCells), removes the top-scoring fraction matching the known doublet rate, and tests whether Slingshot/monocle trajectory re-inference reproduces the clean-data branch structure or introduces spurious branches.

## When to use

Use when you need to benchmark doublet detection methods by their downstream effect on developmental trajectory inference, especially on synthetic data with known doublet labels and a bifurcating trajectory.

## When NOT to use

- You lack a reference trajectory (clean data or known ground truth) against which to compare branch structure — the evaluation depends on it
- The dataset has no doublet contamination or the doublet proportion is unknown, so the fixed top-20% removal fraction cannot be justified
- Your analysis goal is doublet classification accuracy (precision/recall vs labels), not downstream trajectory robustness

## Inputs

- Synthetic scRNA-seq dataset with labelled doublets and a bifurcating trajectory (e.g. DataPSE: 600 cells, 20% doublets)
- Clean doublet-free reference trajectory from the same data
- Doublet scores per cell from Chord, Scrublet, bcds, cxds, and doubletCells

## Outputs

- Per-method trajectory branch-structure comparison vs the clean-data reference
- Record of which methods introduce spurious extra branches in Slingshot and/or monocle
- Comparison figure (e.g. trajectory_comparison.png)

## How to apply

Load a synthetic scRNA-seq dataset with known doublet labels and a bifurcating trajectory (e.g. DataPSE, 600 cells with 20% labelled doublets) into R. Compute a doublet score per cell with each method — Chord (GBM-boosted ensemble of scds cxds/bcds and DoubletFinder), Scrublet, bcds, cxds, and doubletCells. For each method independently, remove the top 20% of cells ranked by score, matching the known 20% doublet proportion so all methods are compared at equal stringency. Reconstruct the trajectory from each filtered dataset with Slingshot (cluster-based minimum spanning tree) and monocle (reversed graph embedding), then compare branch structure against the clean doublet-free reference trajectory, recording whether extra spurious branches appear.

## Related tools

- **Chord** (Ensemble doublet scorer (GBM boosting of scds cxds/bcds and DoubletFinder) whose filtered data should preserve trajectory branching) — github.com/13308204545/Chord
- **Scrublet** (k-NN doublet scorer used as an independent filtering baseline for trajectory re-inference) — github.com/AllonKleinLab/scrublet
- **scds** (Provides the cxds and bcds individual doublet scores evaluated in the comparison) — github.com/kostkalab/scds
- **DoubletFinder** (Artificial-doublet-based scorer integrated into the Chord ensemble) — github.com/chris-mcginnis-ucsf/DoubletFinder
- **doubletCells** (scran-based doublet scorer whose filtering introduced spurious branches in the benchmark)
- **Slingshot** (Cluster-based MST lineage and pseudotime inference used to re-derive trajectories from filtered data)
- **monocle** (Reversed graph embedding trajectory inference used as a second trajectory method)

## Examples

```
chord(seu = seu, doubletrate = 0.20, overkill = TRUE, outname = "datapse")  # then remove top 20% by each method's score and re-run Slingshot/monocle
```

## Evaluation signals

- Chord- and Scrublet-filtered datasets yield Slingshot and monocle trajectories with no extra branches relative to the clean-data trajectory
- bcds-, cxds- and doubletCells-filtered datasets show additional spurious branches in Slingshot and/or monocle (the expected negative result)
- All methods are compared at identical stringency: exactly the top 20% of scored cells removed, matching the 20% known doublet proportion
- Branch structure (number and position of bifurcations) of each re-inferred trajectory is compared cell-by-cell against the doublet-free reference
- Reproduced figure (trajectory_comparison.png) matches the reported per-method result

## Limitations

- Requires ground truth: both the doublet labels (to set the removal fraction) and a clean-data reference trajectory; results on synthetic data may not transfer to real data without these
- Removing a fixed top-20% of cells by score is exact-stringency only for data with a known 20% doublet rate; with unknown rates the equal comparison breaks down
- Doublet contamination affects dimensionality reduction, clustering, differential expression and trajectories — this skill probes only the trajectory-branching consequence

## Evidence

- [intro] Heterotypic doublets are considered to have more impact on downstream analyses, including dimensionality reduction, cell clustering, differential expression, and cell developmental trajectories: [evidence span withheld — anchor status: not_found]
- [methods] Doublet scores drive filtering at fixed fraction: [evidence span withheld — anchor status: not_found]
- [results] Task workflow records the trajectory comparison outcome: [evidence span withheld — anchor status: not_found]
- [readme] Scrublet scoring mechanism: [evidence span withheld — anchor status: not_found]
