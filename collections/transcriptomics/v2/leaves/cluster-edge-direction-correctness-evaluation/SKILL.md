---
name: cluster-edge-direction-correctness-evaluation
description: Use when a spliced/unspliced AnnData with ground-truth cell-type labels
  has been processed by UniTVelo (or scVelo) and you must validate directionality
  of specific cluster-to-cluster transitions, e.g. Fev+ to Delta.
when_to_use_negative:
- No ground-truth cell-type annotations or known cluster_edges exist, since utv.evaluate
  requires 'Ground truth should be given via cluster_edges'
- The AnnData has no 'velocity' field yet (run_model/run scVelo must complete first);
  evaluating an un-run model yields undefined metrics
- The goal is goodness-of-fit of gene dynamics rather than trajectory directionality
  — use the R2 coefficient of determination for dynamically informed genes instead
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: UniTVelo
  role: Provides utv.evaluate for Cross Boundary Direction Correctness and utv.run_model
    for the velocity being evaluated
  repo: github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Preprocesses the AnnData (gene filtering, moments) and renders the scv.pl.velocity_embedding_stream
    corroboration plot
- name: CellRank
  role: Source of the cell fate-mapping result used as the biological reference for
    the Fev+ to Delta transition (Supplementary Fig. S7d,e)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:126d499f10deef9919a8a80078d412f0c2241d6bb317cd320eac8dcb5ca52e0e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellRank
  - Python 3
  - UniTVelo
  - scVelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cluster-edge-direction-correctness-evaluation

## Summary

Quantifies whether inferred RNA velocity points in the biologically correct direction across known cell-state transitions using UniTVelo's utv.evaluate Cross Boundary Direction Correctness metric, with cluster_edges encoding ground-truth transitions such as Fev+ pre-endocrine to Delta cells.

## When to use

Use when a spliced/unspliced AnnData with ground-truth cell-type labels has been processed by UniTVelo (or scVelo) and you must validate directionality of specific cluster-to-cluster transitions, e.g. Fev+ to Delta.

## When NOT to use

- No ground-truth cell-type annotations or known cluster_edges exist, since utv.evaluate requires 'Ground truth should be given via cluster_edges'
- The AnnData has no 'velocity' field yet (run_model/run scVelo must complete first); evaluating an un-run model yields undefined metrics
- The goal is goodness-of-fit of gene dynamics rather than trajectory directionality — use the R2 coefficient of determination for dynamically informed genes instead

## Inputs

- AnnData with spliced/unspliced counts preprocessed by scVelo and a 'velocity' field computed by UniTVelo (utv.run_model output)
- cluster_edges: list of (source_cluster, target_cluster) tuples encoding known transitions, e.g. Fev+ pre-endocrine to Delta
- Label string naming the adata.obs column with ground-truth cell-type annotations

## Outputs

- Dict of Cross Boundary Direction Correctness (and related) metrics per cluster edge, serializable to CSV
- Velocity embedding stream plots (scv.pl.velocity_embedding_stream) used as visual corroboration

## How to apply

After running utv.run_model on the scVelo-preprocessed pancreas AnnData, define cluster_edges as pairs of (source, target) cell types encoding the expected transition (e.g. ('Fev+ pre-endocrine', 'Delta')) and pass the adata.obs column holding ground-truth cell-type labels. Call utv.evaluate(adata, cluster_edges, label, 'velocity'), which returns a dict of Cross Boundary Direction Correctness values per cluster edge; a positive value means the velocity field points from source to target, consistent with CellRank-based fate mapping (Supplementary Fig. S7d,e). In the UniTVelo task the metric is computed both for the Independent-mode run (FIT_OPTION='2' set on a utv.config.Configuration object) and for the run with cell-cycle genes regressed out, so the two runs can be compared in direction and strength. Pair the numeric check with a scv.pl.velocity_embedding_stream plot to visually confirm the subtle Fev+ to Delta transition.

## Related tools

- **UniTVelo** (Provides utv.evaluate for Cross Boundary Direction Correctness and utv.run_model for the velocity being evaluated) — github.com/StatBiomed/UniTVelo
- **scVelo** (Preprocesses the AnnData (gene filtering, moments) and renders the scv.pl.velocity_embedding_stream corroboration plot)
- **CellRank** (Source of the cell fate-mapping result used as the biological reference for the Fev+ to Delta transition (Supplementary Fig. S7d,e))

## Examples

```
velo = utv.config.Configuration(); velo.FIT_OPTION = '2'; adata = utv.run_model(path_to_adata, label, config_file=velo); metrics = utv.evaluate(adata, cluster_edges, label, 'velocity')
```

## Evaluation signals

- The direction-correctness value for the Fev+ pre-endocrine to Delta cluster edge is positive in the run matching the Supplementary Fig. S7d,e setting
- utv.evaluate returns a dict populated with metrics (not an empty {} placeholder) for every cluster edge supplied
- The stream plot visually reproduces the subtle Fev+ to Delta transition consistent with CellRank fate mapping on both pancreas datasets
- Output metric files exist and are in correct CSV format
- Direction and strength of the Fev+ to Delta transition are comparable between the with- and without-cell-cycle-regression runs

## Limitations

- Possible over-counting in unspliced reads can make a few genes dramatically affect cell transition metrics from the RNA velocity vector
- Existing velocity methods may return erroneous or inconsistent results due to low signal-to-noise ratio in unspliced mRNAs and violated model assumptions
- Direction correctness requires curated ground-truth labels and cluster_edges; it cannot validate transitions absent from the annotation

## Evidence

- [methods] Cross Boundary Direction Correctness with ground truth via cluster_edges: "# Cross Boundary Direction Correctness
# Ground truth should be given via `cluster_edges`"
- [methods] utv.evaluate invocation signature: "metrics = utv.evaluate(adata, cluster_edges, label, 'velocity')"
- [discussion] Over-counting caveat affecting transition metrics: "the possible over-counting may explain that a few genes dramatically affect cell transition metrics from the RNA velocity vector"
- [readme] Label argument definition for running and evaluating the model: "3. Run model (label refers to column name in adata.obs specifying celltypes)"
