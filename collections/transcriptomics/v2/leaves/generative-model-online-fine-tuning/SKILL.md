---
name: generative-model-online-fine-tuning
description: Use when transferring cell annotations (discrete types or continuous
  fate probabilities) from a labeled reference scRNA-seq dataset to an independent
  query dataset whose batch or distribution differs, e.g. cross-species or cross-platform
  queries in h5ad format.
when_to_use_negative:
- Reference datasets with fewer than ~1000 cells, where the over-parameterized neural
  network may suffer from under-training
- Queries where labeled reference data are already perfectly matched and batch-corrected,
  making the online-tuning machinery unnecessary
- Annotation tasks solvable by simple label voting on identical gene spaces where
  adversarial batch alignment risks over-correcting
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Cell BLAST
  role: Overall cell-querying framework; annotation transfer from hit cells to query
    cells via cell-to-cell similarity
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Neural network-based deep generative model producing low-dimensional cell
    embeddings robust to batch effect
- name: Cell_BLAST.directi.fit_DIRECTi
  role: Fits the DIRECTi model on the reference dataset with online tuning to adapt
    embeddings to the query distribution
  repo: https://github.com/gao-lab/Cell_BLAST
- name: Cell_BLAST.blast.BLAST
  role: Builds the index from the fitted reference model and computes posterior-distribution-based
    cell-to-cell similarity for querying
  repo: https://github.com/gao-lab/Cell_BLAST
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:b0888c320a6a8b5763740bb9e37c5a48629e6496d83f6d22027f5b93bd383322
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - Cell_BLAST.blast.BLAST
  - Cell_BLAST.directi.fit_DIRECTi
  - DIRECTi
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# generative-model-online-fine-tuning

## Summary

Adapts a pretrained DIRECTi deep generative model (Cell BLAST) to an independent query dataset via online-tuning, so that low-dimensional cell embeddings align across batch/species and annotations can be transferred through a posterior-distribution-based cell-to-cell similarity search.

## When to use

Use when transferring cell annotations (discrete types or continuous fate probabilities) from a labeled reference scRNA-seq dataset to an independent query dataset whose batch or distribution differs, e.g. cross-species or cross-platform queries in h5ad format.

## When NOT to use

- Reference datasets with fewer than ~1000 cells, where the over-parameterized neural network may suffer from under-training
- Queries where labeled reference data are already perfectly matched and batch-corrected, making the online-tuning machinery unnecessary
- Annotation tasks solvable by simple label voting on identical gene spaces where adversarial batch alignment risks over-correcting

## Inputs

- Labeled reference scRNA-seq dataset (AnnData .h5ad) with cell fate or cell type annotations, e.g. mouse Tusi hematopoietic progenitor data
- Query scRNA-seq dataset (AnnData .h5ad), e.g. human Velten HSPC data
- One-to-one ortholog mapping table to unify gene spaces across species
- Known lineage marker gene lists (e.g. Supplementary Fig. 17 marker lists) for validation

## Outputs

- DIRECTi model embedding with online-tuned query alignment
- BLAST hit list of similar reference cells per query cell with posterior-distribution distances
- Predicted fate probability per query cell
- Per-fate Spearman correlation table, e.g. fate_probability_spearman.csv

## How to apply

First harmonize gene spaces (e.g. one-to-one ortholog mapping for cross-species transfer, so mouse Tusi and human Velten matrices share common genes) and load both as AnnData h5ad objects. Train the DIRECTi deep generative model on the reference with Cell_BLAST.directi.fit_DIRECTi, then apply online tuning so the model adapts its embeddings to the query's batch and distribution — this adversarial batch alignment is what the paper credits for improved cross-dataset transfer. Build a Cell_BLAST.blast.BLAST index from the fitted reference model and query cells, measuring cell-to-cell similarity as the distance between posterior distributions rather than plain cosine or Hamming metrics. Transfer reference annotations (cell fate probabilities) to each query cell as weighted aggregates of hit-cell annotations, then validate quantitatively (e.g. Spearman correlation of predicted fate probabilities against known lineage marker expression per Supplementary Fig. 17 marker lists). Requires a reasonable training sample size: ~3000 cells is generally sufficient, while datasets under 1000 cells risk under-training.

## Related tools

- **Cell BLAST** (Overall cell-querying framework; annotation transfer from hit cells to query cells via cell-to-cell similarity) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Neural network-based deep generative model producing low-dimensional cell embeddings robust to batch effect)
- **Cell_BLAST.directi.fit_DIRECTi** (Fits the DIRECTi model on the reference dataset with online tuning to adapt embeddings to the query distribution) — https://github.com/gao-lab/Cell_BLAST
- **Cell_BLAST.blast.BLAST** (Builds the index from the fitted reference model and computes posterior-distribution-based cell-to-cell similarity for querying) — https://github.com/gao-lab/Cell_BLAST

## Examples

```
import Cell_BLAST as cb; model = cb.directi.fit_DIRECTi(ref.X, ref.var_names, ref.obs, online_tuning=query); blast = cb.blast.BLAST(model, ref); hits = blast.search(query.X)
```

## Evaluation signals

- Spearman rank correlation between predicted fate probabilities and known lineage marker expression is computed for each fate and written to fate_probability_spearman.csv
- Online-tuned transfer should show significantly higher correlation than non-tuned baselines for most lineages in the cross-species transfer to Velten (per the reported finding)
- Query and reference share a common gene space after one-to-one ortholog mapping (matrix dimensions match)
- Import check succeeds: 'import Cell_BLAST as cb' and cb.__version__ prints in the interpreter
- Each query cell receives hit cells from the BLAST search, yielding a full set of predicted fate probabilities without NaN entries

## Limitations

- The Cell BLAST model is over-parameterized (like all neural network-based models) and requires a reasonable number of samples to train properly
- A cell number of 3000 is generally sufficient for good performance, while small datasets with fewer than 1000 cells may suffer from under-training issues
- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition

## Evidence

- [results] Benefiting from its adversarial batch alignment-based online-tuning mode, Cell BLAST shows significantly higher correlation between predicted cell fate probabilities and known lineage marker expression for most lineages in the inter-species transfer to 'Velten'.: [evidence span withheld — anchor status: not_found]
- [intro] we introduce Cell BLAST, a cell-querying tool employing a customized neural network-based generative model that effectively handles batch effect: "we introduce Cell BLAST, a cell-querying tool employing a customized neural network-based generative model that effectively handles batch effect"
- [discussion] By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [methods] import Cell_BLAST as cb: "import Cell_BLAST as cb"
- [readme] Check if the package can be imported in Python interpreter: [evidence span withheld — anchor status: external_doc]
