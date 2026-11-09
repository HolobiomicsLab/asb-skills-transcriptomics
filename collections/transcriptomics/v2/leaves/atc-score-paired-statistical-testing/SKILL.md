---
name: atc-score-paired-statistical-testing
description: Use when comparing trajectory-tree alignment quality (ATC scores) of
  CAPITAL against integration baselines and you need paired one-sided Wilcoxon signed-rank
  p-values over many synthetic dataset pairs, e.g. to reproduce the Fig. 4a benchmark.
when_to_use_negative:
- Datasets whose pseudotime trajectories have different global topological structure
  — the article reports CAPITAL's alignment accuracy degraded in this regime, so a
  pooled ATC-score win cannot be assumed
- Trajectory data containing cycles or disconnected graphs — CAPITAL handles only
  trajectory trees (including single lineages), so ATC scores are undefined for such
  topologies
- Score samples that are not paired per dataset pair — the one-sided Wilcoxon signed-rank
  test used here requires matched CAPITAL/baseline scores for the same pair, not independent
  samples
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CAPITAL
  role: Computes trajectory trees (tl.trajectory_tree with tl.dpt pseudotime), tree
    alignments (tl.tree_alignment, tl.dtw), and the ATC alignment-accuracy scores
    that form the paired samples being tested
  repo: https://github.com/ykat0/capital
- name: Scanorama
  role: Data-integration baseline that derives cluster correspondences for each synthetic
    pair, providing the competing ATC score distribution
- name: scVI
  role: Data-integration baseline supplying cluster correspondences for the paired
    ATC-score comparison
- name: Seurat
  role: Data-integration baseline supplying cluster correspondences for the paired
    ATC-score comparison
- name: Python
  role: Implementation environment for CAPITAL and the Wilcoxon signed-rank testing
    over the 2,278 paired scores
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:905f4516610c48a25c0e3e531076863e9518115bbe6f1a43de7a673529ee4c58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - Python
  - Scanorama
  - Seurat
  - conda
  - scVI
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# atc-score-paired-statistical-testing

## Summary

Statistical benchmarking pattern that compares the ATC (alignment-accuracy of trajectory clusters) scores of CAPITAL's tree alignment against data-integration baselines (Scanorama, scVI, Seurat) across the same 2,278 synthetic dataset pairs, then tests paired score differences with one-sided Wilcoxon signed-rank tests.

## When to use

Use when comparing trajectory-tree alignment quality (ATC scores) of CAPITAL against integration baselines and you need paired one-sided Wilcoxon signed-rank p-values over many synthetic dataset pairs, e.g. to reproduce the Fig. 4a benchmark.

## When NOT to use

- Datasets whose pseudotime trajectories have different global topological structure — the article reports CAPITAL's alignment accuracy degraded in this regime, so a pooled ATC-score win cannot be assumed
- Trajectory data containing cycles or disconnected graphs — CAPITAL handles only trajectory trees (including single lineages), so ATC scores are undefined for such topologies
- Score samples that are not paired per dataset pair — the one-sided Wilcoxon signed-rank test used here requires matched CAPITAL/baseline scores for the same pair, not independent samples

## Inputs

- 68 synthetic scRNA-seq datasets loaded via capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2
- All 2,278 synthetic dataset pairs with cluster correspondences derived per method (CAPITAL vs. Scanorama, scVI, Seurat)
- Per-pair ATC score vectors for CAPITAL and each baseline, computed on aligned trajectory trees (tree_alignment + dtw)

## Outputs

- Paired ATC score distributions (2,278 matched scores per method)
- One-sided Wilcoxon signed-rank p-values for CAPITAL vs. Scanorama, scVI, and Seurat (reported target: < 1.00 × 10−226)
- Fig. 4a-style ATC-score comparison plot (e.g. fig4a_atc_score_benchmark.png)

## How to apply

Load the 68 synthetic scRNA-seq datasets via capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2 and enumerate all C(68,2) = 2,278 dataset pairs. For each pair, infer the common minimum-spanning-tree pseudotime trajectory with capital.tl.trajectory_tree (pseudotime via capital.tl.dpt), align the trees with capital.tl.tree_alignment and capital.tl.dtw, and compute ATC alignment-accuracy scores on the aligned trees (genes_similarity_score-style scoring); derive cluster correspondences for baselines using Scanorama, scVI, and Seurat. Because each dataset pair yields one ATC score under CAPITAL and one under each baseline, the score vectors are paired — use a one-sided Wilcoxon signed-rank test (greater alternative) on the within-pair differences for CAPITAL vs. each baseline, keeping pair identity as the blocking factor. The published benchmark reports p-values < 1.00 × 10−226 for CAPITAL vs. Scanorama, scVI, and Seurat on the 2,278 pairs, which is the reproduction target (rendered as Fig. 4a).

## Related tools

- **CAPITAL** (Computes trajectory trees (tl.trajectory_tree with tl.dpt pseudotime), tree alignments (tl.tree_alignment, tl.dtw), and the ATC alignment-accuracy scores that form the paired samples being tested) — https://github.com/ykat0/capital
- **Scanorama** (Data-integration baseline that derives cluster correspondences for each synthetic pair, providing the competing ATC score distribution)
- **scVI** (Data-integration baseline supplying cluster correspondences for the paired ATC-score comparison)
- **Seurat** (Data-integration baseline supplying cluster correspondences for the paired ATC-score comparison)
- **Python** (Implementation environment for CAPITAL and the Wilcoxon signed-rank testing over the 2,278 paired scores)

## Examples

```
from scipy.stats import wilcoxon; stat, p = wilcoxon(atc_capital, atc_scanorama, alternative='greater')  # one-sided test over 2,278 paired ATC scores
```

## Evaluation signals

- Pair count invariant: the tested set must contain exactly 2,278 synthetic dataset pairs (all pairs from the 68 synthetic datasets); any deviation changes the statistics
- Reproduction target: one-sided Wilcoxon signed-rank p-values < 1.00 × 10−226 for CAPITAL vs. Scanorama, scVI, and Seurat
- Pairing check: each dataset pair contributes exactly one ATC score under CAPITAL and one under each baseline; verify score vectors are index-aligned before the signed-rank test
- Score provenance check: ATC scores must come from trees built with tl.trajectory_tree + tl.dpt and aligned with tl.tree_alignment + tl.dtw (genes_similarity_score-style scoring), not raw integration metrics
- Cross-check the rendered figure against the Code Ocean compute capsule that reproduces the paper's Fig. 4a

## Limitations

- Alignment accuracy of CAPITAL for datasets with different global topological structure degraded in the article's benchmarking test, so the synthetic-pair result may not transfer to topologically dissimilar data
- The MST-based trajectory computation (minimum spanning tree from clusters) is acknowledged by the authors as possibly not the best method, which bounds what ATC scores can claim
- CAPITAL deals only with trajectory trees (including single lineages), not complex graphs such as cycles or disconnected graphs, so the paired test cannot be extended to such topologies with this skill

## Evidence

- [results] CAPITAL was statistically significantly better than the data integration approaches, with one-sided Wilcoxon signed-rank p-values < 1.00 × 10−226 for CAPITAL vs. Scanorama, scVI, and Seurat: [evidence span withheld — anchor status: not_found]
- [methods] Compare score distributions with one-sided Wilcoxon signed-rank tests across the 2,278 pairs.: [evidence span withheld — anchor status: not_found]
- [discussion] part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method: "part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method"
- [readme] We also provide a Code Ocean [compute capsule](https://codeocean.com/capsule/5673663/tree/v1) to reproduce our results.: [evidence span withheld — anchor status: external_doc]
