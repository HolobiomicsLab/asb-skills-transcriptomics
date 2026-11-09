---
name: activity-regulated-gene-velocity-restriction
description: Use when pseudotime from velocity data must be validated against an externally
  measured elapsed process time (e.g., stimulation duration 0–120 min in scNT-seq
  neurons) and velocity should be estimated only on genes regulated by the perturbation.
when_to_use_negative:
- When the underlying RNA velocity estimation itself is faulty — the article states
  trajectory inference with Cytopath is not meaningful in that case
- When root and terminal cell states are not clearly identifiable from the velocity
  field
- When the biological process of interest is not tied to the genes on which velocity
  is estimated (velocity computed on unrelated gene sets will not track the internal
  clock)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Cytopath
  role: Markov chain simulation-based trajectory inference on the ARG-restricted velocity
    field to yield pseudotime; its Cytopath-Euclidean ablation provides the velocity-free
    control
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Processing of the scNT-seq time-course and estimation of RNA velocity restricted
    to the activity-regulated gene subset
- name: numpy
  role: Computation of per-run Pearson correlations between inferred pseudotime and
    per-cell stimulation duration
- name: pandas
  role: Summarize per-run Pearson correlations for velocity-based and Euclidean-ablation
    variants and export pseudotime_stimulation_correlation.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:31602df77223c2927eab90aef8c264e9721b60bc0ea30cfbfcf49629a7cfcaa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cytopath
  - numpy
  - pandas
  - scipy
  - scvelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# activity-regulated-gene-velocity-restriction

## Summary

Restrict RNA velocity estimation to the activity-regulated gene (ARG) subset before running Cytopath pseudotime on a stimulation time-course (scNT-seq mouse cortical neurons, Qiu et al., 0–120 min), so that the velocity field tracks the transcriptional program actually induced by stimulation rather than background dynamics.

## When to use

Use when pseudotime from velocity data must be validated against an externally measured elapsed process time (e.g., stimulation duration 0–120 min in scNT-seq neurons) and velocity should be estimated only on genes regulated by the perturbation.

## When NOT to use

- When the underlying RNA velocity estimation itself is faulty — the article states trajectory inference with Cytopath is not meaningful in that case
- When root and terminal cell states are not clearly identifiable from the velocity field
- When the biological process of interest is not tied to the genes on which velocity is estimated (velocity computed on unrelated gene sets will not track the internal clock)

## Inputs

- scNT-seq mouse cortical neuron scRNA-seq time-course (GEO: GSE141851, Qiu et al., 0–120 min stimulation durations)
- anndata (h5ad) object with spliced/unspliced counts for velocity estimation
- Activity-regulated gene (ARG) gene list defining the restriction subset
- Per-cell stimulation duration annotations (used as ground-truth 'internal clock')

## Outputs

- Velocity-based Cytopath cell pseudotime computed on the ARG-restricted velocity field
- Cytopath-Euclidean ablation pseudotime (velocity-free control)
- Per-run Pearson correlation values of pseudotime vs. stimulation duration over 10 runs
- Summary table pseudotime_stimulation_correlation.csv

## How to apply

Download the scNT-seq mouse cortical neuron stimulation time-course (GEO: GSE141851; durations 0–120 min) and load it as an anndata object. Process with scvelo and estimate RNA velocity restricted to the activity-regulated gene subset, since these genes directly encode the stimulation response that constitutes the 'internal clock'. Run Cytopath's Markov chain simulation-based trajectory inference on this ARG-restricted velocity field to obtain cell pseudotime, and optionally repeat with the Cytopath-Euclidean ablation (no velocity information) as a negative control. Repeat inference over ~10 independent runs to assess stability, then compute the Pearson correlation of each run's pseudotime with per-cell stimulation duration using numpy/scipy. Success is judged by a monotonic, highly correlated velocity-based pseudotime–duration relationship that exceeds the non-velocity-based ablation's median Pearson correlation.

## Related tools

- **Cytopath** (Markov chain simulation-based trajectory inference on the ARG-restricted velocity field to yield pseudotime; its Cytopath-Euclidean ablation provides the velocity-free control) — https://github.com/aron0093/cytopath
- **scvelo** (Processing of the scNT-seq time-course and estimation of RNA velocity restricted to the activity-regulated gene subset)
- **numpy** (Computation of per-run Pearson correlations between inferred pseudotime and per-cell stimulation duration)
- **pandas** (Summarize per-run Pearson correlations for velocity-based and Euclidean-ablation variants and export pseudotime_stimulation_correlation.csv)

## Examples

```
import scvelo as scv; adata = scv.read('GSE141851.h5ad'); scv.pp.moments(adata); scv.tl.velocity(adata[:, arg_genes]); scv.tl.velocity_graph(adata)
```

## Evaluation signals

- Cytopath pseudotime shows a monotonic, highly correlated relationship with stimulation duration (Pearson r computed per run over 10 runs)
- Velocity-based Cytopath achieves a higher median Pearson correlation than the Cytopath-Euclidean (no-velocity) ablation across the 10 runs
- Per-run correlations are stable across 10 independent Cytopath simulation runs, indicating robust pseudotime inference
- Root and terminal states of the inferred trajectory correspond to pre-stimulation and fully stimulated neuronal states, i.e., endpoints are clearly identifiable

## Limitations

- Trajectory inference with Cytopath is not meaningful when the underlying RNA velocity estimation itself is faulty (e.g., time-invariant rate assumptions violated)
- Pseudotime robustness to root and terminal probability thresholds was reported only within a range of [0.85, 1] and only as 'data not shown'
- Authors do not recommend performing trajectory inference with Cytopath when root and terminal cell states are not clearly identifiable

## Evidence

- [results] scNT-seq mouse cortical neurons dataset (Qiu et al.): "Qiu et al.13 generated a dataset of mouse cortical neurons stimulated for durations ranging from 0–120 min"
- [intro] internal clock assessment: "We also assess the ability of pseudotime estimated by Cytopath to represent the biological process time, also referred to as the “internal clock” of a cell."
- [discussion] neuronal activation correlation analysis: "we investigate in more detail the correlation of pseudotime inferred by Cytopath with stimulation duration for the neuronal activation dataset"
- [readme] velocity-derived transcriptional activity: "Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model; simulation of this model yields an ensemble of possible differentiation trajectories"
- [discussion] velocity prerequisite: "We do not expect that trajectory inference with Cytopath is meaningful when the underlying RNA velocity estimation itself is faulty."
