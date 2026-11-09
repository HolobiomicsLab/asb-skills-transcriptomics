---
name: single-cell-trajectory-inference
description: Use when single-cell transcriptomic data from two or more conditions
  must be compared along a dynamic process (e.g., differentiation, EMT), and you need
  to decide whether a common trajectory suffices or conditions differ in topology,
  pseudotime progression, fate selection, or gene expression.
when_to_use_negative:
- Cells are in a steady-state system with no trajectory/continuum — use cluster-based
  differential abundance (e.g., milo, DAseq, MAST/scde) instead, since assessing condition
  imbalance per cluster ignores trajectory structure only when dynamics are absent
- The chosen trajectory inference method cannot accept a specified underlying skeleton
  — topologyTest requires a maintained mapping across condition-label permutations
  and works only with methods like slingshot or TSCAN
- Batch and condition are confounded such that condition-label permutation tests would
  be uninterpretable; the article notes some confounding is inherent and unresolvable
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: condiments
  role: Core package providing topologyTest, progressionTest, fateSelectionTest, and
    imbalance_score for multi-condition trajectory analysis
  repo: github.com/HectorRDB/condiments
- name: slingshot
  role: Infers the common trajectory/skeleton on the low-dimensional embedding and
    supplies pseudotimes and lineage curve weights downstream of the tests
- name: tradeSeq
  role: Fits gene-wise negative-binomial GAMs (fitGAM) on the trajectory and runs
    conditionTest for gene-level condition-specific DE
  repo: statomics.github.io/tradeSeq
- name: milo
  role: Differential abundance benchmarking baseline relying on data-driven local
    neighborhoods over a common low-dimensional representation
- name: DAseq
  role: Differential abundance benchmarking baseline using data-driven local neighborhoods
- name: dyngen
  role: Simulation framework generating multi-lineage trajectory datasets with ground
    truth for benchmarking the tests
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:8720a1cc88f261ee2439f7acf24aab71eb87a99648a92e557804503f5a212f33
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - DAseq
  - R
  - SingleCellExperiment
  - Slingshot
  - TinGa
  - Totem
  - condiments
  - cowplot
  - dplyr
  - dyndimred
  - dyngen
  - ggplot2
  - milo
  - slingshot
  - tidyr
  - tradeSeq
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- trajectory-inference-from-single-cell-data
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-trajectory-inference

## Summary

A three-step condiments-based workflow for inferring and interpreting single-cell developmental trajectories across multiple conditions: testing differential topology, differential progression/fate selection, and gene-level condition-specific expression along lineages.

## When to use

Use when single-cell transcriptomic data from two or more conditions must be compared along a dynamic process (e.g., differentiation, EMT), and you need to decide whether a common trajectory suffices or conditions differ in topology, pseudotime progression, fate selection, or gene expression.

## When NOT to use

- Cells are in a steady-state system with no trajectory/continuum — use cluster-based differential abundance (e.g., milo, DAseq, MAST/scde) instead, since assessing condition imbalance per cluster ignores trajectory structure only when dynamics are absent
- The chosen trajectory inference method cannot accept a specified underlying skeleton — topologyTest requires a maintained mapping across condition-label permutations and works only with methods like slingshot or TSCAN
- Batch and condition are confounded such that condition-label permutation tests would be uninterpretable; the article notes some confounding is inherent and unresolvable

## Inputs

- Integrated, normalized single-cell count matrix with per-cell condition labels (e.g., mock vs TGF-beta)
- Low-dimensional embedding (e.g., reduced-dimension coordinates such as Dim1/Dim2 from PCA or UMAP)
- Cluster labels per cell for slingshot trajectory inference
- Optionally, dyngen-simulated datasets with known topology and ground-truth condition changes for benchmarking

## Outputs

- topologyTest statistics and permutation-based p-value table (e.g., topology_test_results.csv)
- progressionTest global and per-lineage p-value table (e.g., progression_test_pvalues.csv)
- fateSelectionTest results on slingshot curve weights
- Condition-specific DE gene list at FDR 5% from tradeSeq conditionTest (e.g., 1,993 genes in the EMT study)
- Performance metrics (TNR, TPR, PPV, NPV, F1) when benchmarked against dyngen simulation ground truth

## How to apply

