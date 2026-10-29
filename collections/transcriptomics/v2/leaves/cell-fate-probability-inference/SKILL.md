---
name: cell-fate-probability-inference
description: Use when a scRNA-seq task asks to transfer continuous (non-discrete)
  annotations such as 7-dimensional cell fate probability vectors between two sequencing
  runs of the Tusi hematopoietic progenitor dataset, and to rank annotation-transfer
  methods by JSD.
when_to_use_negative:
- The annotation of interest is a discrete cell type label rather than a continuous
  fate probability distribution — the Cell BLAST ontology-based curation path (BLAST2CO
  confidence threshold 0.5) targets discrete Cell Ontology terms instead
- The reference dataset has fewer than ~1000 cells, where the over-parameterized DIRECTi/Cell
  BLAST neural network is under-trained
- The goal is mega-scale dataset integration rather than cross-run annotation transfer
  — over-correcting batches with highly skewed cell-type composition becomes a risk
edam_operation: http://edamontology.org/operation_2422
tools:
- name: Cell BLAST
  role: 'Querying tool: searches reference cells by posterior-distribution-based cell-to-cell
    similarity and transfers fate probabilities to query cells'
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model fitted on the reference run to learn low-dimensional
    cell embeddings that model batch effect
  repo: github.com/gao-lab/Cell_BLAST
- name: scVI
  role: Competing variational-autoencoder-based annotation-transfer baseline compared
    via JSD in the benchmark
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:42d74cd6966673ab23753aaf9eb344735ee17079d6f72d185a1226ffde066938
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - DIRECTi
  - scVI
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cell-fate-probability-inference

## Summary

Transfer continuous cell fate probabilities across experimental batches of the Tusi hematopoietic progenitor dataset by treating one sequencing run as reference and the other as query with Cell BLAST, then benchmark accuracy via Jensen–Shannon divergence against scmap, CellFishing.jl, CCA anchors, and scANVI.

## When to use

Use when a scRNA-seq task asks to transfer continuous (non-discrete) annotations such as 7-dimensional cell fate probability vectors between two sequencing runs of the Tusi hematopoietic progenitor dataset, and to rank annotation-transfer methods by JSD.

## When NOT to use

- The annotation of interest is a discrete cell type label rather than a continuous fate probability distribution — the Cell BLAST ontology-based curation path (BLAST2CO confidence threshold 0.5) targets discrete Cell Ontology terms instead
- The reference dataset has fewer than ~1000 cells, where the over-parameterized DIRECTi/Cell BLAST neural network is under-trained
- The goal is mega-scale dataset integration rather than cross-run annotation transfer — over-correcting batches with highly skewed cell-type composition becomes a risk

## Inputs

- Two sequencing runs of the Tusi hematopoietic progenitor dataset as h5ad (AnnData) objects, one designated reference and one query
- Seven-dimensional continuous cell fate probability annotations on the reference run

## Outputs

- jsd_comparison.csv — a table with one Jensen–Shannon divergence value per method (Cell BLAST, scmap, CellFishing.jl, CCA anchor-based mapping, scANVI)
- A Supplementary Fig. 16b-style comparison plot of JSD per method
- Predicted fate probability vectors assigned to each query cell

## How to apply

Load the two sequencing runs of the Tusi hematopoietic progenitor dataset as h5ad (AnnData) objects, designating one run as reference and the other as query. Fit a DIRECTi deep generative model on the reference run with Cell BLAST to learn low-dimensional cell embeddings that model multilevel batch effect and uncertainty in cell-to-cell similarity. Query the held-out run against the reference with BLAST, retrieving hit cells by posterior-distribution-based cell-to-cell similarity, and transfer the seven-dimensional cell fate probabilities from hits to query cells. Apply the four competing annotation-transfer methods — scmap, CellFishing.jl, CCA anchor-based mapping, and scANVI — to the identical reference/query split of fate probabilities. Compute the Jensen–Shannon divergence between transferred and reference fate probability distributions per method, tabulate them, and render the comparison in the style of Supplementary Fig. 16b.

## Related tools

- **Cell BLAST** (Querying tool: searches reference cells by posterior-distribution-based cell-to-cell similarity and transfers fate probabilities to query cells) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model fitted on the reference run to learn low-dimensional cell embeddings that model batch effect) — github.com/gao-lab/Cell_BLAST
- **scVI** (Competing variational-autoencoder-based annotation-transfer baseline compared via JSD in the benchmark)

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST && python -c "import Cell_BLAST as cb; print(cb.__version__)"
```

## Evaluation signals

- jsd_comparison.csv contains a JSD value for each of the five methods (Cell BLAST, scmap, CellFishing.jl, CCA anchor-based mapping, scANVI)
- Cell BLAST ranks among the lowest Jensen–Shannon divergences, matching the pattern reported in Supplementary Fig. 16b
- Each transferred fate probability vector is 7-dimensional (matches the ground-truth fate probability dimensionality)
- Query cells receive non-degenerate fate distributions (JSD is finite and within [0, 1] per cell)

## Limitations

- Cell BLAST is over-parameterized like all neural network-based models and requires a reasonable number of samples to train properly; 3000 cells is generally sufficient, while datasets with fewer than 1000 cells may suffer from under-training
- Integrating datasets at mega-scale may over-correct batches with highly skewed cell-type composition

## Evidence

- [discussion] Cell BLAST is an accurate and robust querying algorithm for heterogeneous single-cell transcriptome datasets.: "Cell BLAST is an accurate and robust querying algorithm for heterogeneous single-cell transcriptome datasets."
- [abstract] we demonstrate the effectiveness of Cell BLAST in annotating discrete cell types and continuous cell differentiation potential: "we demonstrate the effectiveness of Cell BLAST in annotating discrete cell types and continuous cell differentiation potential"
- [readme] For each query cell, it searches for most similar cells in the reference database.: "For each query cell, it searches for most similar cells in the reference database."
- [methods] pip install Cell-BLAST: "pip install Cell-BLAST"
- [discussion] a cell number of 3000 is generally sufficient, but datasets with fewer than 1000 cells may suffer from under-training issues.: [evidence span withheld — anchor status: not_found]
