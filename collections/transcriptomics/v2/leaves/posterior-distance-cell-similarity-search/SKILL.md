---
name: posterior-distance-cell-similarity-search
description: Use when transferring cell type or continuous fate annotations from a
  labeled reference scRNA-seq dataset (AnnData/h5ad) to unlabeled query cells, especially
  across batches or species with batch effects.
when_to_use_negative:
- Reference datasets with fewer than 1000 cells, which may suffer from under-training
  issues (a cell number of 3000 is generally sufficient)
- Mega-scale dataset integration with highly skewed cell-type composition, where batch
  over-correction may occur
- When the query already has reliable curated annotations and no cross-dataset annotation
  transfer is needed
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST
  role: Cell-querying tool that searches query cells against a reference database
    and transfers annotations based on cell-to-cell similarities
  repo: https://github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Neural network-based deep generative model (fit via Cell_BLAST.directi.fit_DIRECTi)
    that learns batch-corrected low-dimensional cell embeddings for the BLAST index
  repo: https://github.com/gao-lab/Cell_BLAST
- name: Cell_BLAST.blast.BLAST
  role: Index/query module that computes cell-to-cell similarity as distance between
    posterior distributions of the fitted model
  repo: https://github.com/gao-lab/Cell_BLAST
- name: BLAST2CO
  role: Ontology-aware inference strategy that scores hit-derived Cell Ontology terms,
    propagates scores to ancestors, and assigns the top-scoring leaf term (threshold
    0.5)
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:5014afe09a3ae6823f1a39211bf8fba747b70d2902dae992db1e512f60ac5d75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gao-lab/Cell_BLAST
  tools:
  - BLAST2CO
  - Cell BLAST
  - Cell_BLAST.blast
  - Cell_BLAST.blast.BLAST
  - Cell_BLAST.data
  - Cell_BLAST.directi
  - Cell_BLAST.directi.fit_DIRECTi
  - Cell_BLAST.latent
  - DCA
  - DIRECTi
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# posterior-distance-cell-similarity-search

## Summary

Query single-cell transcriptomics datasets against a reference using Cell BLAST's DIRECTi deep generative model and its customized cell-to-cell similarity metric based on distances between posterior distributions, enabling annotation transfer across batches and species.

## When to use

Use when transferring cell type or continuous fate annotations from a labeled reference scRNA-seq dataset (AnnData/h5ad) to unlabeled query cells, especially across batches or species with batch effects.

## When NOT to use

- Reference datasets with fewer than 1000 cells, which may suffer from under-training issues (a cell number of 3000 is generally sufficient)
- Mega-scale dataset integration with highly skewed cell-type composition, where batch over-correction may occur
- When the query already has reliable curated annotations and no cross-dataset annotation transfer is needed

## Inputs

- Reference scRNA-seq dataset in AnnData/h5ad format with cell annotations (cell type or cell-differentiation potential)
- Query scRNA-seq dataset in AnnData/h5ad format (unlabeled cells)
- One-to-one ortholog mappings when transferring across species (e.g., mouse Tusi to human Velten)

## Outputs

- Hit lists of similar reference cells per query cell with posterior-distance-based similarities
- Transferred cell type annotations or predicted fate probabilities per query cell
- Per-fate Spearman correlation tables (e.g., fate_probability_spearman.csv)

## How to apply

Load reference and query datasets as AnnData (h5ad) objects via Cell_BLAST.data, mapping genes between species via one-to-one orthologs if needed. Fit the DIRECTi deep generative model on the reference with Cell_BLAST.directi.fit_DIRECTi, optionally applying online tuning so the model's low-dimensional embeddings adapt to the query dataset's batch and distribution. Build a Cell_BLAST.blast.BLAST index from the fitted model and query the cells, computing cell-to-cell similarity as the distance between posterior distributions rather than approximate cosine or LSH Hamming distances. Transfer annotations (discrete Cell Ontology terms or continuous fate probabilities) from hit cells to query cells using weighted hit annotations; for curated curation, propagate BLAST2CO hit-based confidence scores to ancestor terms and assign the leaf term with maximal confidence above the default 0.5 threshold. For continuous validation, compute Spearman rank correlations between transferred fate probabilities and known lineage marker expression.

## Related tools

- **Cell BLAST** (Cell-querying tool that searches query cells against a reference database and transfers annotations based on cell-to-cell similarities) — https://github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Neural network-based deep generative model (fit via Cell_BLAST.directi.fit_DIRECTi) that learns batch-corrected low-dimensional cell embeddings for the BLAST index) — https://github.com/gao-lab/Cell_BLAST
- **Cell_BLAST.blast.BLAST** (Index/query module that computes cell-to-cell similarity as distance between posterior distributions of the fitted model) — https://github.com/gao-lab/Cell_BLAST
- **BLAST2CO** (Ontology-aware inference strategy that scores hit-derived Cell Ontology terms, propagates scores to ancestors, and assigns the top-scoring leaf term (threshold 0.5))

## Examples

```
import Cell_BLAST as cb; model = cb.directi.fit_DIRECTi(ref.X, ref.var_names); blast = cb.blast.BLAST(model); hits = blast.query(query_data)
```

## Evaluation signals

- The BLAST index builds from a fitted DIRECTi model and returns hit lists for every query cell without import errors (cb.__version__ prints)
- Predicted fate probabilities correlate positively with known lineage marker expression; Spearman correlations per fate reproduce the relative ranking reported in Supplementary Fig. 17
- For Cell Ontology curation, assigned leaf terms exceed the default confidence threshold of 0.5 and are consistent with hit-derived annotations
- MAP scores with K set to 1% of total cells place Cell BLAST among the best-performing embedding methods relative to DCA, PCA, and tSNE (Supplementary Table 1/2, Fig. 3)

## Limitations

- Like all neural network-based models, Cell BLAST is over-parameterized and requires a reasonable number of samples to train properly; datasets with fewer than 1000 cells may suffer from under-training issues
- Integrating datasets at mega-scale may incur over-correcting of batches with highly skewed cell-type composition

## Evidence

- [abstract] Herein, we present Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric.: "Herein, we present Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric."
- [methods] a deep generative model (currently named DIRECTi): "a deep generative model (currently named **DIRECTi**)"
- [discussion] leaf terms (among those exceeding a given confidence threshold, by default 0.5) with maximal confidence score is assigned: "leaf terms (among those exceeding a given confidence threshold, by default 0.5) with maximal confidence score is assigned to the unlabeled cell as putative Cell Ontology curation"
- [readme] For each query cell, it searches for most similar cells in the reference database. Annotations in reference cells, e.g. cell type, can then be transfered: "For each query cell, it searches for most similar cells in the reference database.
Annotations in reference cells, e.g. cell type, can then be transfered to query
cells based on cell-to-cell"
- [discussion] small-sized datasets with fewer than 1000 cells may suffer from under-training issues: "a cell number of 3000 is generally sufficient for good performance, while small-sized datasets with fewer than 1000 cells may suffer from under-training issues."
