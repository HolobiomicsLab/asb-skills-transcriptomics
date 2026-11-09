---
name: single-cell-query-based-cell-type-annotation
description: Use when annotating unlabeled scRNA-seq query cells against a curated
  reference with known cell types, especially across datasets with batch effects,
  and when negative queries (cell types absent from the reference) must be rejected.
when_to_use_negative:
- Query datasets with fewer than ~1000 cells — the over-parameterized DIRECTi model
  may be under-trained (3000 cells is generally sufficient)
- Mega-scale reference integration with highly skewed cell-type composition, which
  may cause over-correcting of batches
- Tasks needing a continuous latent embedding for exploratory dimensionality reduction
  rather than annotation transfer via cell querying
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cell BLAST
  role: 'Cell-querying tool: searches reference for cells similar to each query cell
    via posterior-distance similarity and transfers reference annotations'
  repo: https://github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model fitted on the reference dataset to produce unbiased
    cell embeddings that handle multilevel batch effect
  repo: https://github.com/gao-lab/Cell_BLAST
- name: BLAST2CO
  role: 'Ontology-aware post-processing: computes hit-based confidence scores for
    Cell Ontology terms, propagates them to ancestors, and assigns the best leaf term'
  repo: https://github.com/gao-lab/Cell_BLAST
- name: scmap
  role: Baseline comparator performing correlation-based mapping of query cells for
    the benchmark
- name: CellFishing.jl
  role: Baseline comparator using LSH Hamming distance with a cutoff of 120 for the
    benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:5d1e2e17aabf71501d5cec7ee2b23c8158ce4cdd9362d6baf11b9a31b209e211
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gao-lab/Cell_BLAST
  tools:
  - BLAST2CO
  - Cell BLAST
  - CellFishing.jl
  - DIRECTi
  - scmap
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-query-based-cell-type-annotation

## Summary

Transfer cell-type annotations from curated scRNA-seq reference datasets to unlabeled query cells by searching references with Cell BLAST, which fits a DIRECTi generative model and ranks hits by distance between posterior distributions rather than cosine or LSH Hamming distance. Benchmarked with mean balanced accuracy over positive and negative queries, it outperformed scmap and CellFishing.jl, especially in rejecting negative queries.

## When to use

Use when annotating unlabeled scRNA-seq query cells against a curated reference with known cell types, especially across datasets with batch effects, and when negative queries (cell types absent from the reference) must be rejected.

## When NOT to use

- Query datasets with fewer than ~1000 cells — the over-parameterized DIRECTi model may be under-trained (3000 cells is generally sufficient)
- Mega-scale reference integration with highly skewed cell-type composition, which may cause over-correcting of batches
- Tasks needing a continuous latent embedding for exploratory dimensionality reduction rather than annotation transfer via cell querying

## Inputs

- Reference scRNA-seq dataset with known cell-type annotations (e.g. curated ACA reference panels, h5/h5ad format)
- Query scRNA-seq dataset (unlabeled cells)
- Optional: positive and negative query groups per benchmark dataset group
- Optional: Cell Ontology terms for ontology-aware annotation transfer

## Outputs

- Per-query-cell hit list of similar reference cells ranked by posterior distance
- Transferred cell-type / Cell Ontology term annotations with confidence scores
- Per-query-set balanced accuracy scores and a per-method mean balanced accuracy table (benchmark_balanced_accuracy.csv)

## How to apply

