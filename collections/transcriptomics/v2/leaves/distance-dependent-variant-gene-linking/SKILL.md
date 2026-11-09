---
name: distance-dependent-variant-gene-linking
description: Use when fine-mapped noncoding GWAS variants (with posterior probabilities)
  overlap scATAC-seq peaks and you must assign distal regulatory target genes beyond
  the nearest gene, accounting for distance-dependent effects.
when_to_use_negative:
- No pseudotemporal/dynamic multiomic context available — GrID-Net requires multiomic
  single-cell data from a matching tissue undergoing a dynamic process (e.g., differentiation)
  to orient the cell DAG
- Only nearest-gene assignment is needed and distal interactions are out of scope,
  or variants do not overlap any accessible ATAC peaks in the reference dataset
- Input is already a curated gene list with no variant-level or peak-level resolution
  to score
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: GrID-Net
  role: Produces Granger causality test statistics for candidate peak-gene pairs that
    feed the distance-weighted logistic regression
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint multimodal cell representations (RNA + ATAC) used to build the
    cell DAG for Granger inference
  repo: https://github.com/rs239/schema
- name: PyTorch
  role: Implementation backend for the 10-layer GrID-Net GNN models (Adam, lr 0.001,
    batch size 1024)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:936eeb4b363e5308d42569894abef7904126f766e3680272b8209721b9c81cfe
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# distance-dependent-variant-gene-linking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Links fine-mapped GWAS variants to the genes they putatively regulate by combining GrID-Net Granger causal peak-gene scores with a genomic-distance-aware logistic regression, replacing naive nearest-gene assignment. Applied in Wu et al. 2025 to Trubetskoy et al. fine-mapped SCZ variants to yield 132 candidate SCZ genes.

## When to use

Use when fine-mapped noncoding GWAS variants (with posterior probabilities) overlap scATAC-seq peaks and you must assign distal regulatory target genes beyond the nearest gene, accounting for distance-dependent effects.

## When NOT to use

- No pseudotemporal/dynamic multiomic context available — GrID-Net requires multiomic single-cell data from a matching tissue undergoing a dynamic process (e.g., differentiation) to orient the cell DAG
- Only nearest-gene assignment is needed and distal interactions are out of scope, or variants do not overlap any accessible ATAC peaks in the reference dataset
- Input is already a curated gene list with no variant-level or peak-level resolution to score

## Inputs

- Fine-mapped GWAS variant list with FINEMAP posterior probabilities (SCZ: PP >= 0.05 threshold)
- Unprocessed RNA-seq AnnData object with gene TSS annotations (chr_no, txstart in adata.var) from 10x Multiome human corticogenesis dataset
- Unprocessed ATAC-seq AnnData object with peak coordinates (chr_no, start, end in adata.var)
- Training peak-gene pairs with regulatory labels (SHARE-seq mouse skin eQTL peak-gene pairs, P < 1e-3)

## Outputs

- Variant-to-gene link table (variant_gene_links.csv) with up to 3 top-scoring genes per variant
- Set of candidate SCZ genes (132 genes in the reference analysis, enriched FDR = 0.037)
- Per-pair logistic model scores combining GrID-Net Granger statistics and log genomic distance

## How to apply

First run GrID-Net on a single-cell multimodal dataset (e.g., 10x Multiome human corticogenesis) to obtain Granger causality statistics for candidate peak-gene pairs, restricting peaks to within 250 kb of gene TSSs (1 Mb default elsewhere), keeping genes expressed in >=1% and peaks accessible in >=0.1% of cells after gridnet.preprocess_multimodal. Retain fine-mapped variants with FINEMAP PP >= 0.05 that overlap ATAC peaks. Fit a logistic regression predicting known peak-gene pairs (SHARE-seq mouse skin eQTLs, P < 1e-3) from the GrID-Net score plus log genomic distance, which captures distance-dependent regulatory effects. Score each variant-overlapping peak's genes with this model, keep only peak-gene pairs in the top 50% of scores, and link each variant to up to 3 of its highest-scoring genes. Output the variant-gene link table (e.g., variant_gene_links.csv).

## Related tools

- **GrID-Net** (Produces Granger causality test statistics for candidate peak-gene pairs that feed the distance-weighted logistic regression) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint multimodal cell representations (RNA + ATAC) used to build the cell DAG for Granger inference) — https://github.com/rs239/schema
- **PyTorch** (Implementation backend for the 10-layer GrID-Net GNN models (Adam, lr 0.001, batch size 1024))

## Examples

```
candidates_df = gridnet.identify_all_peak_gene_link_candidates(rna_adata, atac_adata, distance_thresh=250000); dag_adjacency_matrix = gridnet.construct_dag(X_joint, iroot, n_neighbors=15, pseudotime_algo='dpt'); results_df = gridnet.run_gridnet(X, Y, X_feature_names, Y_feature_names, candidate_XY_pairs, dag_adjacency_matrix)
```

## Evaluation signals

- Reproduce the reported headline statistic: 132 candidate SCZ-linked genes after linking fine-mapped variants through peak-gene pairs
- Check agreement on shared variants against the paper's baseline: 9/12 agreement expected
- Variant coverage should exceed nearest-gene baseline: 30/78 (38%) vs 22/78 (28%) of PP>50% variants assigned genes
- Top 50% score filter applied: verify exactly half of variant-overlapping peak-gene pairs are retained before the up-to-3-genes-per-variant link step
- Enrichment check: resulting gene set should be significantly enriched for genes implicated in the original SCZ GWAS (FDR = 0.037)

## Limitations

- Inferred locus-gene associations are causal hypotheses requiring perturbation-based validation; eQTL cross-referencing only mimics such validation
- Tissue matching is imperfect: the 10x Multiome corticogenesis dataset may explain only a subset of SCZ phenotypes
- Computational burden of large single-cell datasets may exceed GPU memory capacity (models trained on a single NVIDIA Tesla V100)
- GrID-Net's mathematical formulation is deferred to Supplementary Information

## Evidence

- [discussion] using the genomic distance-dependent GrID-Net scores, we linked the variant to up to 3 of the peak's highest-scoring genes, retaining only peak-gene pairs in the top 50% of all peak-gene scores.: [evidence span withheld — anchor status: not_found]
- [discussion] We retained all fine-mapped variants with PP≥0.05 that overlapped peaks from the 10x Multiome human corticogenesis dataset.: [evidence span withheld — anchor status: not_found]
