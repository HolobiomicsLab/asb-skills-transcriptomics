---
name: cell-cycle-gene-regression-preprocessing
description: Use when applying UniTVelo to a pancreas AnnData with spliced/unspliced
  counts where a known subtle fate transition (e.g., Fev+ to Delta) must be recovered
  both with and without cell-cycle gene regression, to match a reference fate-mapping
  result.
when_to_use_negative:
- When the goal is a unified latent time across the whole transcriptome — UniTVelo's
  default Unified-time mode (FIT_OPTION='1') is the appropriate choice, not the Independent
  mode used here
- When cell-cycle phase itself is the biological signal of interest (e.g., retina
  development with a clear cell cycle phase), since regressing out cell-cycle genes
  would remove the signal under study
- When input is raw counts without scVelo preprocessing — gene filtering and moment
  computation must be performed before utv.run_model
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: UniTVelo
  role: Fits the RNA velocity model on the pancreas data via utv.run_model in Independent
    mode (FIT_OPTION='2') and provides utv.evaluate for Cross Boundary Direction Correctness
  repo: https://github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Preprocessing of spliced/unspliced counts (gene filtering and moment computation)
    and velocity embedding stream visualization
- name: CellRank
  role: Reference cell fate mapping result (Supplementary Fig. S7d,e) against which
    the Fev+ to Delta velocity transition is compared
- name: Python 3
  role: Implementation environment for the UniTVelo pipeline
- name: TensorFlow
  role: Backend on which UniTVelo's automatic-differentiation model fitting runs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:e8eb34b0e13b6b22747bcdf71edd6ff3e65c41e5be9cefbce7028bf2cc49711f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/StatBiomed/UniTVelo
  tools:
  - CellRank
  - Python 3
  - TensorFlow
  - UniTVelo
  - scVelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cell-cycle-gene-regression-preprocessing

## Summary

A UniTVelo RNA-velocity preprocessing skill that regresses out cell-cycle genes from the pancreas spliced/unspliced AnnData before model fitting, and contrasts the regressed run against a non-regressed run to test whether the subtle Fev+ pre-endocrine to Delta cell transition is preserved in Independent mode (FIT_OPTION='2').

## When to use

Use when applying UniTVelo to a pancreas AnnData with spliced/unspliced counts where a known subtle fate transition (e.g., Fev+ to Delta) must be recovered both with and without cell-cycle gene regression, to match a reference fate-mapping result.

## When NOT to use

- When the goal is a unified latent time across the whole transcriptome — UniTVelo's default Unified-time mode (FIT_OPTION='1') is the appropriate choice, not the Independent mode used here
- When cell-cycle phase itself is the biological signal of interest (e.g., retina development with a clear cell cycle phase), since regressing out cell-cycle genes would remove the signal under study
- When input is raw counts without scVelo preprocessing — gene filtering and moment computation must be performed before utv.run_model

## Inputs

- Pancreas dataset AnnData (.h5ad) with spliced/unspliced counts and a cell-type label column in adata.obs
- Cell-type cluster labels for Fev+ pre-endocrine and Delta cells
- cluster_edges specification encoding the Fev+ to Delta transition for utv.evaluate
- utv.config.Configuration object with FIT_OPTION='2' (Independent mode)

## Outputs

- Two velocity-annotated AnnData objects (one without and one with cell-cycle gene regression)
- Velocity embedding stream plots (scv.pl.velocity_embedding_stream) for both runs
- Cross Boundary Direction Correctness metrics for the Fev+ to Delta cluster edge from utv.evaluate, exported in CSV format

## How to apply

Load the pancreas AnnData and preprocess with scVelo (gene filtering and moment computation), then sub-class utv.config.Configuration setting FIT_OPTION='2' for the gene-independent (Independent) mode and specifying the cell-type label column. Run utv.run_model once without cell-cycle regression and once after regressing out cell-cycle genes, rendering each result with scv.pl.velocity_embedding_stream. For each run, call utv.evaluate with cluster_edges encoding the Fev+ pre-endocrine to Delta transition and the ground-truth cluster labels to obtain Cross Boundary Direction Correctness. The rationale is to determine whether the subtle transition, consistent with the CellRank-based fate mapping in Supplementary Fig. S7d,e, depends on cell-cycle-driven structure; the direction-correctness value for the Fev+ to Delta edge must be positive (correct direction) in the run matching the reference setting.

## Related tools

- **UniTVelo** (Fits the RNA velocity model on the pancreas data via utv.run_model in Independent mode (FIT_OPTION='2') and provides utv.evaluate for Cross Boundary Direction Correctness) — https://github.com/StatBiomed/UniTVelo
- **scVelo** (Preprocessing of spliced/unspliced counts (gene filtering and moment computation) and velocity embedding stream visualization)
- **CellRank** (Reference cell fate mapping result (Supplementary Fig. S7d,e) against which the Fev+ to Delta velocity transition is compared)
- **Python 3** (Implementation environment for the UniTVelo pipeline)
- **TensorFlow** (Backend on which UniTVelo's automatic-differentiation model fitting runs)

## Examples

```
velo = utv.config.Configuration(); velo.FIT_OPTION = '2'; adata = utv.run_model(path_to_adata, label, config_file=velo); metrics = utv.evaluate(adata, cluster_edges, label, 'velocity')
```

## Evaluation signals

- Cross Boundary Direction Correctness from utv.evaluate for the Fev+ pre-endocrine to Delta cluster edge is positive (correct direction) in at least the run matching the Supplementary Fig. S7d,e setting
- Stream plots visually reproduce the subtle Fev+ to Delta transition in the expected run
- Output CSV files exist and conform to the expected metrics format
- Configuration check: FIT_OPTION is '2' (Independent mode) and the cell-type label column is correctly specified
- Both runs (with and without cell-cycle regression) are produced and compared for transition direction and strength

## Limitations

- The Fev+ to Delta transition is subtle; direction and strength may change between the cell-cycle-regressed and non-regressed runs
- Possible over-counting in unspliced read counting is not resolved and may affect cell transition metrics from the RNA velocity vector
- No unsupervised method exists for selecting dynamically informed genes, so velocity projections may depend on gene-choice artifacts

## Evidence

- [readme] Independent mode configuration: "velo.FIT_OPTION (str), '1' Unified-time mode (default), '2' Independent mode"
- [readme] Run model invocation: "# 3. Run model (label refers to column name in adata.obs specifying celltypes)"
- [readme] Evaluation metric: "# Cross Boundary Direction Correctness
# Ground truth should be given via `cluster_edges`"
- [discussion] Gene-independent mode support: "UniTVelo also supports a gene-independent mode to assign the latent time to each gene independently, similar to scVelo."
- [methods] Implementation stack: "Temporally unified RNA velocity for single cell trajectory inference (UniTVelo) is implementated on Python 3 and TensorFlow 2."
- [readme] Gene-specific time for complex datasets: "Retaining gene-spcific time matrics (`Independent mode`) for complex datasets"
