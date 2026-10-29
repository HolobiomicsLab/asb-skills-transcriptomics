---
name: pseudotime-stimulation-duration-correlation
description: Use when a scRNA-seq dataset carries a known per-cell elapsed-time label
  (e.g. stimulation minutes) and you need to test whether velocity-derived pseudotime
  tracks biological process time, ideally against a no-velocity ablation.
when_to_use_negative:
- When the underlying RNA velocity estimation is itself faulty — the authors note
  trajectory inference with Cytopath is not meaningful in that case (e.g. spurious
  root/terminal inference on the erythroid gastrulation dataset)
- When root and terminal cell states are not clearly identifiable, which the authors
  explicitly do not recommend for Cytopath trajectory inference
- When the dataset has no per-cell elapsed-time or ordering ground truth to correlate
  pseudotime against
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cytopath
  role: Markov chain simulation-based trajectory inference producing the velocity-based
    pseudotime; the Cytopath-Euclidean ablation removes velocity information for the
    control pseudotime
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: RNA velocity estimation on the activity-regulated gene (ARG) subset, supplying
    the velocity field Cytopath consumes
- name: numpy
  role: Numerical computation of Pearson correlations between pseudotime and stimulation
    duration
- name: scipy
  role: Pearson correlation statistics for each of the 10 independent runs
- name: pandas
  role: Tabulation of per-run correlations and export to pseudotime_stimulation_correlation.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:080f134293a8c0c9b7c433e972362f3f546b8e6f9314a4bb89a80723136193fb
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# pseudotime-stimulation-duration-correlation

## Summary

Validates RNA velocity-based pseudotime (Cytopath) as a proxy for the cell's internal clock by correlating it with a known external time label — stimulation duration (0–120 min) in the scNT-seq mouse cortical neuron dataset — and benchmarking it against the Cytopath-Euclidean ablation that removes velocity information.

## When to use

Use when a scRNA-seq dataset carries a known per-cell elapsed-time label (e.g. stimulation minutes) and you need to test whether velocity-derived pseudotime tracks biological process time, ideally against a no-velocity ablation.

## When NOT to use

- When the underlying RNA velocity estimation is itself faulty — the authors note trajectory inference with Cytopath is not meaningful in that case (e.g. spurious root/terminal inference on the erythroid gastrulation dataset)
- When root and terminal cell states are not clearly identifiable, which the authors explicitly do not recommend for Cytopath trajectory inference
- When the dataset has no per-cell elapsed-time or ordering ground truth to correlate pseudotime against

## Inputs

- scNT-seq mouse cortical neuron stimulation time-course dataset (GEO: GSE141851) in AnnData (.h5ad) format
- Per-cell stimulation duration labels (0–120 min) in adata.obs
- RNA velocity field estimated on the activity-regulated gene (ARG) subset via scvelo
- Cytopath pseudotime and Cytopath-Euclidean ablation pseudotime over 10 independent runs

## Outputs

- Per-run Pearson correlation coefficients of Cytopath and Cytopath-Euclidean pseudotime with stimulation duration
- pseudotime_stimulation_correlation.csv summarizing the per-run correlations for both variants

## How to apply

Download the scNT-seq mouse cortical neuron stimulation time-course (GEO: GSE141851; Qiu et al., 0–120 min) and load it as an AnnData object. Estimate RNA velocity with scvelo restricted to the activity-regulated gene (ARG) subset, since Cytopath leverages transcriptional activity from the velocity field. Run Cytopath's Markov chain simulation-based trajectory inference to obtain cell pseudotime, and repeat with the Cytopath-Euclidean ablation (velocity removed) to yield ablation pseudotime; repeat both inferences over 10 independent runs to assess stability (Cytopath pseudotime is reported as robust to root/terminal probability thresholds within [0.85, 1]). Compute the Pearson correlation of each run's pseudotime against the per-cell stimulation duration with numpy/scipy. Judge success by a monotonic, highly correlated relationship for the velocity-based variant, with higher median Pearson correlation over the 10 runs than the Euclidean ablation.

## Related tools

- **Cytopath** (Markov chain simulation-based trajectory inference producing the velocity-based pseudotime; the Cytopath-Euclidean ablation removes velocity information for the control pseudotime) — https://github.com/aron0093/cytopath
- **scvelo** (RNA velocity estimation on the activity-regulated gene (ARG) subset, supplying the velocity field Cytopath consumes)
- **numpy** (Numerical computation of Pearson correlations between pseudotime and stimulation duration)
- **scipy** (Pearson correlation statistics for each of the 10 independent runs)
- **pandas** (Tabulation of per-run correlations and export to pseudotime_stimulation_correlation.csv)

## Examples

```
from scipy.stats import pearsonr; r = pearsonr(cytopath_pseudotime, adata.obs['stim_duration'])[0]
```

## Evaluation signals

- Velocity-based Cytopath pseudotime shows a monotonic, highly correlated relationship with stimulation duration (0–120 min)
- Median Pearson correlation over 10 runs is higher for Cytopath than for the Cytopath-Euclidean (no-velocity) ablation
- Per-run correlations are stable across the 10 independent inference runs, consistent with reported robustness to root/terminal probability thresholds in [0.85, 1]
- Output table pseudotime_stimulation_correlation.csv contains one Pearson r per run for both variants (velocity-based and Euclidean ablation)

## Limitations

- Cytopath pseudotime is only as good as the velocity estimate: simplifying assumptions (time-invariant transcription/splicing/degradation rates, single regulatory regime per gene) can make RNA velocity unrepresentative of true dynamics
- Faulty velocity estimation yields spurious root/terminal states, e.g. on the erythroid gastrulation dataset and a simulated dataset with time-dependent degradation rates
- Authors do not recommend Cytopath trajectory inference when root and terminal cell states are not clearly identifiable; correctness of velocity cannot always be verified even with marker-based information

## Evidence

- [results] Qiu et al.13 generated a dataset of mouse cortical neurons stimulated for durations ranging from 0–120 min: "Qiu et al.13 generated a dataset of mouse cortical neurons stimulated for durations ranging from 0–120 min"
- [discussion] correlation of pseudotime with stimulation duration: "we investigate in more detail the correlation of pseudotime inferred by Cytopath with stimulation duration for the neuronal activation dataset"
- [intro] internal clock assessment: "We also assess the ability of pseudotime estimated by Cytopath to represent the biological process time, also referred to as the “internal clock” of a cell."
- [discussion] robustness thresholds: "We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown)."
- [readme] Cytopath Markov chain from RNA velocity: "Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model; simulation of this model yields an ensemble of possible differentiation trajectories"
