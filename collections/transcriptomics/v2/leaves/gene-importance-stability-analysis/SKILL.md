---
name: gene-importance-stability-analysis
description: Use when you need to verify that a trained SIMS/TabNet classifier's sparse
  gene-selection masks for a specific cell class (e.g., Pvalb+ interneurons) are consistent
  across independent runs, or to recover an interpretable marker-gene set.
when_to_use_negative:
- Input is already a curated marker-gene/feature table rather than raw cell-by-gene
  expression requiring model-derived importance
- You need cross-species label transfer or gene-importance transfer — SIMS has only
  been validated within the same species because neuronal subtypes diverge between
  species
- The expression matrices are in gene x cell orientation; SIMS requires cell x gene
  since training samples are individual transcriptomes
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: SIMS
  role: Pipeline that wraps TabNet for single-cell classification and provides the
    explain() method used to sample sparse feature-selection masks
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying attention-based tabular architecture whose sparse masks produce
    per-gene explain values; configured with n_a=64, n_d=64
  repo: https://github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Training framework used to set up EarlyStopping (val_loss, patience=50), LearningRateMonitor,
    and the 100-epoch trainer
  repo: https://github.com/braingeneers/SIMS
- name: Scanpy
  role: Used for light filtering and normalization (filter_cells, filter_genes, normalize_total,
    log1p, scale) before explain runs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:c5c0b9f28fd519ed558b571690e8420ad1a0dd0358b7634b9e55763809328cbe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jlehrer1/sims
  repo_url: https://github.com/braingeneers/SIMS
  tools:
  - PyTorch Lightning
  - SIMS
  - Scanpy
  - TabNet
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# Gene-importance stability analysis via repeated TabNet explain masks (SIMS)

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the reproducibility of SIMS/TabNet sparse feature-selection masks by running the explain procedure many times (e.g., 300 runs) for a target cell class and aggregating per-gene explain values, demonstrating stable, concentrated gene selection for Pvalb+ interneurons.

## When to use

Use when you need to verify that a trained SIMS/TabNet classifier's sparse gene-selection masks for a specific cell class (e.g., Pvalb+ interneurons) are consistent across independent runs, or to recover an interpretable marker-gene set.

## When NOT to use

- Input is already a curated marker-gene/feature table rather than raw cell-by-gene expression requiring model-derived importance
- You need cross-species label transfer or gene-importance transfer — SIMS has only been validated within the same species because neuronal subtypes diverge between species
- The expression matrices are in gene x cell orientation; SIMS requires cell x gene since training samples are individual transcriptomes

## Inputs

- Cell-by-gene single-cell RNA-seq expression matrix (AnnData .h5ad) from the Allen mouse cortex/hippocampus dataset, log1p-normalized with filter_cells(min_genes=100) and filter_genes(min_cells=3) applied
- Trained SIMS/TabNet model checkpoint (.ckpt) with class_label annotations
- Target cell-class label (e.g., Pvalb+ interneuron) for which explain masks are sampled

## Outputs

- Per-gene mean explain-value table aggregated over 300 explain runs (top genes ~0.2 vs ~1e-6 for median genes)
- Dispersion index for the top 50 genes
- Ranked selected-gene set for the target class, comparable against reported Pvalb+ markers (Kcnip, Igf1, Rpp25, Dlx1, Dlx5, Gad1, Cck, Satb1, Pvalb, Lypd6, Dlx6os-1, Bmp3)

## How to apply

Train a SIMS model on the Allen mouse cortex/hippocampus single-cell matrix with labels_key='class_label', configuring TabNet with n_a=64 and n_d=64 and weighting the loss inversely by label frequency via sims.weights. Set up the PyTorch Lightning trainer with EarlyStopping monitoring val_loss (patience=50), LearningRateMonitor per epoch, and num_epochs=100, then call sims.train(). Run the TabNet explain procedure (sims.explain) 300 times for the Pvalb+ interneuron class to sample the sparse feature-selection masks. Aggregate the 300 runs into per-gene mean explain values and compute the dispersion index for the top 50 genes. Rationale: SIMS is based on TabNet and optimized for tabular data, so its attention-based masks yield per-gene importance; repetition across runs tests whether the selected gene set is an artifact of a single training run.

## Related tools

- **SIMS** (Pipeline that wraps TabNet for single-cell classification and provides the explain() method used to sample sparse feature-selection masks) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying attention-based tabular architecture whose sparse masks produce per-gene explain values; configured with n_a=64, n_d=64) — https://github.com/braingeneers/SIMS
- **PyTorch Lightning** (Training framework used to set up EarlyStopping (val_loss, patience=50), LearningRateMonitor, and the 100-epoch trainer) — https://github.com/braingeneers/SIMS
- **Scanpy** (Used for light filtering and normalization (filter_cells, filter_genes, normalize_total, log1p, scale) before explain runs)

## Examples

```
from scsims import SIMS; sims = SIMS(adata=adata, labels_key='class_label'); sims.setup_model(n_a=64, n_d=64, weights=sims.weights); sims.train(); explainability_matrix, _labels = sims.explain(adata)
```

## Evaluation signals

- Mean explain values for top-ranked genes reach ~0.2, versus ~1e-6 for the median gene, after aggregating 300 runs
- Top 50 genes show low dispersion index across the 300 runs, indicating stable selection
- The top-ranked gene set recovers the reported Pvalb+ marker set: Kcnip, Igf1, Rpp25, Dlx1, Dlx5, Gad1, Cck, Satb1, Pvalb, Lypd6, Dlx6os-1, Bmp3 (Figure 3E-F comparison)
- Per-gene explain table is produced for all genes in the input matrix with no missing per-run values
- The explain procedure was run on data preprocessed identically to training (normalize_total, log1p, scale)

## Limitations

- Feature masks are derived from a supervised model that relies on pre-existing annotations; technical factors such as sequencing technology or incorrect normalization can propagate errors into the recovered gene set
- SIMS has only been validated for label transfer within the same species; trans-species gene-importance interpretation remains untested
- Explain-value stability must be assessed against the dispersion index; a single explain run is insufficient to distinguish stable selection from run-to-run artifacts

## Evidence

- [readme] Data normalization for model ingestion: [evidence span withheld — anchor status: external_doc]
- [readme] SIMS explainability method: [evidence span withheld — anchor status: external_doc]
