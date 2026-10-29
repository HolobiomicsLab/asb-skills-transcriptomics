---
name: cross-metric-benchmark-comparison-scrna-seq
description: Use when you must decide which similarity metric (posterior distance
  vs. Euclidean) better distinguishes same-type nearest neighbors between query and
  reference scRNA-seq cells within a fixed embedding space.
when_to_use_negative:
- The analysis goal is annotating query cells (ontology term transfer) rather than
  comparing similarity metrics — use Cell BLAST's BLAST2CO curation workflow instead
- Reference datasets contain fewer than ~1000 cells, since Cell BLAST's neural network
  model is over-parameterized and under-training would confound the metric comparison
- You only have a single embedding without a posterior/variational model — NPD requires
  posterior distributions, so only the Euclidean arm of the comparison is computable
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST
  role: Python package providing data loading (Cell_BLAST.data), embedding, and the
    NPD cell-to-cell similarity metric being benchmarked
  repo: https://github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model trained on the reference data whose adversarial variational
    posterior supplies the embedding and posterior distributions for NPD
  repo: https://github.com/gao-lab/Cell_BLAST
- name: scVI
  role: Canonical Gaussian-diagonal variational autoencoder baseline trained on the
    same reference data for the cross-embedding comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:3fa48232df2edcabedb31ec89fe8cf06033c41a0620c85b5ff2dc7a606c510be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gao-lab/Cell_BLAST
  tools:
  - Cell BLAST
  - DIRECTi
  - scVI
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cross-metric-benchmark-comparison-scrna-seq

## Summary

Benchmark two cell-to-cell similarity metrics (posterior-based normalized projection distance vs. Euclidean distance) within the same low-dimensional embedding by ROC/AUC analysis of nearest-neighbor cell-type querying, as done in Cell BLAST's Supplementary Figs. 6g-h and 8c.

## When to use

Use when you must decide which similarity metric (posterior distance vs. Euclidean) better distinguishes same-type nearest neighbors between query and reference scRNA-seq cells within a fixed embedding space.

## When NOT to use

- The analysis goal is annotating query cells (ontology term transfer) rather than comparing similarity metrics — use Cell BLAST's BLAST2CO curation workflow instead
- Reference datasets contain fewer than ~1000 cells, since Cell BLAST's neural network model is over-parameterized and under-training would confound the metric comparison
- You only have a single embedding without a posterior/variational model — NPD requires posterior distributions, so only the Euclidean arm of the comparison is computable

## Inputs

- Query scRNA-seq dataset in h5ad format (e.g., pancreatic benchmark query data)
- Reference scRNA-seq dataset in h5ad format with cell-type annotations
- Trained DIRECTi embedding of the reference data
- Trained scVI embedding of the same reference data (Gaussian-diagonal VAE baseline)
- Query-reference nearest-neighbor pairs with cell-type labels

## Outputs

- Per-pair NPD (posterior distribution distance) values and Euclidean distances between embedding vectors
- ROC curves for NPD vs. Euclidean distance on the same-type-neighbor task
- AUC values per metric per embedding (DIRECTi and scVI), comparable to Supplementary Figs. 6g-h and 8c

## How to apply

Load the query and reference scRNA-seq datasets in h5ad format via the Cell_BLAST.data module, then train a DIRECTi model on the reference (default RMSpropOptimizer) and a scVI model as the canonical Gaussian-diagonal variational autoencoder baseline. Retrieve nearest-neighbor pairs between query cells and reference cells under both the DIRECTi and scVI low-dimensional embeddings. For each query-reference pair, compute two distances: the posterior-distribution distance (NPD) and the plain Euclidean distance between embedding vectors. Build ROC curves for the binary task 'nearest reference cell is of the same type as the query cell' under each metric and compute AUC values; the decision point is whether the NPD-based ROC curve dominates the Euclidean-distance ROC curve. This reproduces the Supplementary Figs. 6g-h and 8c comparison, confirming that NPD is more accurate than Euclidean distance with Cell BLAST's adversarial variational posterior, including in a ten-dimensional embedding with independent query data.

## Related tools

- **Cell BLAST** (Python package providing data loading (Cell_BLAST.data), embedding, and the NPD cell-to-cell similarity metric being benchmarked) — https://github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model trained on the reference data whose adversarial variational posterior supplies the embedding and posterior distributions for NPD) — https://github.com/gao-lab/Cell_BLAST
- **scVI** (Canonical Gaussian-diagonal variational autoencoder baseline trained on the same reference data for the cross-embedding comparison)

## Examples

```
conda create -n cb python=3.9 && source activate cb && pip install Cell-BLAST; import Cell_BLAST as cb; models = cb.directi.fit_DIRECTi(reference)
```

## Evaluation signals

- The NPD-based ROC curve dominates (lies above) the Euclidean-distance ROC curve within the same embedding
- NPD yields a higher AUC than Euclidean distance for the same-type-neighbor task, as reported for Cell BLAST's adversarial variational posterior
- AUC improvement with the variational posterior is confirmed in a ten-dimensional embedding with independent query data
- Both metrics are evaluated on identical nearest-neighbor pairs under both DIRECTi and scVI embeddings, so the comparison is paired
- Import check passes: 'import Cell_BLAST as cb' executes and cb.__version__ prints

## Limitations

- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition, which would distort nearest-neighbor pairs
- Cell BLAST is over-parameterized and requires a reasonable number of samples; datasets with fewer than 1000 cells may suffer from under-training issues (3000 cells generally sufficient)
- The NPD advantage depends on the variational posterior; scVI's Gaussian diagonal-covariance posterior is the canonical baseline against which Cell BLAST's posterior is contrasted

## Evidence

- [supplementary] NPD superiority with the adversarial variational posterior: [evidence span withheld — anchor status: not_found]
- [discussion] Posterior model difference: "the canonical variational autoencoder model used by scVI and several other tools11,25–27 enforces Gaussian distribution (with diagonal-covariance matrices) for the variational posterior."
- [discussion] Metric design rationale: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [readme] Installation and import check: "pip install Cell-BLAST"
