---
name: ftest-model-comparison-statistics
description: Use when ranking candidate causal peak-gene pairs by Granger causal strength
  from paired scATAC-seq/scRNA-seq data on a cell-state DAG, or when quantifying forecasting
  improvement of a full model over a reduced model per candidate pair.
when_to_use_negative:
- Input lacks a DAG or partial ordering over observations — the F-test on forecasting
  improvement requires DAG-structured dynamical systems, not total orderings or static
  correlation tables
- Genes are expressed in fewer than 1% of cells or peaks accessible in fewer than
  0.1% of cells — candidates are filtered out before evaluation
- The data are static and observational without pseudotemporal context — the article
  notes GrID-Net relies on multiomic data from a dynamic process such as differentiation
  to provide the pseudotemporal context
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: GrID-Net
  role: Provides gridnet.run_gridnet, which trains reduced/full lagged message-passing
    GNN forecasting models and computes the one-tailed F-test statistics quantifying
    Granger causal strength per peak-gene pair
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Generates joint RNA+ATAC cell representations used to build the kNN DAG over
    which the forecasting models are fit
  repo: https://github.com/rs239/schema
- name: PyTorch
  role: Implementation framework for the GNN forecasting models compared by the F-test
- name: NVIDIA Tesla V100 GPU
  role: Hardware on which the GrID-Net forecasting models are trained
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:0df4b2d0aecf764ce9a5f94ace17c4c128e334afeb1d958f39fa858260941fda
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/alexw16/gridnet
  repo_url: https://github.com/alexw16/gridnet
  tools:
  - GrID-Net
  - NVIDIA Tesla V100 GPU
  - PyTorch
  - Schema
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# ftest-model-comparison-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies a one-tailed F-test comparing a full forecasting model (which includes the candidate cause) against a reduced model (which uses only the effect's own past) to quantify Granger causal strength, as implemented in GrID-Net's lagged message-passing GNN framework for peak-gene links on a single-cell DAG.

## When to use

Use when ranking candidate causal peak-gene pairs by Granger causal strength from paired scATAC-seq/scRNA-seq data on a cell-state DAG, or when quantifying forecasting improvement of a full model over a reduced model per candidate pair.

## When NOT to use

- Input lacks a DAG or partial ordering over observations — the F-test on forecasting improvement requires DAG-structured dynamical systems, not total orderings or static correlation tables
- Genes are expressed in fewer than 1% of cells or peaks accessible in fewer than 0.1% of cells — candidates are filtered out before evaluation
- The data are static and observational without pseudotemporal context — the article notes GrID-Net relies on multiomic data from a dynamic process such as differentiation to provide the pseudotemporal context

## Inputs

- Peak accessibility matrix X (N x C numpy array) from ATAC-seq, with peak names (X_feature_names)
- Gene expression matrix Y (N x T numpy array) from RNA-seq, with gene names (Y_feature_names)
- Candidate peak-gene pairs as (atac_id, gene) tuples, e.g. from gridnet.identify_all_peak_gene_link_candidates(rna_adata, atac_adata, distance_thresh=1e6)
- DAG adjacency matrix (N x N) of cells, rows/columns ordered consistently with X and Y (e.g. gridnet.construct_dag with k=15 kNN, diffusion pseudotime edge orientation)

## Outputs

- Ranked results DataFrame annotating candidate peak-gene pairs with one-tailed F-test statistics and p-values (e.g. artifact_peakgene_links.csv with columns peak, gene, F-test statistic, p-value)

## How to apply

Train both a reduced and a full GrID-Net forecasting model per candidate pair: the full model includes the candidate cause (peak) while the reduced model uses only the effect's (gene's) own past. GrID-Net models consist of L=10 GNN layers trained with the Adam optimizer at learning rate 0.001, mini-batch size 1024 candidate peak-gene pairs, up to 20 epochs, implemented in PyTorch on an NVIDIA Tesla V100 GPU. Evaluate each candidate pair with a one-tailed F-test on the forecasting improvement of the full model over the reduced model; candidates are peaks within 1 Mb of a gene's TSS (distance_thresh=1e6) filtered by rna_filter_gene_percent=1% and atac_filter_peak_percent=0.1%. Run gridnet.run_gridnet with the peak accessibility matrix X (N x C), gene expression matrix Y (N x T), feature-name lists, candidate_XY_pairs as (atac_id, gene) tuples, and the DAG adjacency matrix ordered consistently with the rows of X and Y. Rank peak-gene pairs by the resulting Granger causal strength statistics.

## Related tools

- **GrID-Net** (Provides gridnet.run_gridnet, which trains reduced/full lagged message-passing GNN forecasting models and computes the one-tailed F-test statistics quantifying Granger causal strength per peak-gene pair) — https://github.com/alexw16/gridnet
- **Schema** (Generates joint RNA+ATAC cell representations used to build the kNN DAG over which the forecasting models are fit) — https://github.com/rs239/schema
- **PyTorch** (Implementation framework for the GNN forecasting models compared by the F-test)
- **NVIDIA Tesla V100 GPU** (Hardware on which the GrID-Net forecasting models are trained)

## Examples

```
results_df = gridnet.run_gridnet(X, Y, X_feature_names, Y_feature_names, candidate_XY_pairs, dag_adjacency_matrix)
```

## Evaluation signals

- Output table schema contains peak, gene, F-test statistic, and p-value columns
- Recomputing F-test statistics on a small held-out candidate set matches the reported values
- Every evaluated (atac_id, gene) tuple corresponds to a peak within 1 Mb of the gene's TSS that passed the 1% gene-expression and 0.1% peak-accessibility filters
- Rows of the DAG adjacency matrix are ordered consistently with the rows of X and Y
- Resulting ranking of pairs by Granger causal strength is deterministic given a fixed DAG and seed, and full-model improvement over the reduced model is non-negative per the one-tailed test

## Limitations

- The GrID-Net mathematical formulation is deferred to Supplementary Information and not given in the main text
- Inferred causal associations are hypotheses requiring perturbation-based validation, which was only mimicked via eQTL cross-referencing; inferred interactions are likely indirect and mediated by additional biophysical events such as transcription factor binding
- Computational burden of large single-cell datasets may exceed GPU memory capacity

## Evidence

- [intro] one-tailed F-test on forecasting improvement: [evidence span withheld — anchor status: not_found]
- [discussion] GNN training hyperparameters: [evidence span withheld — anchor status: not_found]
