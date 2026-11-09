---
name: simulated-benchmark-validation
description: Use when validating that a trajectory-suitability or trajectory-detection
  step correctly discriminates between simulated scRNA-seq datasets with no true trajectory
  (homogeneous cells or disconnected clusters) and datasets with a known trajectory
  signal.
when_to_use_negative:
- The input dataset has a known biological trajectory and the goal is to score embeddings
  or select processing choices — Escort's Step 1 flags are designed for detecting
  trajectory-free data, not for ranking embeddings (use Escort's benchmark scoring
  instead).
- The trajectory inference method does not use two-dimensional representations — Escort
  presently only considers trajectory inference approaches that use two-dimensional
  representations, so the Step 1 flags cannot be assumed transferable.
- The reference data is not an scRNA-seq count matrix suitable for Scaffold simulation
  (e.g. bulk RNA-seq, spatial transcriptomics), since estimateScaffoldParameters()/simulateScaffold()
  operate on single-cell references.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: Escort
  role: Runs the Step 1 suitability assessment (HD_DCClusterscheck and step1_testHomogeneous)
    on the simulated datasets and emits the suitability flags being validated
  repo: https://github.com/xiaorudong/Escort
- name: Scaffold
  role: Generates the simulated scRNA-seq benchmark datasets with known ground-truth
    trajectory content (0% vs 20% dynamic genes) from the human pancreas reference
- name: R
  role: Execution environment in which Scaffold simulations, Escort Step 1 checks,
    and result compilation are carried out
  repo: https://github.com/xiaorudong/Escort
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:92e85e73e4ae69452524f8b452462b843f231ae171ad7154bd6d7dbbc3fbeabb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaorudong/Escort
  repo_url: https://github.com/xiaorudong/Escort
  tools:
  - Escort
  - R
  - Scaffold
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# simulated-benchmark-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validate a trajectory-suitability assessment (Escort Step 1) by generating simulated scRNA-seq datasets with known ground truth — trajectory-free (0% dynamic genes) and trajectory-bearing (20% dynamic genes) — using the Scaffold v0.2.0 R package on a human pancreas reference, then confirming that trajectory-free simulations are flagged as unsuitable and linear-trajectory simulations are not falsely flagged.

## When to use

Use when validating that a trajectory-suitability or trajectory-detection step correctly discriminates between simulated scRNA-seq datasets with no true trajectory (homogeneous cells or disconnected clusters) and datasets with a known trajectory signal.

## When NOT to use

- The input dataset has a known biological trajectory and the goal is to score embeddings or select processing choices — Escort's Step 1 flags are designed for detecting trajectory-free data, not for ranking embeddings (use Escort's benchmark scoring instead).
- The trajectory inference method does not use two-dimensional representations — Escort presently only considers trajectory inference approaches that use two-dimensional representations, so the Step 1 flags cannot be assumed transferable.
- The reference data is not an scRNA-seq count matrix suitable for Scaffold simulation (e.g. bulk RNA-seq, spatial transcriptomics), since estimateScaffoldParameters()/simulateScaffold() operate on single-cell references.

## Inputs

- human pancreas single-cell RNA-seq reference dataset (used as Scaffold simulation reference)
- Scaffold v0.2.0 simulation parameters from estimateScaffoldParameters()
- Escort R package (Step 1 functions: HD_DCClusterscheck, step1_testHomogeneous)

## Outputs

- Two simulated scRNA-seq datasets: one with 0% dynamic genes (homogeneous cells, no trajectory) and one with 20% dynamic genes (linear trajectory)
- step1_suitability_results.csv compiling Escort Step 1 suitability flags for both simulated datasets
- Per-cluster Jaccard index connectivity scores (within- and between-cluster) from the HD_DCClusterscheck
- Spearman correlation p-values for the first principal component vs. top 100 highly variable genes from step1_testHomogeneous

## How to apply

Generate two simulated scRNA-seq datasets from the human pancreas single-cell reference using Scaffold v0.2.0: first configure parameters with estimateScaffoldParameters(), then run simulateScaffold() — one dataset with no dynamically expressed genes (homogeneous cells) and one with 20% dynamic genes forming a linear trajectory. Load each simulation into R with the Escort package and run the Step 1 cluster-disconnectedness check (HD_DCClusterscheck): cluster cells with scLCA, build the Manhattan distance matrix, compute within- and between-cluster Jaccard index scores, and mark two clusters as connected only if more than 10 cells per cluster meet the 0.3 Jaccard cutoff. Then run the homogeneity test (step1_testHomogeneous): compute Spearman correlations between the first principal component and a random subset of the top 100 highly variable genes, assessed with a 20,000-iteration permutation test with false discovery rate correction, flagging homogeneity when significant correlations (p < 0.05) exceed 46% of genes. Compile the Step 1 suitability flags into step1_suitability_results.csv. The rationale is that simulations provide known ground truth (0% vs 20% dynamic genes), so correct behavior is unambiguous: every trajectory-free simulation must be flagged as unsuitable while the linear-trajectory simulation must not be falsely flagged.

## Related tools

- **Escort** (Runs the Step 1 suitability assessment (HD_DCClusterscheck and step1_testHomogeneous) on the simulated datasets and emits the suitability flags being validated) — https://github.com/xiaorudong/Escort
- **Scaffold** (Generates the simulated scRNA-seq benchmark datasets with known ground-truth trajectory content (0% vs 20% dynamic genes) from the human pancreas reference)
- **R** (Execution environment in which Scaffold simulations, Escort Step 1 checks, and result compilation are carried out) — https://github.com/xiaorudong/Escort

## Examples

```
if (!requireNamespace("remotes", quietly=TRUE)) install.packages("remotes"); remotes::install_github("xiaorudong/Escort"); library(Escort)
```

## Evaluation signals

- Every simulated dataset with no true trajectory (0% dynamic genes / homogeneous cells) is detected and flagged as unsuitable for trajectory fitting by Escort Step 1.
- The linear-trajectory simulation (20% dynamic genes) is NOT falsely flagged as unsuitable — no false positives on trajectory-bearing ground truth.
- In HD_DCClusterscheck, two clusters are only marked connected when more than 10 cells per cluster meet the 0.3 Jaccard cutoff, and the Manhattan distance matrix was built from scLCA cluster assignments.
- In step1_testHomogeneous, homogeneity is flagged only when significant Spearman correlations (p < 0.05) exceed 46% of the top 100 highly variable genes under the 20,000-iteration permutation test with FDR correction.
- Flags for both datasets are compiled into step1_suitability_results.csv with one row per simulated dataset.

## Limitations

- Escort only considers trajectory inference approaches that use two-dimensional representations; suitability flags may not generalize to other representation choices.
- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated, even when a trajectory exists.
- Validation is grounded in Scaffold simulations parameterized from a single human pancreas reference (and six publicly available simulated datasets), so detection thresholds (0.3 Jaccard cutoff, 46% significant correlations) may behave differently on real, noisier datasets.
- Quality control steps (removing cells with extreme outliers in unique genes, total molecule counts, mitochondrial read percentage) and normalization choices upstream may significantly impact Escort's embedding evaluations and thus the Step 1 flags.

## Evidence

- [other] every simulated dataset with no true trajectory was detected and flagged by Escort's first step, and these flagged datasets produced even less accurate trajectories than non-recommended embeddings: [evidence span withheld — anchor status: not_found]
- [other] Generate two simulated scRNA-seq datasets with the Scaffold v0.2.0 R package using the human pancreas single-cell RNA-seq reference: [evidence span withheld — anchor status: not_found]
