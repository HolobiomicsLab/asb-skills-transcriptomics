---
name: statistical-results-table-generation
description: Use when you have a per-dataset performance metric matrix (e.g., ARI)
  for a proposed method plus baselines over a fixed benchmark and must produce a paired
  significance-test table.
when_to_use_negative:
- The score matrix is incomplete — any missing ARI values per dataset break the paired
  test design across the same 15 datasets.
- You need unpaired or independent-sample testing; the benchmark uses matched datasets
  per method, so paired=TRUE is required.
- The comparison vectors come from a single method's output rather than a method-vs-method
  benchmark matrix.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Runtime used for stats::wilcox.test paired signed-rank testing and CSV table
    assembly (R version 4.0.5 in the paper)
  repo: https://github.com/LingyiC/CASCC_benchmark
- name: CASCC
  role: Method under test whose ARI vector is compared against each baseline
  repo: https://github.com/LingyiC/CASCC
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae283
    title: 'CASCC: a co-expression-assisted single-cell RNA-seq data clustering method'
  source_content_hash: sha256:5c347b54b841aa66630f94130952cb986808c94a5a5a8f63af362831d5c445f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LingyiC/CASCC_benchmark
  tools:
  - CASCC
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# statistical-results-table-generation

## Summary

Generates a consolidated statistical significance table by running paired Wilcoxon signed-rank tests in R comparing CASCC's ARI scores against six baseline clustering methods (CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN) across the 15-dataset benchmark, mirroring the paper's Supplementary Table S4.

## When to use

Use when you have a per-dataset performance metric matrix (e.g., ARI) for a proposed method plus baselines over a fixed benchmark and must produce a paired significance-test table.

## When NOT to use

- The score matrix is incomplete — any missing ARI values per dataset break the paired test design across the same 15 datasets.
- You need unpaired or independent-sample testing; the benchmark uses matched datasets per method, so paired=TRUE is required.
- The comparison vectors come from a single method's output rather than a method-vs-method benchmark matrix.

## Inputs

- Per-dataset ARI score matrix (15 benchmark datasets x 7 methods: CASCC + CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN) from LingyiC/CASCC_benchmark results

## Outputs

- supp_table_S4_wilcoxon_significance.csv — one row per CASCC-vs-baseline comparison, each with a Wilcoxon test statistic and p-value

## How to apply

Load the per-dataset ARI score matrix containing CASCC and the six baselines (CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN) over the 15 Table S1 benchmark datasets from the LingyiC/CASCC_benchmark results. For each baseline, call stats::wilcox.test in R with paired=TRUE, passing the CASCC ARI vector and that baseline's ARI vector matched by dataset. Collect each test's statistic and p-value into a single table with one row per CASCC-vs-baseline comparison. Write the result to supp_table_S4_wilcoxon_significance.csv, reproducing the significance results reported in the paper's Supplementary Table S4.

## Related tools

- **R** (Runtime used for stats::wilcox.test paired signed-rank testing and CSV table assembly (R version 4.0.5 in the paper)) — https://github.com/LingyiC/CASCC_benchmark
- **CASCC** (Method under test whose ARI vector is compared against each baseline) — https://github.com/LingyiC/CASCC

## Examples

```
Rscript -e 'w <- wilcox.test(ari$CASCC, ari$SIMLR, paired=TRUE); data.frame(method="SIMLR", statistic=w$statistic, p.value=w$p.value)'
```

## Evaluation signals

- The output table contains exactly six rows, one per CASCC-vs-baseline comparison (CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN).
- Every row has both a non-missing test statistic and a p-value; any missing row or field fails the check.
- The significance conclusions match those reported in the paper's Supplementary Table S4.
- Each paired test used the same 15 benchmark datasets for both CASCC and the baseline (vectors are aligned by dataset).

## Limitations

- Paired Wilcoxon tests require complete, dataset-aligned score vectors for both methods; missing ARI values for any of the 15 datasets invalidate the pairing.
- The table reproduces a specific paper result (Supplementary Table S4); extending to other metrics or datasets requires re-deriving the comparison structure.

## Evidence

- [other] For each of the six baselines, run a paired Wilcoxon signed-rank test in R using stats::wilcox.test with paired=TRUE: [evidence span withheld — anchor status: not_found]
- [results] Wilcoxon tests confirm significance, Supplementary Table S4: [evidence span withheld — anchor status: not_found]
- [readme] Benchmark repository reference: "# Benchmarking
https://github.com/LingyiC/CASCC_benchmark"
