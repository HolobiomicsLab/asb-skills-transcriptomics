---
name: robustness-benchmark-visualization
description: 'Use when stress-testing a trajectory-alignment method''s robustness:
  synthetic single-cell trajectory pairs are available and the goal is to measure
  how alignment distance and conservation scores change as noise standard deviation
  increases.'
when_to_use_negative:
- The input datasets have different global topological structures — the paper reports
  CAPITAL's alignment accuracy degraded in that benchmarking setting, so noise-robustness
  curves would not transfer.
- The trajectory data contain complex graphs such as cycles or disconnected graphs
  rather than trees, which CAPITAL in its present form cannot handle.
- The goal is data integration of cell subpopulations (e.g., Scanorama/Seurat-style
  anchor transformation) rather than scoring trajectory-tree alignment robustness.
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: CAPITAL
  role: Provides preprocessing, trajectory_tree inference, dpt pseudotime, tree_alignment,
    and dtw used to score each noisy trajectory pair, plus the synthetic dataset loaders
  repo: github.com/ykat0/capital
- name: Python 3.8
  role: Implementation environment in which CAPITAL is installed and the perturbation-aggregation
    benchmark script is run
- name: conda
  role: Creates the isolated 'capital' environment with graphviz needed by CAPITAL's
    tree rendering
- name: graphviz
  role: Dependency installed with the conda environment to support CAPITAL's trajectory-tree
    structures
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:108f3246a83d28f9904fdd631f23502bc45a510755fcd4ed655bd73018c14cd6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAPITAL
  - Python
  - Python 3.8
  - conda
  - graphviz
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# robustness-benchmark-visualization

## Summary

A benchmarking skill that quantifies how CAPITAL's trajectory-tree alignment quality degrades under graded Gaussian noise on synthetic single-cell trajectory pairs, using the normalized alignment distance and ATC score as robustness metrics (Fig. 3a-b).

## When to use

Use when stress-testing a trajectory-alignment method's robustness: synthetic single-cell trajectory pairs are available and the goal is to measure how alignment distance and conservation scores change as noise standard deviation increases.

## When NOT to use

- The input datasets have different global topological structures — the paper reports CAPITAL's alignment accuracy degraded in that benchmarking setting, so noise-robustness curves would not transfer.
- The trajectory data contain complex graphs such as cycles or disconnected graphs rather than trees, which CAPITAL in its present form cannot handle.
- The goal is data integration of cell subpopulations (e.g., Scanorama/Seurat-style anchor transformation) rather than scoring trajectory-tree alignment robustness.

## Inputs

- 6,800 synthetic single-cell trajectory pairs from capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2
- Gaussian noise levels (standard deviations 0 through 5.0 and beyond), 100 repeats per level

## Outputs

- noise_robustness_results.csv with normalized alignment distance and ATC score distributions aggregated by noise standard deviation
- Noise-robustness curves of normalized alignment distance and ATC score (Fig. 3a-b analogue)

## How to apply

Create a Python 3.8 conda environment with graphviz and install CAPITAL via 'pip install capital'. Load the 6,800 synthetic trajectory pairs with capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2, then perturb each pair with independent Gaussian noise at standard deviations 0 through 5.0 and beyond, repeating each noise level 100 times to yield 95,200 perturbed pairs. For each perturbed pair run capital.tl.preprocessing, infer the trajectory tree with capital.tl.trajectory_tree, compute pseudotime with capital.tl.dpt, and align the trees with capital.tl.tree_alignment and capital.tl.dtw. Record the normalized alignment distance and the ATC score per alignment, aggregate distributions by noise level into noise_robustness_results.csv, and inspect whether both metrics deteriorate once the cell space begins to shatter at standard deviation 3.0 or higher.

## Related tools

- **CAPITAL** (Provides preprocessing, trajectory_tree inference, dpt pseudotime, tree_alignment, and dtw used to score each noisy trajectory pair, plus the synthetic dataset loaders) — github.com/ykat0/capital
- **Python 3.8** (Implementation environment in which CAPITAL is installed and the perturbation-aggregation benchmark script is run)
- **conda** (Creates the isolated 'capital' environment with graphviz needed by CAPITAL's tree rendering)
- **graphviz** (Dependency installed with the conda environment to support CAPITAL's trajectory-tree structures)

## Examples

```
conda create -n capital python=3.8 graphviz && conda activate capital && pip install capital
```

## Evaluation signals

- The results file contains 95,200 alignment records (6,800 pairs x 100 repeats per noise level) aggregated by noise standard deviation.
- Normalized alignment distance and ATC score remain stable for noise SD below 3.0 and deteriorate for SD >= 3.0, when the cell space begins to be shattered.
- Each noisy pair yields a valid trajectory tree from capital.tl.trajectory_tree and a finite alignment distance from capital.tl.tree_alignment plus capital.tl.dtw.
- Aggregated distributions in noise_robustness_results.csv reproduce the monotone degradation pattern of Fig. 3a-b.

## Limitations

- CAPITAL's trajectory computation as a minimum spanning tree may not be the best method, so robustness curves are conditional on the MST-based tree inference.
- Alignment accuracy of CAPITAL degrades for datasets with different global topological structure, limiting generalization of the noise thresholds.
- CAPITAL can deal only with trajectory trees and not complex graphs such as cycles or disconnected graphs; a predictive model handling graphs beyond trees plus graph alignment would be required for complex comparative pseudotime analysis.

## Evidence

- [abstract] Robustness benchmark grounded in CAPITAL's accuracy claim: "Computational tests on synthetic datasets and authentic bone marrow cells datasets indicate that CAPITAL has achieved accurate and robust alignments of trajectory trees"
- [methods] Environment setup used for the benchmark: "conda create -n capital python=3.8 graphviz"
- [discussion] Scope of the tool whose robustness is measured: "CAPITAL is a computational tool for comparing cell-state transition trajectory trees of different but related scRNA-seq datasets"
- [readme] Reproducibility capsule for the robustness experiments: [evidence span withheld — anchor status: external_doc]
