---
name: non-velocity-ablation-benchmarking
description: Use when a trajectory method uses RNA velocity and you need to quantify
  its contribution by ablating velocity (distance-only variant) and comparing pseudotime
  against an elapsed-time or known-ordering reference.
when_to_use_negative:
- The underlying RNA velocity estimation is faulty — the authors state trajectory
  inference with Cytopath is not meaningful in that case, so the ablation comparison
  would test against a broken baseline.
- Root and terminal cell states are not clearly identifiable; the authors do not recommend
  performing trajectory inference with Cytopath in that setting.
- No external reference of elapsed process time or known cell ordering exists to serve
  as the ground-truth clock for correlation.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Cytopath
  role: Velocity-based Markov chain simulation trajectory inference providing the
    pseudotime being benchmarked
  repo: github.com/aron0093/cytopath
- name: Cytopath-Euclidean
  role: Velocity-ablated (no velocity information) ablation variant used as the negative-control
    baseline
  repo: https://cytopath.readthedocs.io/en/latest/
- name: scvelo
  role: RNA velocity estimation on the activity-regulated gene subset feeding the
    Cytopath transition matrix
- name: numpy
  role: Pearson correlation of inferred pseudotime with per-cell stimulation duration
- name: scipy
  role: Statistical correlation computation between pseudotime and stimulation duration
- name: pandas
  role: Assembling the per-run correlation table for export to CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:bf86bbdde44d167f30925789f7bb15819f037cf980edcbaf71315692cf700a26
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytopath
  - Cytopath-Euclidean
  - numpy
  - pandas
  - scipy
  - scvelo
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# non-velocity-ablation-benchmarking

## Summary

Benchmarks velocity-based Cytopath pseudotime against a velocity-ablated (Cytopath-Euclidean) variant by correlating each with an external ground-truth clock (stimulation duration), quantifying how much RNA velocity contributes to pseudotime accuracy.

## When to use

Use when a trajectory method uses RNA velocity and you need to quantify its contribution by ablating velocity (distance-only variant) and comparing pseudotime against an elapsed-time or known-ordering reference.

## When NOT to use

- The underlying RNA velocity estimation is faulty — the authors state trajectory inference with Cytopath is not meaningful in that case, so the ablation comparison would test against a broken baseline.
- Root and terminal cell states are not clearly identifiable; the authors do not recommend performing trajectory inference with Cytopath in that setting.
- No external reference of elapsed process time or known cell ordering exists to serve as the ground-truth clock for correlation.

## Inputs

- Preprocessed spliced/unspliced count matrix (anndata/.h5ad) with per-cell stimulation duration labels (scNT-seq GSE141851, 0–120 min)
- RNA velocity field estimated with scvelo on the activity-regulated gene (ARG) subset
- Pre-computed or internally derived Louvain clustering for terminal-state identification

## Outputs

- Per-cell pseudotime vectors from full velocity-based Cytopath (one per independent run)
- Per-cell pseudotime vectors from the Cytopath-Euclidean ablation (one per independent run)
- Summary table (e.g. pseudotime_stimulation_correlation.csv) of per-run Pearson correlations with stimulation duration for both variants

## How to apply

Run Cytopath's Markov-chain simulation-based trajectory inference on the RNA velocity field (here scvelo velocities restricted to activity-regulated genes from the scNT-seq GSE141851 cortical neuron dataset, 0–120 min stimulation). Then rerun the same pipeline with the Cytopath-Euclidean ablation, which removes velocity information so cell assignment relies only on static expression profiles. Repeat both variants over 10 independent runs to assess stability, compute the Pearson correlation of each run's pseudotime with per-cell stimulation duration (numpy/scipy), and summarize per-run correlations for both variants (e.g. to pseudotime_stimulation_correlation.csv). The velocity-based variant should show a monotonic, higher median correlation with the internal clock than the Euclidean ablation; if ablation matches full Cytopath, velocity is not contributing.

## Related tools

- **Cytopath** (Velocity-based Markov chain simulation trajectory inference providing the pseudotime being benchmarked) — github.com/aron0093/cytopath
- **Cytopath-Euclidean** (Velocity-ablated (no velocity information) ablation variant used as the negative-control baseline) — https://cytopath.readthedocs.io/en/latest/
- **scvelo** (RNA velocity estimation on the activity-regulated gene subset feeding the Cytopath transition matrix)
- **numpy** (Pearson correlation of inferred pseudotime with per-cell stimulation duration)
- **scipy** (Statistical correlation computation between pseudotime and stimulation duration)
- **pandas** (Assembling the per-run correlation table for export to CSV)

## Examples

```
import cytopath; cytopath.cytopath(adata, velocity_key='velocity', basis='pca', n_jobs=8)  # repeat with Euclidean ablation; scipy.stats.pearsonr(adata.obs['cytopath_pseudotime'], adata.obs['stim_duration'])
```

## Evaluation signals

- Velocity-based Cytopath pseudotime shows a monotonic, highly correlated relationship with stimulation duration across 0–120 min.
- Cytopath-Euclidean achieves a lower median Pearson correlation over the 10 runs than the velocity-based variant.
- Per-run correlations are stable across 10 independent runs (low run-to-run variance) for the velocity-based variant.
- Both variants produce pseudotime vectors covering the same cells as the input anndata object with no missing assignments.
- The exported CSV contains one row per run per variant with a valid Pearson correlation in [-1, 1].

## Limitations

- Cytopath pseudotime is reported to be robust to root and terminal probability thresholds only within [0.85, 1], and that robustness was noted as 'data not shown'.
- RNA velocity can misrepresent true transcriptional dynamics due to simplifying assumptions (time-invariant rates, single regulatory regime per gene), corrupting both the full method and the ablation benchmark.
- In the erythroid gastrulation dataset, incorrect velocity led to spuriously inferred root and terminal states, so the ablation comparison cannot rescue a bad velocity field.

## Evidence

- [results] Cytopath pseudotime correlates with internal clock on scNT-seq: "Qiu et al.13 generated a dataset of mouse cortical neurons stimulated for durations ranging from 0–120 min"
- [abstract] Velocity-based pseudotime association with elapsed time demonstrated: "We demonstrate the capability to reconstruct differentiation trajectories, assess the association of RNA velocity-based pseudotime with actually elapsed process time"
- [discussion] Root/terminal probability threshold robustness range: "We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown)."
- [discussion] Velocity-ablation benchmarking must not be run on faulty velocity: "We do not expect that trajectory inference with Cytopath is meaningful when the underlying RNA velocity estimation itself is faulty."
- [readme] Cytopath uses RNA velocity Markov chain simulation: "Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model; simulation of this model yields an ensemble of possible differentiation trajectories"
