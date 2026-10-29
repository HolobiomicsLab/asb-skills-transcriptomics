---
name: cross-method-benchmark-comparison
description: Use when a scRNA-seq annotation-transfer method must be compared head-to-head
  against alternatives on shared reference/query data, using balanced accuracy for
  cell typing or JSD for continuous-label transfer.
when_to_use_negative:
- The task is a single-method application with no competing method to compare — run
  the method directly instead of building a benchmark harness
- Reference datasets have fewer than ~1000 cells, where the Cell BLAST model itself
  is under-trained and any benchmark result would reflect under-training rather than
  method differences
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST
  role: Method under test; queries via DIRECTi posterior-distribution-based cell-to-cell
    similarity for cell-type transfer and continuous fate-probability transfer
  repo: https://github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model fitted on each reference to learn batch-corrected low-dimensional
    cell embeddings before BLAST indexing
  repo: https://github.com/gao-lab/Cell_BLAST
- name: scmap
  role: Competing annotation-transfer method using correlation-based mapping on the
    same reference/query splits
- name: CellFishing.jl
  role: Competing annotation-transfer method using LSH Hamming distance with a cutoff
    of 120
- name: scANVI
  role: Competing annotation-transfer method applied to the identical reference/query
    split of fate probabilities
- name: scVI
  role: Canonical variational autoencoder baseline whose diagonal-covariance Gaussian
    posterior contrasts with DIRECTi's posterior model
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:201a0e35e6ace851252c80c31b97dc55e81df9dec7fc3323fcc439a9ea4d2717
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gao-lab/Cell_BLAST
  tools:
  - Cell BLAST
  - CellFishing.jl
  - DIRECTi
  - scANVI
  - scVI
  - scmap
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cross-method-benchmark-comparison

## Summary

Benchmark Cell BLAST's query-based cell-type transfer against competing methods (scmap, CellFishing.jl, CCA anchor-based mapping, scANVI) on identical reference/query splits, scoring discrete typing by mean balanced accuracy and continuous fate transfer by Jensen–Shannon divergence.

## When to use

Use when a scRNA-seq annotation-transfer method must be compared head-to-head against alternatives on shared reference/query data, using balanced accuracy for cell typing or JSD for continuous-label transfer.

## When NOT to use

- The task is a single-method application with no competing method to compare — run the method directly instead of building a benchmark harness
- Reference datasets have fewer than ~1000 cells, where the Cell BLAST model itself is under-trained and any benchmark result would reflect under-training rather than method differences

## Inputs

- Reference and query scRNA-seq datasets as h5ad (AnnData) objects, split per benchmark group
- Positive query sets (cell types present in the reference) and negative query sets (cell types absent from the reference)
- Reference annotations to transfer: discrete cell-type labels or continuous cell-fate probability vectors (e.g. 7-dimensional Tusi fate probabilities)
- Installed method packages: Cell BLAST (Cell-BLAST via pip), scmap, CellFishing.jl, scANVI, CCA-based mapping

## Outputs

- Per-method balanced accuracy table across dataset groups (e.g. benchmark_balanced_accuracy.csv)
- Per-method Jensen–Shannon divergence values for continuous-label transfer (e.g. jsd_comparison.csv)
- Method ranking comparable to Fig. 2b-c and Supplementary Figs. 8-9 and 16b

## How to apply

Load reference and query scRNA-seq datasets as h5ad (AnnData) objects, deliberately including both positive queries (cell types present in the reference) and negative queries (cell types absent) so that specificity, not just sensitivity, can be scored. Fit a DIRECTi generative model on each reference with Cell_BLAST.directi and build an index via Cell_BLAST.blast, whose cell-to-cell similarity is the distance between posterior distributions; run the competing methods — scmap (correlation-based mapping), CellFishing.jl (LSH Hamming distance with cutoff 120), CCA anchor-based mapping, and scANVI — on the exact same reference/query split. For discrete cell typing, compute balanced accuracy per query set, combining correct cell-type assignment on positives with correct rejection of negatives, then aggregate into a mean balanced accuracy across dataset groups (as in Fig. 2b-c and Supplementary Figs. 8-9). For continuous labels, transfer values (e.g. seven-dimensional cell fate probabilities from the Tusi hematopoietic progenitor runs) from hit cells and measure Jensen–Shannon divergence between transferred and ground-truth distributions per method (Supplementary Fig. 16b). Write per-method score tables (e.g. benchmark_balanced_accuracy.csv, jsd_comparison.csv) so rankings can be validated against the published figures, where Cell BLAST achieved the highest overall mean balanced accuracy, superior negative-query specificity, and among the lowest JSD.

## Related tools

- **Cell BLAST** (Method under test; queries via DIRECTi posterior-distribution-based cell-to-cell similarity for cell-type transfer and continuous fate-probability transfer) — https://github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model fitted on each reference to learn batch-corrected low-dimensional cell embeddings before BLAST indexing) — https://github.com/gao-lab/Cell_BLAST
- **scmap** (Competing annotation-transfer method using correlation-based mapping on the same reference/query splits)
- **CellFishing.jl** (Competing annotation-transfer method using LSH Hamming distance with a cutoff of 120)
- **scANVI** (Competing annotation-transfer method applied to the identical reference/query split of fate probabilities)
- **scVI** (Canonical variational autoencoder baseline whose diagonal-covariance Gaussian posterior contrasts with DIRECTi's posterior model)

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST; import Cell_BLAST as cb; model = cb.directi.fit_DIRECTi(ref); blast = cb.blast.blast(model, ref)
```

## Evaluation signals

- Output table contains one score (balanced accuracy or JSD) for every compared method on every dataset group/query split
- Cell BLAST's negative-query MBA (specificity on cell types absent from the reference) exceeds that of scmap and CellFishing.jl, matching the published superiority claim
- Relative method ranking of mean balanced accuracy matches the pattern in Fig. 2b-c and Supplementary Figs. 8-9
- JSD comparison contains values for all five methods with Cell BLAST among the lowest divergences, matching Supplementary Fig. 16b
- Both positive and negative query sets are represented in each benchmark group, so balanced accuracy combines correct assignment and correct rejection

## Limitations

- Cell BLAST models are over-parameterized and require a reasonable number of samples to train properly; ~3000 cells is generally sufficient while datasets with fewer than 1000 cells may suffer under-training issues, biasing benchmark results
- Integrating datasets at mega-scale may over-correct batches with highly skewed cell-type composition, confounding cross-method comparisons on such data
- Existing competing metrics (approximate cosine distance, LSH Hamming distance) may not faithfully reflect cell-to-cell similarity and suffer from nonbiological confounding variation such as batch effect, which must be considered when interpreting method differences

## Evidence

- [results] In the query-based cell typing benchmark, Cell BLAST achieved the highest overall mean balanced accuracy and showed significantly superior specificity (higher negative-query MBA) compared to scmap and CellFishing.jl.: [evidence span withheld — anchor status: not_found]
- [supplementary] Run cell-type transfer for every query set with each method, including CellFishing.jl via LSH Hamming distance with a cutoff of 120.: [evidence span withheld — anchor status: not_found]
- [supplementary] Measured by Jensen–Shannon divergence between predicted and ground-truth cell fate probabilities, Cell BLAST made the most accurate predictions among the compared methods (Supplementary Fig. 16b).: [evidence span withheld — anchor status: not_found]
- [discussion] Cell BLAST explicitly models multilevel batch effect and uncertainty, motivating its use as a benchmarked querying method.: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [readme] Installation and import of the package under benchmarking.: "pip install Cell-BLAST"
