---
name: slingshot-trajectory-reconstruction
description: Use when single-cell data is expected to follow a continuous developmental
  or bifurcating trajectory and you need lineage/pseudotime inference, or when validating
  that filtering steps (e.g. doublet removal) did not distort branch structure.
when_to_use_negative:
- Data has no expected continuous or bifurcating developmental structure (e.g. discrete
  immune cell types) — cluster-based MST lineage inference would impose a pseudo-trajectory
- Input is already a completed trajectory/pseudotime object and the goal is only downstream
  differential expression along it
- The question is doublet-detection accuracy itself (AUC/precision) rather than downstream
  trajectory integrity — score-based benchmarks are more direct
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: Slingshot
  role: Cluster-based minimum spanning tree lineage and pseudotime inference on the
    filtered dataset
- name: monocle
  role: Independent reversed graph embedding trajectory inference used to cross-check
    Slingshot branch structure
- name: Chord
  role: Upstream doublet removal (GBM-boosted ensemble of scds cxds/bcds and DoubletFinder)
    whose filtering preserves the bifurcating trajectory
  repo: https://github.com/13308204545/Chord
- name: Scrublet
  role: Upstream doublet scoring whose filtered data also yields trajectories matching
    the clean reference
  repo: https://github.com/AllonKleinLab/scrublet
- name: scds
  role: Provides cxds and bcds doublet scores used as comparison filters that introduced
    spurious branches
  repo: https://github.com/kostkalab/scds
- name: DoubletFinder
  role: Component doublet detector integrated into Chord's ensemble
  repo: https://github.com/chris-mcginnis-ucsf/DoubletFinder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-022-03476-9
    title: 'Chord: an ensemble machine learning algorithm to identify doublets in
      single-cell RNA sequencing data'
  source_content_hash: sha256:84fd2883b8c4264f7b944840091ebc48a826985c773775c009f6f268c1c5df70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/13308204545/Chord
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# slingshot-trajectory-reconstruction

## Summary

Reconstruct developmental trajectories and pseudotime from scRNA-seq data with Slingshot's cluster-based minimum spanning tree lineage inference, and use the resulting branch structure as a sensitive readout of data quality — e.g. detecting whether doublet removal preserved a bifurcating trajectory or introduced spurious branches.

## When to use

Use when single-cell data is expected to follow a continuous developmental or bifurcating trajectory and you need lineage/pseudotime inference, or when validating that filtering steps (e.g. doublet removal) did not distort branch structure.

## When NOT to use

- Data has no expected continuous or bifurcating developmental structure (e.g. discrete immune cell types) — cluster-based MST lineage inference would impose a pseudo-trajectory
- Input is already a completed trajectory/pseudotime object and the goal is only downstream differential expression along it
- The question is doublet-detection accuracy itself (AUC/precision) rather than downstream trajectory integrity — score-based benchmarks are more direct

## Inputs

- R SingleCellExperiment/Seurat object of filtered scRNA-seq counts (doublets removed, e.g. top 20% scored cells removed)
- Clean doublet-free reference dataset for trajectory comparison
- Synthetic bifurcating trajectory dataset (e.g. DataPSE, 600 cells with 20% labelled doublets)

## Outputs

- Slingshot lineage graph (cluster-based minimum spanning tree) with branch structure
- Pseudotime values per cell along each inferred lineage
- Per-method trajectory comparison (e.g. saved as trajectory_comparison.png) recording extra spurious branches vs. the clean reference

## How to apply

Load the filtered scRNA-seq dataset (e.g. after removing the top 20% of cells ranked by a doublet score, matching the known doublet proportion) into R and run Slingshot's cluster-based minimum spanning tree lineage and pseudotime inference. Run Slingshot in parallel with an independent method such as monocle (reversed graph embedding) so branch-structure conclusions are not tool-specific. Compare the reconstructed trajectory's branch structure against a reference trajectory computed from clean doublet-free data, recording whether extra spurious branches appear rather than relying only on pseudotime correlation. In the DataPSE benchmark, Chord- and Scrublet-filtered data preserved the bifurcating structure with no extra branches, whereas bcds-, cxds- and doubletCells-filtered data introduced spurious branches in Slingshot and/or monocle, so a trajectory-integrity check is a stricter validation than score-based doublet metrics alone.

## Related tools

- **Slingshot** (Cluster-based minimum spanning tree lineage and pseudotime inference on the filtered dataset)
- **monocle** (Independent reversed graph embedding trajectory inference used to cross-check Slingshot branch structure)
- **Chord** (Upstream doublet removal (GBM-boosted ensemble of scds cxds/bcds and DoubletFinder) whose filtering preserves the bifurcating trajectory) — https://github.com/13308204545/Chord
- **Scrublet** (Upstream doublet scoring whose filtered data also yields trajectories matching the clean reference) — https://github.com/AllonKleinLab/scrublet
- **scds** (Provides cxds and bcds doublet scores used as comparison filters that introduced spurious branches) — https://github.com/kostkalab/scds
- **DoubletFinder** (Component doublet detector integrated into Chord's ensemble) — https://github.com/chris-mcginnis-ucsf/DoubletFinder

## Examples

```
slingshot(redDim = reducedDims(sce)$PCA, clusterLabels = sce$cluster); # then compare branch structure against the clean-reference trajectory
```

## Evaluation signals

- Reconstructed trajectory branch structure matches the clean doublet-free reference trajectory with no extra branches
- The bifurcating topology of the original data (e.g. DataPSE's bifurcating trajectory) is preserved after filtering
- Result is concordant between Slingshot and monocle (reversed graph embedding) — no tool-specific spurious branching
- Reproduced comparison figure (trajectory_comparison.png) matches the reported pattern: Chord/Scrublet clean, bcds/cxds/doubletCells spurious
- Removal stringency is matched across methods (e.g. top 20% of cells by doublet score removed, equal to the known 20% doublet proportion)

## Limitations

- Heterotypic doublets specifically distort downstream developmental trajectory inference, so homotypic doublet contamination may not be caught by a trajectory check
- Slingshot is cluster-based, so results depend on clustering resolution; branch-structure artefacts can be method-dependent (bcds, cxds, doubletCells failed in Slingshot and/or monocle, not always both)
- Comparing at a fixed removal fraction (e.g. 20%) assumes the true doublet rate is known; misestimated rates change trajectory outcomes

## Evidence

- [full_text] On the synthetic pseudotime dataset DataPSE (600 cells with 20% labelled doublets and a bifurcating trajectory): [evidence span withheld — anchor status: not_found]
- [results] Chord and Scrublet produced trajectories similar to the clean data with no new branches, whereas bcds, cxds and doubletCells introduced spurious additional branches: [evidence span withheld — anchor status: not_found]
- [full_text] Reconstruct the developmental trajectory from each filtered dataset with Slingshot (cluster-based minimum spanning tree lineage and pseudotime inference) and with monocle (reversed graph embedding): [evidence span withheld — anchor status: not_found]
- [intro] heterotypic doublets are considered to have more impact on downstream analyses, including dimensionality reduction, cell clustering, differential expression, and cell developmental trajectories: "heterotypic doublets are considered to have more impact on downstream analyses, including dimensionality reduction, cell clustering, differential expression, and cell developmental trajectories"
- [full_text] For each method independently, remove the top 20% of cells ranked by its doublet score, matching the known 20% doublet proportion: [evidence span withheld — anchor status: not_found]
