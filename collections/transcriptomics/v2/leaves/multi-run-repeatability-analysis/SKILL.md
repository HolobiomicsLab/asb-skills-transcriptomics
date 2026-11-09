---
name: multi-run-repeatability-analysis
description: Use when a stochastic trajectory or pseudotime method (e.g. Cytopath
  on RNA velocity fields) is rerun across independent runs and stability or median
  performance across runs must be reported, e.g. Pearson correlation with elapsed
  stimulation time (0–120 min).
when_to_use_negative:
- The trajectory method is deterministic (no stochastic simulation), so repeated runs
  would be identical and repeatability analysis is uninformative.
- Root and terminal cell states are not clearly identifiable or the underlying RNA
  velocity estimation is faulty — the article does not recommend performing Cytopath
  trajectory inference in these cases, so run-level correlation summaries would be
  meaningless.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Cytopath
  role: Markov chain simulation-based trajectory inference whose stochasticity motivates
    repeated independent runs; run repeatedly (and as Cytopath-Euclidean ablation)
    to assess pseudotime repeatability
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Estimates the RNA velocity field on ARGs that Cytopath consumes; also provides
    latent-time and simulation baselines in the benchmark
- name: numpy
  role: Computes per-run Pearson correlations between pseudotime and per-cell stimulation
    duration
- name: scipy
  role: Statistical correlation computation alongside numpy for per-run summaries
- name: pandas
  role: Assembles and saves the per-run correlation table (pseudotime_stimulation_correlation.csv)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:d5f6897cdf42a187f78a50de540c7edf2c769453f7246890c59e8e5abdfdbcae
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# multi-run-repeatability-analysis

## Summary

Quantifies run-to-run stability of stochastic trajectory inference by repeating Cytopath's Markov-chain pseudotime inference over multiple independent runs (e.g. 10) and summarizing per-run metrics such as Pearson correlation with the scNT-seq stimulation-duration internal clock.

## When to use

Use when a stochastic trajectory or pseudotime method (e.g. Cytopath on RNA velocity fields) is rerun across independent runs and stability or median performance across runs must be reported, e.g. Pearson correlation with elapsed stimulation time (0–120 min).

## When NOT to use

- The trajectory method is deterministic (no stochastic simulation), so repeated runs would be identical and repeatability analysis is uninformative.
- Root and terminal cell states are not clearly identifiable or the underlying RNA velocity estimation is faulty — the article does not recommend performing Cytopath trajectory inference in these cases, so run-level correlation summaries would be meaningless.

## Inputs

- AnnData (.h5ad) object with per-cell stimulation duration metadata and RNA velocity (scvelo-computed on activity-regulated genes) for the scNT-seq cortical neuron dataset (GEO: GSE141851)
- Per-run Cytopath pseudotime vectors from ≥10 independent runs, optionally including Cytopath-Euclidean ablation runs

## Outputs

- Table of per-run Pearson correlations between inferred pseudotime and stimulation duration (pseudotime_stimulation_correlation.csv)
- Median Pearson correlation over runs for velocity-based vs. Euclidean-ablation variants, used as a repeatability/performance summary

## How to apply

Repeat the full Cytopath Markov-chain simulation and pseudotime assignment pipeline over 10 independent runs on the same velocity-annotated AnnData (here the scNT-seq mouse cortical neuron dataset with ARG-restricted RNA velocity from scvelo). For each run, compute the Pearson correlation (numpy/scipy) between per-cell pseudotime and the known stimulation duration (0–120 min) serving as the internal clock. When including an ablation (Cytopath-Euclidean, without velocity), compute the same per-run correlations and compare the median over the 10 runs between variants. Store the per-run correlation table (e.g. pseudotime_stimulation_correlation.csv) and treat high, monotonic correlation with low cross-run variance as evidence of robust inference; the article reports Cytopath pseudotime is robust across multiple runs (Spearman correlation with known cell-type ordering, Figure 2D–E).

## Related tools

- **Cytopath** (Markov chain simulation-based trajectory inference whose stochasticity motivates repeated independent runs; run repeatedly (and as Cytopath-Euclidean ablation) to assess pseudotime repeatability) — https://github.com/aron0093/cytopath
- **scvelo** (Estimates the RNA velocity field on ARGs that Cytopath consumes; also provides latent-time and simulation baselines in the benchmark)
- **numpy** (Computes per-run Pearson correlations between pseudotime and per-cell stimulation duration)
- **scipy** (Statistical correlation computation alongside numpy for per-run summaries)
- **pandas** (Assembles and saves the per-run correlation table (pseudotime_stimulation_correlation.csv))

## Examples

```
import numpy as np, pandas as pd; cors = [np.corrcoef(run_pseudotime, adata.obs['stim_duration'])[0,1] for run in range(10)]; pd.DataFrame({'pearson_r': cors}).to_csv('pseudotime_stimulation_correlation.csv')
```

## Evaluation signals

- Per-run Pearson correlation of Cytopath pseudotime with stimulation duration should remain high and monotonic across all 10 runs.
- Velocity-based Cytopath should achieve a higher median Pearson correlation over the 10 runs than the non-velocity Cytopath-Euclidean ablation.
- The output CSV contains exactly 10 rows per variant with finite correlation values in [-1, 1].
- Run-level correlations should show low variance (stability), consistent with the article's report that Cytopath pseudotime is high and robust across multiple runs (Figure 2D–E).
- Pseudotime should be robust to root/terminal probability thresholds within [0.85, 1], per the authors' discussion.

## Limitations

- Robustness to root and terminal probability thresholds was reported only as 'data not shown' for a range of [0.85, 1].
- Trajectory inference with Cytopath is not meaningful when the underlying RNA velocity estimation itself is faulty (e.g. spurious root/terminal states in the erythroid gastrulation dataset and time-dependent degradation-rate simulations).
- The authors do not recommend performing trajectory inference when root and terminal cell states are not clearly identifiable, which invalidates run-repeatability correlation summaries in such datasets.

## Evidence

- [results] The Spearman correlation between pseudotime inferred by Cytopath for each trajectory to known ordering of cell types is high (Figure 2D) and robust across multiple runs (Figure 2E).: "The Spearman correlation between pseudotime inferred by Cytopath for each trajectory to known ordering of cell types is high (Figure 2D) and robust across multiple runs (Figure 2E)."
- [results] Qiu et al.13 generated a dataset of mouse cortical neurons stimulated for durations ranging from 0–120 min: "Qiu et al.13 generated a dataset of mouse cortical neurons stimulated for durations ranging from 0–120 min"
- [discussion] We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown).: "We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown)."
- [abstract] Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory.: "Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory."
- [readme] Demonstration of [Cytopath](https://github.com/aron0093/cytopath) on publicly available single cell RNA sequencing datasets: [evidence span withheld — anchor status: external_doc]