Load paired reference and query scRNA-seq datasets, ensuring the query set includes both positive queries (cell types present in the reference) and negative queries (cell types absent). Fit a DIRECTi generative model on each reference via Cell_BLAST.directi and build a searchable index with Cell_BLAST.blast; the model explicitly handles multilevel batch effect and the posterior-distance similarity metric captures uncertainty in cell-to-cell similarity. Run the query through the Cell BLAST hit search and transfer reference annotations (e.g. cell type, Cell Ontology terms) to query cells based on the returned hits. Optionally post-process hits ontology-aware: BLAST2CO assigns a hit-based confidence score per identified Cell Ontology term, propagates scores to ancestor terms, and assigns the leaf term with maximal confidence above a default threshold of 0.5. Judge success by balanced accuracy — combining correct cell-type assignment on positive queries with correct rejection of negative queries — aggregated into a mean balanced accuracy comparable to Fig. 2b-c and Supplementary Figs. 8-9, and export scores to benchmark_balanced_accuracy.csv.

## Related tools

- **Cell BLAST** (Cell-querying tool: searches reference for cells similar to each query cell via posterior-distance similarity and transfers reference annotations) — https://github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model fitted on the reference dataset to produce unbiased cell embeddings that handle multilevel batch effect) — https://github.com/gao-lab/Cell_BLAST
- **BLAST2CO** (Ontology-aware post-processing: computes hit-based confidence scores for Cell Ontology terms, propagates them to ancestors, and assigns the best leaf term) — https://github.com/gao-lab/Cell_BLAST
- **scmap** (Baseline comparator performing correlation-based mapping of query cells for the benchmark)
- **CellFishing.jl** (Baseline comparator using LSH Hamming distance with a cutoff of 120 for the benchmark)

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST; python -c "import Cell_BLAST as cb; print(cb.__version__)"
```

## Evaluation signals

- Balanced accuracy per query set combining positive-query cell-type assignment accuracy with negative-query rejection (specificity / negative-query MBA)
- Mean balanced accuracy across the four benchmark dataset groups should be highest for Cell BLAST relative to scmap and CellFishing.jl, reproducing Fig. 2b-c / Supplementary Figs. 8-9
- Negative-query mean balanced accuracy (specificity) should exceed that of scmap and CellFishing.jl
- Package imports and reports a version via cb.__version__ after pip installation
- BLAST2CO-assigned Cell Ontology leaf terms exceed the default confidence threshold of 0.5

## Limitations

- Cell BLAST, like all neural network-based models, is over-parameterized and requires a reasonable number of samples (3000 cells generally sufficient; datasets with fewer than 1000 cells may suffer from under-training)
- Integrating datasets at mega-scale may incur over-correcting of batches with highly skewed cell-type composition
- Benchmark baselines rely on metrics (approximate cosine distance, LSH Hamming distance) that may not faithfully reflect cell-to-cell similarity under nonbiological confounding variation such as batch effect

## Evidence

- [other] Load the reference and query scRNA-seq datasets for each of the four benchmark groups, including both positive queries (cell types present in the reference) and negative queries (cell types absent from the reference): [evidence span withheld — anchor status: not_found]
- [other] Fit DIRECTi generative models on each reference dataset and build Cell BLAST indexes using Cell_BLAST.directi and Cell_BLAST.blast: [evidence span withheld — anchor status: not_found]
- [other] In the query-based cell typing benchmark, Cell BLAST achieved the highest overall mean balanced accuracy and showed significantly superior specificity (higher negative-query MBA): [evidence span withheld — anchor status: not_found]
- [intro] we introduce Cell BLAST, a cell-querying tool employing a customized neural network-based generative model that effectively handles batch effect, as well as a cell-to-cell similarity metric: "we introduce Cell BLAST, a cell-querying tool employing a customized neural network-based generative model that effectively handles batch effect, as well as a cell-to-cell similarity metric"
- [discussion] leaf terms (among those exceeding a given confidence threshold, by default 0.5) with maximal confidence score is assigned to the unlabeled cell: "leaf terms (among those exceeding a given confidence threshold, by default 0.5) with maximal confidence score is assigned to the unlabeled cell as putative Cell Ontology curation"
- [methods] import Cell_BLAST as cb: "import Cell_BLAST as cb"
- [readme] pip install Cell-BLAST: "pip install Cell-BLAST"
