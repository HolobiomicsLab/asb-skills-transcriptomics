---
name: gene-expression-program-recovery-benchmarking
description: Decision support for choosing between ICA- and NMF-based decomposition
  of scRNA-seq matrices, or for validating a program-recovery pipeline by matching
  recovered components to known embedded gene expression programs (GEPs).
when_to_use_negative:
- The goal is annotating real tumor scRNA-seq data against pretrained meta-components
  rather than benchmarking decomposition method choice — use the MetaTiME annotator
  projection instead.
- Input data has no known ground-truth gene expression programs embedded, so recovery
  scores cannot be computed against true GEPs.
- Comparison targets are integration/dataset-alignment algorithms (Harmony, scVI,
  scArches) rather than single-dataset matrix decomposition into independent components.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scsim
  role: Simulator used to generate synthetic single-cell count matrices with pre-embedded
    transcriptional signatures for the recovery benchmark
- name: MetaTiME
  role: Provides the ICA-based meta-component recovery pipeline whose program recovery
    is benchmarked against NMF
  repo: https://github.com/yi-zhang/MetaTiME
- name: scikit-learn
  role: Source of the NMF-based decomposition pipeline used as the comparison method
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:c53b2d8d59ab90679b739d60b4b3f241a617a86ee0d26c5de19203b6d10b9469
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  - scikit-learn
  - scsim
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gene-expression-program-recovery-benchmarking

## Summary

Benchmark how well dimensional-reduction methods (ICA vs NMF) recover known gene expression programs embedded in simulated multi-cohort single-cell RNA-seq data, as used to justify MetaTiME's choice of ICA for learning meta-components (MeCs).

## When to use

Decision support for choosing between ICA- and NMF-based decomposition of scRNA-seq matrices, or for validating a program-recovery pipeline by matching recovered components to known embedded gene expression programs (GEPs).

## When NOT to use

- The goal is annotating real tumor scRNA-seq data against pretrained meta-components rather than benchmarking decomposition method choice — use the MetaTiME annotator projection instead.
- Input data has no known ground-truth gene expression programs embedded, so recovery scores cannot be computed against true GEPs.
- Comparison targets are integration/dataset-alignment algorithms (Harmony, scVI, scArches) rather than single-dataset matrix decomposition into independent components.

## Inputs

- Simulated single-cell count matrices from scsim with pre-embedded gene expression programs (e.g. 20 cohorts × 14 GEPs: 13 cell-type-specific + 1 signaling)
- Configuration of cohort-size settings to pool (5, 10, 20 datasets)
- Ground-truth list of embedded GEPs for component matching

## Outputs

- simulation_benchmark_results.csv comparing ICA vs NMF recovery scores across cohort sizes 5/10/20
- Per-program recovery scores from matching recovered components to the 14 known GEPs per simulation

## How to apply

Generate synthetic single-cell count matrices with scsim, embedding a known set of transcriptional signatures — in the MetaTiME benchmark, 20 datasets each carrying 14 GEPs (13 cell-type-specific programs plus 1 signaling program). Pool synthetic datasets at multiple cohort sizes (5, 10, and 20 datasets) and run each decomposition pipeline — the ICA-based meta-component recovery pipeline used by MetaTiME (which integrates tumor single-cell datasets using ICA to optimize information independence among components) and an NMF-based pipeline (e.g. scikit-learn) — on the pooled matrices. For each run, match recovered components back to the 14 known GEPs and compute per-program recovery scores. Aggregate scores into a comparison table over method (ICA vs NMF) and cohort size (5/10/20), and write simulation_benchmark_results.csv. The expected outcome is that ICA recovers true GEPs slightly better than NMF and that recovery improves as pooled cohorts increase from 5 to 10 to 20, which grounds the rationale that meta-components must reflect independent components of gene expression variation.

## Related tools

- **scsim** (Simulator used to generate synthetic single-cell count matrices with pre-embedded transcriptional signatures for the recovery benchmark)
- **MetaTiME** (Provides the ICA-based meta-component recovery pipeline whose program recovery is benchmarked against NMF) — https://github.com/yi-zhang/MetaTiME
- **scikit-learn** (Source of the NMF-based decomposition pipeline used as the comparison method)

## Examples

```
python run_benchmark.py --simulator scsim --n_cohorts 20 --n_geps 14 --methods ica,nmf --cohort_sizes 5,10,20 --out simulation_benchmark_results.csv
```

## Evaluation signals

- ICA recovers the 14 true GEPs slightly better than NMF across matched cohort-size settings
- Recovery scores improve monotonically as the number of pooled cohorts increases from 5 to 10 to 20
- Each recovered component can be matched back to one of the 14 known embedded GEPs (13 cell-type-specific + 1 signaling) with a computed recovery score
- simulation_benchmark_results.csv contains a method × cohort-size grid covering ICA/NMF × 5/10/20 with non-empty per-program scores
- All 20 synthetic cohorts are generated with exactly 14 embedded gene expression programs each before any pooling or decomposition

## Limitations

- The benchmark conclusion (ICA slightly better than NMF) holds in simulations with pre-embedded signatures; the margin over NMF is slight, not decisive.
- Recovery depends on pooling enough cohorts: with fewer pooled datasets (e.g. 5) recovery is worse, so single-dataset benchmarks may understate performance.
- Simulation-based validation may not capture all structure of real tumor immune microenvironment scRNA-seq data.

## Evidence

- [discussion] ICA vs NMF program-recovery rationale: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [intro] Why MeCs reflect independent expression components: [evidence span withheld — anchor status: not_found]
- [abstract] Scope of learned components across cancer types: "Using millions of TME single cells, MetaTiME learns meta-components that encode independent components of gene expression observed across cancer types."
- [readme] Pretrained components deliverable: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment."