Integrate and normalize count matrices across conditions, then infer a common trajectory with slingshot on a shared low-dimensional representation (e.g., Dim1/Dim2 of the condiments toy_dataset) using cluster labels: `sds <- slingshot(rd, df$cl)`. Step 1: run `topologyTest` with the condition vector, permuting condition labels (default 100 repeats) to test the null of identical trajectories; this requires TI methods that accept a fixed skeleton (slingshot, TSCAN) and includes a minimum-magnitude requirement so only biologically relevant topological shifts are significant. Step 2: extract per-lineage pseudotimes with `slingPseudotime` and run `progressionTest` (default custom Kolmogorov-Smirnov test, `global = TRUE, lineages = TRUE`) for differential progression, and `fateSelectionTest` (classifier test on curve weights) for differential fate selection. Step 3: fit gene-wise negative-binomial GAMs with `tradeSeq::fitGAM(counts, sds, conditions)` and run `conditionTest` to call condition-specific DE genes, then optionally rank the list for `fgsea` enrichment; adjust p-values with Benjamini-Hochberg at 5% FDR.

## Related tools

- **condiments** (Core package providing topologyTest, progressionTest, fateSelectionTest, and imbalance_score for multi-condition trajectory analysis) — github.com/HectorRDB/condiments
- **slingshot** (Infers the common trajectory/skeleton on the low-dimensional embedding and supplies pseudotimes and lineage curve weights downstream of the tests)
- **tradeSeq** (Fits gene-wise negative-binomial GAMs (fitGAM) on the trajectory and runs conditionTest for gene-level condition-specific DE) — statomics.github.io/tradeSeq
- **milo** (Differential abundance benchmarking baseline relying on data-driven local neighborhoods over a common low-dimensional representation)
- **DAseq** (Differential abundance benchmarking baseline using data-driven local neighborhoods)
- **dyngen** (Simulation framework generating multi-lineage trajectory datasets with ground truth for benchmarking the tests)

## Examples

```
library(condiments); sds <- slingshot(rd, df$cl); topologyTest(sds, conditions = df$conditions); progressionTest(sds, conditions = df$conditions, global = TRUE, lineages = TRUE)
```

## Evaluation signals

- topologyTest p-value is permutation-based over 100 repeats; a non-significant result (e.g., p = 0.38 in the EMT study) validates that a common slingshot trajectory is adequate before Steps 2–3
- In the EMT case study, the progressionTest global p-value should be below 2.2e-16, and per-lineage results should localize the difference (toy dataset: lineage 2 significant, lineage 1 not)
- After Benjamini-Hochberg correction at 5% FDR, conditionTest should recover biologically coherent gene sets; in the EMT study, fgsea on the ranked list recovers adhesion and locomotion enrichment
- In dyngen simulations (topologies T1–T4 with 2, 3, 2, and 5 lineages), the compiled performance table must contain TNR, TPR, PPV, NPV, and F1 for every method, with progressionTest expected to outperform fateSelectionTest, DAseq, and milo
- Output CSV tables must contain complete test statistics and p-values (global and per-lineage) matching the paper's Table 2 / Fig. 2e-style annotations

## Limitations

- Pseudotime and curve-weight estimates are reused (double-dipping) in downstream tests; the dependence is ignored, as in other differential abundance methods
- P-values are treated as numerical summaries rather than given strong probabilistic interpretations, which would involve hard-to-verify assumptions
- Some batch-condition confounding is inherent to the problem and cannot be resolved by the method
- No independent benchmarking of the workflow had been performed; the authors call for it, and real-world benchmarking case studies were still becoming available
- topologyTest only works with trajectory inference methods that allow specification of an underlying skeleton structure (e.g., slingshot, TSCAN)

## Evidence

- [methods] topologyTest permutes condition labels: "The __topologyTest__ assess the quality of the common trajectory inference"
- [methods] progression test definition: "To test for differential progression, we use the __progressionTest__."
- [intro] three-step workflow: "In Step 2, it tests for global differences between conditions, both along lineages – differential progression – and between lineages – differential fate selection."
- [methods] GAM fitting invocation: "sce <- fitGAM(counts = counts, sds = sds, conditions = df$conditions)"
- [discussion] double-dipping caveat: "Here, we ignore this dependence, as is the case in other differential abundance methods"
- [readme] README installation: [evidence span withheld — anchor status: external_doc]
