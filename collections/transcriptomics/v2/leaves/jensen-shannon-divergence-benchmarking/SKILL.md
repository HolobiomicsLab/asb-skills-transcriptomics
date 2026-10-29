---
name: jensen-shannon-divergence-benchmarking
description: Use when transferring continuous annotations (e.g. 7-dimensional cell
  fate probabilities) between batches or datasets and you need a distribution-level
  metric to rank annotation-transfer methods.
when_to_use_negative:
- The annotation to transfer is a discrete cell-type label rather than a continuous
  probability distribution — label-accuracy metrics would be more appropriate
- The query and reference datasets have fewer than ~1000 cells, since the DIRECTi
  model may suffer under-training issues
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST
  role: Cell-querying method whose DIRECTi embeddings and posterior-distribution-based
    cell-to-cell similarity are used to transfer fate probabilities; the method expected
    to achieve lowest JSD
  repo: https://github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model fit on the reference run to learn low-dimensional cell
    embeddings that model batch effect
- name: scVI
  role: Canonical variational autoencoder baseline referenced in discussion of the
    generative-model design underlying the comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:86a6a66be103d97f629e92d27a49d623960e0c3f9ffa53ccbaf641f0f8cf606f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gao-lab/Cell_BLAST
  tools:
  - Cell BLAST
  - DIRECTi
  - scVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# jensen-shannon-divergence-benchmarking

## Summary

Benchmark annotation-transfer methods by measuring Jensen–Shannon divergence (JSD) between transferred and reference cell fate probability distributions, as done to compare Cell BLAST against scmap, CellFishing.jl, CCA anchor-based mapping, and scANVI on the Tusi hematopoietic progenitor dataset.

## When to use

Use when transferring continuous annotations (e.g. 7-dimensional cell fate probabilities) between batches or datasets and you need a distribution-level metric to rank annotation-transfer methods.

## When NOT to use

- The annotation to transfer is a discrete cell-type label rather than a continuous probability distribution — label-accuracy metrics would be more appropriate
- The query and reference datasets have fewer than ~1000 cells, since the DIRECTi model may suffer under-training issues

## Inputs

- Two sequencing runs of the Tusi hematopoietic progenitor dataset as h5ad (AnnData) objects, split into reference and query
- Seven-dimensional cell fate probabilities annotated on reference cells

## Outputs

- jsd_comparison.csv containing a JSD value per compared method (Cell BLAST, scmap, CellFishing.jl, CCA anchor-based mapping, scANVI)
- Per-method comparison plot in the style of Supplementary Fig. 16b

## How to apply

Load the two sequencing runs of the Tusi hematopoietic progenitor dataset as h5ad (AnnData) objects, designating one run as reference and the other as query. Fit a DIRECTi deep generative model on the reference with Cell BLAST to learn low-dimensional cell embeddings that model batch effect, then query the held-out run via BLAST to retrieve hit cells by posterior-distribution-based cell-to-cell similarity and transfer the seven-dimensional fate probabilities from hits to query cells. Run the competing methods — scmap, CellFishing.jl, CCA anchor-based mapping, and scANVI — on the identical reference/query split. For each method, compute the Jensen–Shannon divergence between transferred and reference fate probability distributions per cell, tabulate the JSD values in jsd_comparison.csv, and render the per-method comparison in the style of Supplementary Fig. 16b. Rationale: JSD is a symmetric, bounded divergence suited to comparing full probability distributions rather than single labels, so it captures accuracy of continuous fate-probability transfer, where Cell BLAST achieved the lowest divergences.

## Related tools

- **Cell BLAST** (Cell-querying method whose DIRECTi embeddings and posterior-distribution-based cell-to-cell similarity are used to transfer fate probabilities; the method expected to achieve lowest JSD) — https://github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model fit on the reference run to learn low-dimensional cell embeddings that model batch effect)
- **scVI** (Canonical variational autoencoder baseline referenced in discussion of the generative-model design underlying the comparison)

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST && python -c "import Cell_BLAST as cb; print(cb.__version__)"
```

## Evaluation signals

- jsd_comparison.csv contains a JSD value for each of the five compared methods
- The relative method ranking matches Supplementary Fig. 16b, with Cell BLAST among the lowest divergences
- All methods are evaluated on the identical reference/query split of fate probabilities
- JSD values are bounded in [0, 1] (with base-2 logarithm) and finite for every query cell

## Limitations

- Cell BLAST is over-parameterized (like all neural network-based models) and requires a reasonable number of samples to train properly
- A cell number of 3000 is generally sufficient, but datasets with fewer than 1000 cells may suffer from under-training issues
- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition

## Evidence

- [supplementary] Cell BLAST made the most accurate predictions among the compared methods: [evidence span withheld — anchor status: not_found]
- [discussion] Cell BLAST is accurate and robust for heterogeneous datasets: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [abstract] Cell BLAST built on a neural network-based generative model: "Herein, we present Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric."
- [readme] Import the package in Python: "import Cell_BLAST as cb
print(cb.__version__)"
