---
name: gaussian-noise-perturbation-of-scrnaseq-data
description: Use when you need to quantify the noise robustness of a single-cell trajectory
  inference or alignment pipeline by sweeping Gaussian noise levels on synthetic scRNA-seq
  data and recording performance-metric distributions per noise level.
when_to_use_negative:
- The input is authentic scRNA-seq data (e.g., paul15, setty19, velten17) rather than
  synthetic trajectory pairs — this sweep is designed for synthetic data where ground-truth
  trajectory structure is known.
- You need to compare trajectory data containing cycles or disconnected graphs — CAPITAL
  in its present form can deal only with trajectory trees, so downstream metric computation
  would not be valid.
- Datasets have very different global topological structures — the article reports
  alignment accuracy of CAPITAL degraded for such datasets, so a noise-robustness
  curve would not isolate noise effects.
tools:
- name: CAPITAL
  role: Implements the trajectory-tree inference (preprocessing, trajectory_tree,
    dpt, tree_alignment, dtw) whose alignment metrics are evaluated under the Gaussian
    noise sweep
  repo: https://github.com/ykat0/capital
- name: Python 3.8
  role: Runtime environment for the conda environment in which CAPITAL is installed
    and the noise perturbation is executed
- name: conda
  role: Environment manager used to create the isolated CAPITAL environment with graphviz
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:eee5cb0af643a1c4413546cbe9480ee4418fa16ef017cf3afbc26c5de72b05c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - Python
  - Python 3.8
  - conda
  - graphviz
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# gaussian-noise-perturbation-of-scrnaseq-data

## Summary

A robustness-benchmarking procedure that perturbs synthetic single-cell trajectory datasets with independent Gaussian noise across increasing standard deviations and scores how a trajectory-alignment method (CAPITAL) degrades via normalized alignment distance and ATC score.

## When to use

Use when you need to quantify the noise robustness of a single-cell trajectory inference or alignment pipeline by sweeping Gaussian noise levels on synthetic scRNA-seq data and recording performance-metric distributions per noise level.

## When NOT to use

- The input is authentic scRNA-seq data (e.g., paul15, setty19, velten17) rather than synthetic trajectory pairs — this sweep is designed for synthetic data where ground-truth trajectory structure is known.
- You need to compare trajectory data containing cycles or disconnected graphs — CAPITAL in its present form can deal only with trajectory trees, so downstream metric computation would not be valid.
- Datasets have very different global topological structures — the article reports alignment accuracy of CAPITAL degraded for such datasets, so a noise-robustness curve would not isolate noise effects.

## Inputs

- Synthetic single-cell trajectory pairs from capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2 (6,800 pairs)
- Noise sweep specification: Gaussian standard deviations 0 to >= 5.0, 100 repeats per level
- Python 3.8/3.9 conda environment with graphviz and CAPITAL (ver. 1.1.2) installed from PyPI

## Outputs

- Per-alignment normalized alignment distance values for each noise level and repeat
- Per-alignment ATC (alignment tree comparison) scores for each noise level and repeat
- Aggregated noise-level score distributions written to noise_robustness_results.csv
- Noise-robustness curves analogous to Fig. 3a-b of the CAPITAL paper

## How to apply

Generate synthetic trajectory pairs with the capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2 loaders inside a Python 3.8/3.9 conda environment with graphviz, installing CAPITAL via 'pip install capital'. For each pair, add independent Gaussian noise at standard deviations from 0 through 5.0 and beyond, repeating each noise level 100 times (the paper's sweep totals 95,200 perturbed pairs from 6,800 synthetic pairs). For each perturbed pair run the full CAPITAL pipeline: capital.tl.preprocessing, capital.tl.trajectory_tree, capital.tl.dpt for pseudotime, then capital.tl.tree_alignment and capital.tl.dtw. Record the normalized alignment distance and the ATC score per alignment, aggregate distributions by noise level into noise_robustness_results.csv, and judge the method robust if metrics remain stable below SD 3.0; the article reports both metrics deteriorate for SD >= 3.0 as the cell space becomes shattered.

## Related tools

- **CAPITAL** (Implements the trajectory-tree inference (preprocessing, trajectory_tree, dpt, tree_alignment, dtw) whose alignment metrics are evaluated under the Gaussian noise sweep) — https://github.com/ykat0/capital
- **Python 3.8** (Runtime environment for the conda environment in which CAPITAL is installed and the noise perturbation is executed)
- **conda** (Environment manager used to create the isolated CAPITAL environment with graphviz)

## Examples

```
conda create -n capital python=3.8 graphviz; conda activate capital; pip install capital; from capital.dataset import synthetic_dataset1, synthetic_dataset2
```

## Evaluation signals

- Alignment quality remains stable for Gaussian noise SD < 3.0; both normalized alignment distance and ATC score deteriorate at SD >= 3.0, matching Fig. 3a-b.
- Total number of perturbed alignments matches the design: 6,800 synthetic pairs x noise levels x 100 repeats = 95,200 alignments.
- noise_robustness_results.csv contains aggregated score distributions keyed by noise SD with no missing noise levels.
- Each perturbed pair passes through the full CAPITAL chain (preprocessing -> trajectory_tree -> dpt -> tree_alignment -> dtw) without pipeline errors.
- Results should be reproducible via the paper's Code Ocean compute capsule.

## Limitations

- Robustness only holds up to Gaussian SD 3.0; at SD >= 3.0 the cell space begins to be shattered and both metrics deteriorate.
- Findings are limited to trajectory trees — CAPITAL cannot handle complex graphs such as cycles or disconnected graphs.
- Alignment accuracy degrades for datasets with different global topological structure, so noise curves may not transfer to topologically dissimilar pairs.
- Part of the CAPITAL algorithm computing a trajectory as a minimum spanning tree may not be the best method.

## Evidence

- [abstract] Computational tests on synthetic datasets and authentic bone marrow cells datasets indicate that CAPITAL has achieved accurate and robust alignments of trajectory trees: "Computational tests on synthetic datasets and authentic bone marrow cells datasets indicate that CAPITAL has achieved accurate and robust alignments of trajectory trees"
- [methods] Environment setup via conda with graphviz: "conda create -n capital python=3.8 graphviz"
- [results] Findings from noise sweep: [evidence span withheld — anchor status: not_found]
- [readme] Reproducibility via Code Ocean capsule: [evidence span withheld — anchor status: external_doc]
