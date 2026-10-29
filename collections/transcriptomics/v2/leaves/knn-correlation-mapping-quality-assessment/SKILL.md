---
name: knn-correlation-mapping-quality-assessment
description: Use when a query dataset has been mapped into a reference embedding (e.g.,
  with Symphony mapQuery) and you need a per-cell metric quantifying whether the mapped
  coordinates reproduce the local neighborhood structure of a de novo integrated gold
  standard.
when_to_use_negative:
- When the query contains a genuinely novel cell population absent from the reference
  — the article notes mapping metrics may be less informative when the novel population
  is very similar to an existing reference population, and Symphony does not supplant
  de novo query-only analysis for novel cell type discovery.
- When the reference lacks comprehensive cell type coverage (Condition I), since equivalence
  with de novo Harmony integration is then not expected.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Symphony
  role: Produces the mapped query embedding whose neighborhood fidelity is being assessed
    via k-NN correlation
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Provides the de novo integrated gold-standard embedding of all datasets against
    which mapped-query neighborhoods are compared
  repo: https://github.com/immunogenomics/singlecellmethods
- name: singlecellmethods
  role: Hosts helper methods (e.g., calcknncorr-style evaluation utilities) used in
    the benchmark
  repo: https://github.com/immunogenomics/singlecellmethods
- name: LISI
  role: Complementary technology-mixing metric (Local Inverse Simpson's Index) used
    alongside k-NN correlation to evaluate mapping fidelity
  repo: https://github.com/immunogenomics/LISI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:41468a9baba2f53f0c6bf3270aa0c109aa11531e6500df5274e6cae403567db0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - LISI
  - Symphony
  - dplyr
  - irlba
  - singlecellmethods
  - uwot
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# knn-correlation-mapping-quality-assessment

## Summary

Assess the fidelity of query-cell reference mapping (e.g., Symphony-mapped PBMCs) by computing per-cell k-NN correlation between the mapped query embedding and a gold-standard de novo reference embedding (calcknncorr at k=500), and report the fraction of query cells with correlation > 0.4 against a PCA-only baseline.

## When to use

Use when a query dataset has been mapped into a reference embedding (e.g., with Symphony mapQuery) and you need a per-cell metric quantifying whether the mapped coordinates reproduce the local neighborhood structure of a de novo integrated gold standard.

## When NOT to use

- When the query contains a genuinely novel cell population absent from the reference — the article notes mapping metrics may be less informative when the novel population is very similar to an existing reference population, and Symphony does not supplant de novo query-only analysis for novel cell type discovery.
- When the reference lacks comprehensive cell type coverage (Condition I), since equivalence with de novo Harmony integration is then not expected.

## Inputs

- Symphony-mapped query embedding (from mapQuery on held-out query expression, e.g., pbmcs_exprs_small/pbmcs_meta_small with one 10x technology held out)
- Gold-standard de novo Harmony-integrated embedding of all three 10x PBMC datasets (3'v1, 3'v2, 5')
- PCA-only baseline embedding of the query (uncorrected)
- Parameters: k = 500 neighbors, correlation threshold > 0.4

## Outputs

- Per-cell k-NN correlation values between mapped query and de novo reference embeddings
- Fraction of query cells with k-NN correlation > 0.4 (e.g., 87.7% Symphony vs 50.2% PCA-only)
- Rows appended to a benchmark metrics file such as pbmc_mapping_benchmark_metrics.csv alongside LISI and classification accuracy/F1

## How to apply

After mapping the held-out query (e.g., the 10x 5' PBMC dataset) onto a Symphony reference built with vargenes_vst topn=1000, 20 PCs, and K=100 Harmony clusters, compute per-cell k-NN correlation between the Symphony-mapped query embedding and the de novo Harmony-integrated reference embedding using calcknncorr at k=500 neighbors. Record the fraction of query cells whose correlation exceeds 0.4, and compare this against a PCA-only baseline (no Harmony/mapping correction) to isolate the benefit of the mapping. In the leave-one-technology-out PBMC benchmark, Symphony mapping achieved 87.7% of cells above the 0.4 threshold versus 50.2% for the PCA-only baseline. Combine this with technology LISI (Symphony ~2.12 vs de novo Harmony ~2.15) and 5-NN label-transfer accuracy (~97.5%, median F1 ~0.97) for a complete mapping-fidelity assessment, and compile all folds into a metrics CSV such as pbmc_mapping_benchmark_metrics.csv.

## Related tools

- **Symphony** (Produces the mapped query embedding whose neighborhood fidelity is being assessed via k-NN correlation) — https://github.com/immunogenomics/symphony
- **Harmony** (Provides the de novo integrated gold-standard embedding of all datasets against which mapped-query neighborhoods are compared) — https://github.com/immunogenomics/singlecellmethods
- **singlecellmethods** (Hosts helper methods (e.g., calcknncorr-style evaluation utilities) used in the benchmark) — https://github.com/immunogenomics/singlecellmethods
- **LISI** (Complementary technology-mixing metric (Local Inverse Simpson's Index) used alongside k-NN correlation to evaluate mapping fidelity) — https://github.com/immunogenomics/LISI

## Examples

```
knn_corr <- calcknncorr(query$Z, harmonized_ref_Z, k = 500); frac_good <- mean(knn_corr > 0.4)
```

## Evaluation signals

- Majority of query cells exceed k-NN correlation 0.4 at k = 500 (reported 87.7% vs 50.2% PCA-only baseline)
- Technology LISI of the mapped embedding closely matches de novo Harmony (~2.12 vs ~2.15)
- 5-NN label transfer reaches ~97.5% accuracy with median per-class F1 ~0.97
- Metric improvements over the PCA-only baseline are consistent across all three leave-one-technology-out folds (3'v1, 3'v2, 5' held out)
- All folds' LISI, k-NN-correlation fractions, and accuracy/F1 values compile cleanly into pbmc_mapping_benchmark_metrics.csv

## Limitations

- Metrics may be less informative when a novel population in the query is very similar to an existing reference population; identifying novel cell types that failed to map remains an unsolved direction.
- Equivalence with de novo integration requires comprehensive cell type coverage in the reference (Condition I), which limits flexibility.
- The k-NN correlation threshold (0.4) and neighborhood size (k = 500) are benchmark-specific choices tied to the 10x PBMC leave-one-technology-out design and may need re-tuning for other tissues or technologies.

## Evidence

- [methods] k-NN correlation threshold and baseline comparison: [evidence span withheld — anchor status: not_found]
- [methods] Reported benchmark fractions: [evidence span withheld — anchor status: not_found]
- [discussion] Novel population caveat: [evidence span withheld — anchor status: not_found]
- [discussion] Novel discovery limitation: "Symphony does not entirely supplant the need for users interested in novel cell type discovery to conduct de novo analyses of the query alone."
