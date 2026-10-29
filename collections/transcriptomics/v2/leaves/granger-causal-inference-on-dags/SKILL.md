---
name: granger-causal-inference-on-dags
description: Use when inferring causal noncoding locus-gene relationships from paired
  scRNA-seq/scATAC-seq (or other multimodal single-cell) data undergoing a dynamic
  process, where a DAG of cell states can be oriented by pseudotime.
when_to_use_negative:
- Data lack a dynamic biological process (e.g., differentiation or disease trajectory),
  since the method relies on multiomic single-cell data undergoing a dynamic process
  to provide pseudotemporal context
- Only a single modality is available, or you need per-locus resolution consolidated
  into a single per-gene score (MultiVelo-style analysis) rather than pair-level Granger
  causal tests
- The question is purely correlational peak-gene association where no causal time-lag
  modeling is needed
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GrID-Net
  role: 'Core framework: lagged message-passing GNNs perform Granger causal inference
    on the cell DAG via reduced/full forecasting models and one-tailed F-tests'
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint multimodal cell representations unifying RNA and ATAC modalities,
    used to infer pseudotime and orient the kNN DAG edges
  repo: https://github.com/rs239/schema
- name: PyTorch
  role: Implementation backend for the GNN forecasting models, trained on a single
    NVIDIA Tesla V100 GPU
provenance:
  source_task_ids:
  - task_007
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:24cad632acca58b101af95ffbe430cd12af3d04ecdf1152fe3accc1f93008191
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# granger-causal-inference-on-dags

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

GrID-Net (Granger Inference on DAGs) generalizes Granger causality from total orderings to partial orderings via lagged message-passing GNNs, quantifying causal peak-gene links in single-cell multimodal (RNA+ATAC) data on a DAG of cell states. It addresses 'cell-state parallax' — the causal time lag between chromatin accessibility and gene expression that correlational methods miss.

## When to use

Use when inferring causal noncoding locus-gene relationships from paired scRNA-seq/scATAC-seq (or other multimodal single-cell) data undergoing a dynamic process, where a DAG of cell states can be oriented by pseudotime.

## When NOT to use

- Data lack a dynamic biological process (e.g., differentiation or disease trajectory), since the method relies on multiomic single-cell data undergoing a dynamic process to provide pseudotemporal context
- Only a single modality is available, or you need per-locus resolution consolidated into a single per-gene score (MultiVelo-style analysis) rather than pair-level Granger causal tests
- The question is purely correlational peak-gene association where no causal time-lag modeling is needed

## Inputs

- Unprocessed paired AnnData objects: RNA-seq (genes annotated with chr_no, txstart TSS columns) and ATAC-seq (peaks annotated with chr_no, start, end columns)
- Candidate peak-gene pairs table from gridnet.identify_all_peak_gene_link_candidates (distance_thresh=1e6)
- Root cell index or root cell-type marker gene (e.g., 'TOP2A') for pseudotime inference
- DAG adjacency matrix (N x N) with rows/columns ordered to match observation rows of X and Y

## Outputs

- DataFrame of candidate peak-gene pairs with one-tailed F-test statistics and p-values quantifying Granger causal link strength
- Ranked peak-gene link table (e.g., artifact_peakgene_links.csv)

## How to apply

Preprocess paired RNA/ATAC AnnData objects with gridnet.preprocess_multimodal (count normalization and log(1+x)), then assemble candidate peak-gene pairs with gridnet.identify_all_peak_gene_link_candidates (peaks within 1 Mb of gene TSS; 250 kb for 10x Multiome; genes expressed in >=1% of cells, peaks accessible in >=0.1%). Learn joint cell representations with Schema (gridnet.schema_representations), build a kNN graph (k=15) oriented by diffusion pseudotime via gridnet.construct_dag to form a DAG, then run gridnet.run_gridnet: two lagged message-passing GNN forecasting models (reduced using only the target's own past; full including the candidate cause) are trained with L=10 layers, Adam (lr=0.001), mini-batch size 1024 pairs, up to 20 epochs in PyTorch. Each candidate pair's causal strength is evaluated with a one-tailed F-test comparing full-model forecasting improvement over the reduced model; results are ranked by Granger causal strength.

## Related tools

- **GrID-Net** (Core framework: lagged message-passing GNNs perform Granger causal inference on the cell DAG via reduced/full forecasting models and one-tailed F-tests) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint multimodal cell representations unifying RNA and ATAC modalities, used to infer pseudotime and orient the kNN DAG edges) — https://github.com/rs239/schema
- **PyTorch** (Implementation backend for the GNN forecasting models, trained on a single NVIDIA Tesla V100 GPU)

## Examples

```
candidates_df = gridnet.identify_all_peak_gene_link_candidates(rna_adata,atac_adata,distance_thresh=1e6); dag = gridnet.construct_dag(X_joint,iroot,n_neighbors=15,pseudotime_algo='dpt'); results_df = gridnet.run_gridnet(X,Y,X_names,Y_names,candidate_XY_pairs,dag)
```

## Evaluation signals

- Output table schema contains peak, gene, F-test statistic, and p-value columns; recomputed F-test statistics on a small held-out candidate set match reported values
- AUROC of eQTL retrieval (against significant skin eQTLs at P < 1e-3 from eQTL Catalogue, combined with log genomic distance) exceeds every linear-Granger configuration (full trajectory or individual cortex/medulla/IRS branches)
- On the SCZ analysis, distance-augmented GrID-Net scores yield 132 candidate SCZ genes, 9/12 agreement on shared variants, and PP>50% variant coverage of 30/78 (38%) vs 22/78 (28%) for nearest-gene assignment
- DAG adjacency matrix is acyclic and row-ordered consistently with the rows of X and Y

## Limitations

- Inferred locus-gene associations are causal hypotheses that need perturbation-based validation; eQTL cross-referencing only mimics such validation
- Reliance on multiomic single-cell data from matching tissue undergoing a dynamic process; corticogenesis data may explain only a subset of SCZ phenotypes
- Computational burden of large single-cell datasets may exceed GPU memory capacity (sketching with Geosketch or Hopper is recommended)
- Inferred causal interactions are likely indirect and mediated by additional biophysical events such as transcription factor binding; no Hi-C/3D genome or CTCF topology information is incorporated
- GrID-Net mathematical formulation is deferred to Supplementary Information

## Evidence

- [intro] GrID-Net scores candidate causal links by evaluating each pair with a one-tailed F-test: [evidence span withheld — anchor status: not_found]
- [discussion] GNN training hyperparameters: [evidence span withheld — anchor status: not_found]
