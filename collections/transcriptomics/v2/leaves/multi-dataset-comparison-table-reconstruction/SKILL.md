---
name: multi-dataset-comparison-table-reconstruction
description: 'Use when a paper reports a benchmark table comparing methods across
  multiple datasets and the goal is to reproduce it end-to-end: run each method-dataset
  pair, compute the reported metric, and aggregate into a CSV table.'
when_to_use_negative:
- The task is a single-dataset velocity analysis rather than reproducing a benchmark
  table across all datasets and comparator modes
- No ground-truth cluster_edges are available, since Cross-Boundary Direction Correctness
  cannot be computed without them
- Datasets are not among the ten public benchmarks and lack the spliced/unspliced
  layers required by scVelo/UniTVelo
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: UniTVelo
  role: Primary method whose CBDir scores are computed on each of the ten datasets
    via utv.run_model and utv.evaluate
  repo: https://github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Comparator baseline run in both 'stochastic' and 'dynamical' modes on the
    same datasets; also used to load datasets into AnnData objects
- name: Python 3
  role: Implementation environment (UniTVelo is implemented on Python 3 and TensorFlow
    2)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:519cd20cd967d857dd1d5df8b2206693b5fbe9153532479ae7d3589e0cdaedc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/StatBiomed/UniTVelo
  tools:
  - Python 3
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# multi-dataset-comparison-table-reconstruction

## Summary

Reconstructs a multi-method, multi-dataset performance comparison table (e.g., Table 1 of the UniTVelo paper) by running UniTVelo and scVelo (stochastic and dynamical modes) across ten public single-cell RNA velocity datasets and aggregating Cross-Boundary Direction Correctness (CBDir) scores into a single CSV.

## When to use

Use when a paper reports a benchmark table comparing methods across multiple datasets and the goal is to reproduce it end-to-end: run each method-dataset pair, compute the reported metric, and aggregate into a CSV table.

## When NOT to use

- The task is a single-dataset velocity analysis rather than reproducing a benchmark table across all datasets and comparator modes
- No ground-truth cluster_edges are available, since Cross-Boundary Direction Correctness cannot be computed without them
- Datasets are not among the ten public benchmarks and lack the spliced/unspliced layers required by scVelo/UniTVelo

## Inputs

- Ten public spliced/unspliced count-matrix datasets (pancreas, dentate gyrus, retina, scNT-seq GSE158646, scEU-seq E-MTAB-7713, hindbrain, mouse erythroid, human erythroid, bone marrow) loadable as AnnData objects via scvelo
- Per-dataset cell-type annotation column in adata.obs (the 'label' argument for utv.run_model)
- Ground-truth cluster_edges per dataset for Cross-Boundary Direction Correctness evaluation
- Per-dataset UniTVelo configuration choices from Supplementary Table S2 (FIT_OPTION '1' or '2')

## Outputs

- table1_cbdc_comparison.csv with one Cross-Boundary Direction Correctness row per method (UniTVelo, scVelo stochastic, scVelo dynamical) x dataset combination
- Fitted AnnData objects with velocity estimates and latent_time per method-dataset pair

## How to apply

Download the ten public count-matrix datasets (pancreas, dentate gyrus, retina, scNT-seq GSE158646, scEU-seq E-MTAB-7713 intestinal organoid, hindbrain, mouse erythroid, human erythroid, bone marrow) and load each into an AnnData object with scvelo. For each dataset, configure UniTVelo by subclassing utv.config.Configuration (R2_ADJUST=True, IROOT=None, FIT_OPTION '1' Unified-time mode or '2' Independent mode per Supplementary Table S2, GPU=0) and run utv.run_model(path_to_adata, label, config_file=velo); run scVelo in both 'stochastic' and 'dynamical' modes on the same AnnData objects as baselines. For every method-dataset pair, compute Cross-Boundary Direction Correctness via utv.evaluate(adata, cluster_edges, label, 'velocity'), supplying ground-truth cluster_edges for each dataset. Aggregate all CBDir scores into a single CSV (table1_cbdc_comparison.csv) with one row per method x dataset combination. Validate by checking that relative rankings per dataset match those reported in Table 1 — e.g., on mouse erythroid, UniTVelo CBDir=0.793 vs scVelo dynamical −0.445 and stochastic 0.129.

## Related tools

- **UniTVelo** (Primary method whose CBDir scores are computed on each of the ten datasets via utv.run_model and utv.evaluate) — https://github.com/StatBiomed/UniTVelo
- **scVelo** (Comparator baseline run in both 'stochastic' and 'dynamical' modes on the same datasets; also used to load datasets into AnnData objects)
- **Python 3** (Implementation environment (UniTVelo is implemented on Python 3 and TensorFlow 2))

## Examples

```
import unitvelo as utv; velo = utv.config.Configuration(); velo.R2_ADJUST=True; velo.IROOT=None; velo.FIT_OPTION='1'; velo.GPU=0; adata = utv.run_model(path_to_adata, label, config_file=velo); metrics = utv.evaluate(adata, cluster_edges, label, 'velocity')
```

## Evaluation signals

- table1_cbdc_comparison.csv exists and contains exactly one CBDir row per method x dataset combination for UniTVelo plus both scVelo modes (3 methods x 10 datasets)
- Mouse erythroid CBDir values match Table 1: UniTVelo 0.793, scVelo dynamical −0.445, scVelo stochastic 0.129
- Per-dataset relative ranking of methods reproduces the ordering reported in Table 1 of the paper
- Each dataset was run with the FIT_OPTION ('1' Unified-time or '2' Independent mode) specified in Supplementary Table S2
- All CBDir values were computed with utv.evaluate using the ground-truth cluster_edges supplied for each dataset

## Limitations

- scVelo baselines on some datasets return negative CBDir (e.g., −0.445 on mouse erythroid dynamical mode), so agreement is judged on relative ranking rather than exact absolute values
- Only 4 of the 10 datasets ship within the scVelo package; the others must be obtained from the authors' external SharePoint link
- GPU memory constraints apply: for datasets larger than 50,000 cells, a GPU with more than 12GB memory may be required; UniTVelo falls back to CPU (about 8x slower on the human erythroid dataset) if no GPU is available

## Evidence

- [other] On the mouse erythroid dataset, UniTVelo achieves a CBDir of 0.793 while scVelo's dynamical mode scores −0.445 and its stochastic mode 0.129: [evidence span withheld — anchor status: not_found]
- [readme] Configure UniTVelo via Configuration and run model: "3. Run model (label refers to column name in adata.obs specifying celltypes)

python3
adata = utv.run_model(path_to_adata, label, config_file=velo)"
- [readme] Evaluation via cluster_edges: "# Cross Boundary Direction Correctness
# Ground truth should be given via `cluster_edges`
metrics = {}
metrics = utv.evaluate(adata, cluster_edges, label, 'velocity')"
- [methods] UniTVelo implementation environment: "Temporally unified RNA velocity for single cell trajectory inference (UniTVelo) is implementated on Python 3 and TensorFlow 2."
- [abstract] Ten datasets demonstrate generalization: "With ten datasets, we demonstrate that UniTVelo returns the expected trajectory in different biological systems, including hematopoietic differentiation and those even with weak kinetics or complex"
- [discussion] GPU memory limits for large datasets: "larger than 50,000 cells for a GPU with 12GB memory"
