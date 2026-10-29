---
name: granger-causal-inference-single-cell-multimodal
description: Use when you have paired scRNA-seq and scATAC-seq (e.g. 10x Multiome,
  SHARE-seq) from a dynamic process and need causal locus-gene links beyond correlation,
  or to validate/enrich enhancer-priming annotations of prioritized peak-gene pairs.
when_to_use_negative:
- Data are not paired multimodal single-cell measurements from a dynamic biological
  process — GrID-Net's DAG construction relies on multimodal Schema integration and
  diffusion pseudotime over a differentiating trajectory
- The goal is a purely correlational co-accessibility/co-expression screen without
  any causal ordering hypothesis, or cells lack a biologically meaningful pseudotemporal
  axis (e.g. static tissue snapshots)
- Dataset lacks a credible root cell type/marker gene, since pseudotime orientation
  of the DAG requires a root (e.g. TOP2A+ cycling progenitors)
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: GrID-Net
  role: 'Core framework: GNN with lagged message passing on the cell-state DAG that
    scores Granger causal peak-gene links via a one-tailed F-test'
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint RNA+ATAC cell representations used to build the kNN graph (k=15)
    and infer diffusion pseudotime that orients the DAG
  repo: https://github.com/rs239/schema
- name: PyTorch
  role: Implementation framework for GrID-Net models, trained on a single NVIDIA Tesla
    V100 GPU
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:af974ef8ffddae91b885b724ead0408c86034d465f6cbe6225e806a5277d70a3
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
  - PyTorch
  - Schema
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# granger-causal-inference-single-cell-multimodal

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply GrID-Net, a graph neural network framework for Granger causal inference on DAGs, to paired single-cell RNA-seq/ATAC-seq multimodal data to infer causal noncoding peak-gene regulatory links, exploiting pseudotemporal ordering to resolve the causal time lag ('cell-state parallax') that correlation-based methods miss.

## When to use

Use when you have paired scRNA-seq and scATAC-seq (e.g. 10x Multiome, SHARE-seq) from a dynamic process and need causal locus-gene links beyond correlation, or to validate/enrich enhancer-priming annotations of prioritized peak-gene pairs.

## When NOT to use

- Data are not paired multimodal single-cell measurements from a dynamic biological process — GrID-Net's DAG construction relies on multimodal Schema integration and diffusion pseudotime over a differentiating trajectory
- The goal is a purely correlational co-accessibility/co-expression screen without any causal ordering hypothesis, or cells lack a biologically meaningful pseudotemporal axis (e.g. static tissue snapshots)
- Dataset lacks a credible root cell type/marker gene, since pseudotime orientation of the DAG requires a root (e.g. TOP2A+ cycling progenitors)

## Inputs

- Unprocessed paired AnnData objects for scRNA-seq and scATAC-seq from the same cells (e.g. 10x Multiome human corticogenesis dataset)
- rna_adata.var with gene TSS annotations (chr_no, txstart)
- atac_adata.var with peak genomic coordinates (chr_no, start, end)
- Root cell specification: a root cell marker gene (e.g. 'TOP2A') or root cell index
- Optional: correlation-baseline ranked peak-gene pair lists and ChIP-seq peak coordinates (e.g. H3K4me1, H3K27ac from ChIP-Atlas) for downstream validation

## Outputs

- results_df: DataFrame of candidate peak-gene pairs annotated with Granger causality one-tailed F-test statistics
- Ranked peak-gene pairs by Granger causal strength for downstream locus-gene linking (e.g. fine-mapped GWAS variant assignment)
- Downstream validation artifacts such as a poised-to-active enhancer ratio table (e.g. enhancer_priming_ratios.csv) per method comparison

## How to apply

Preprocess the RNA and ATAC AnnData objects (count normalization, log(1+x) transformation), requiring gene TSS annotations (chr_no, txstart) in rna_adata.var and peak coordinates (chr_no, start, end) in atac_adata.var. Identify candidate peak-gene pairs with gridnet.identify_all_peak_gene_link_candidates using distance_thresh=1 Mb (250 kb for 10x Multiome), after filtering genes present in <1% of cells and peaks accessible in <0.1% of cells (rna_filter_gene_percent, atac_filter_peak_percent). Learn joint cell representations with Schema, build a kNN graph (k=15), infer pseudotime via diffusion pseudotime (scanpy implementation), and orient edges by increasing pseudotime to form a DAG via gridnet.construct_dag. Run gridnet.run_gridnet: a 10-layer GNN trained with Adam (lr=0.001, batch size 1024 pairs, up to 20 epochs) uses two forecasting models (reduced and full) and a one-tailed F-test on forecasting improvement to score each candidate peak-gene link. Rank pairs by Granger causal strength; optionally compare to correlation baselines and cross-reference histone marks (e.g. ChIP-Atlas H3K4me1/H3K27ac) to validate chromatin priming of differentially prioritized pairs.

## Related tools

- **GrID-Net** (Core framework: GNN with lagged message passing on the cell-state DAG that scores Granger causal peak-gene links via a one-tailed F-test) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint RNA+ATAC cell representations used to build the kNN graph (k=15) and infer diffusion pseudotime that orients the DAG) — https://github.com/rs239/schema
- **PyTorch** (Implementation framework for GrID-Net models, trained on a single NVIDIA Tesla V100 GPU)

## Examples

```
import gridnet; results_df = gridnet.gridnet_multimodal(rna_adata, atac_adata, root_cell_marker_gene='TOP2A')
```

## Evaluation signals

- results_df exists and covers the expected candidate_XY_pairs derived from identify_all_peak_gene_link_candidates at the chosen distance_thresh (1e6 default; 250 kb for 10x Multiome)
- Every scored pair has a finite one-tailed F-test statistic from the reduced-vs-full forecasting model comparison
- In eQTL cross-validation, GrID-Net prioritized pairs enrich for eQTL-overlapping peaks relative to correlation baselines; on SCZ variants, variant coverage increases by 36% over nearest-gene approaches
- When performing the enhancer-priming validation, GrID-Net-differentiated pairs show a higher poised (H3K4me1-only) to active (H3K4me1+H3K27ac) enhancer ratio than every correlation-method pair set, consistent with Fig. 2f
- DAG sanity: adjacency matrix is acyclic, row/column order matches X and Y observation order, and edges point along increasing pseudotime

## Limitations

- Inferred associations are causal hypotheses requiring perturbation validation; eQTL cross-referencing only mimics such validation
- Relies on multiomic single-cell data from a matching tissue undergoing a dynamic process to provide pseudotemporal context; tissue mismatch limits interpretability (corticogenesis data may explain only a subset of SCZ phenotypes)
- Computational burden on large single-cell datasets may exceed GPU memory capacity
- Inferred causal interactions are likely indirect and mediated by additional biophysical events such as TF binding
- No Hi-C/3D genome structure or CTCF topology information is incorporated

## Evidence

- [methods] The strength of the Granger causal link between x and y is evaluated by a one-tailed F-test: [evidence span withheld — anchor status: not_found]
- [discussion] All GrID-Net models consisted of L=10 GNN layers and were trained using the Adam optimizer with a learning rate of 0.001 and a mini-batch size of 1024 candidate peak-gene pairs.: [evidence span withheld — anchor status: not_found]
