---
name: gwas-variant-peak-genomic-intersection
description: Use when fine-mapped GWAS variants with posterior probabilities (e.g.,
  Trubetskoy SCZ FINEMAP PP >= 0.05) must be intersected with scATAC-seq peaks and
  linked to candidate regulated genes beyond nearest-gene assignment.
when_to_use_negative:
- Variants lack fine-mapping posterior probabilities (only lead-SNP p-values), so
  the PP >= 0.05 overlap filter cannot be applied faithfully
- The multimodal single-cell data come from a static tissue rather than a dynamic
  process, so no pseudotemporal cell DAG exists to support Granger inference
- The goal is a nearest-gene baseline comparison, which should be run separately rather
  than replaced by this distance-augmented linking
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0622
tools:
- name: GrID-Net
  role: Computes Granger causality test statistics for candidate peak-gene pairs;
    these scores plus log distance feed the logistic model used to link variants to
    genes
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint RNA+ATAC cell representations from the 10x Multiome corticogenesis
    data used to build the kNN cell DAG for GrID-Net
  repo: https://github.com/rs239/schema
- name: PyTorch
  role: Framework in which the 10-layer GrID-Net GNN is implemented and trained (Adam,
    lr 0.001, mini-batch 1024)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:9f5b50f03c64d4a5f22fd4dbd55f1b3ff87404b4580abb43d9821f967fb722d7
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
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gwas-variant-peak-genomic-intersection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Intersects fine-mapped GWAS variants (FINEMAP PP >= 0.05) with scATAC-seq peaks from a multimodal dataset and links each variant-overlapping peak to up to 3 genes using distance-augmented GrID-Net Granger scores, replacing nearest-gene assignment; applied to Trubetskoy et al. SCZ variants this yielded 132 candidate genes.

## When to use

Use when fine-mapped GWAS variants with posterior probabilities (e.g., Trubetskoy SCZ FINEMAP PP >= 0.05) must be intersected with scATAC-seq peaks and linked to candidate regulated genes beyond nearest-gene assignment.

## When NOT to use

- Variants lack fine-mapping posterior probabilities (only lead-SNP p-values), so the PP >= 0.05 overlap filter cannot be applied faithfully
- The multimodal single-cell data come from a static tissue rather than a dynamic process, so no pseudotemporal cell DAG exists to support Granger inference
- The goal is a nearest-gene baseline comparison, which should be run separately rather than replaced by this distance-augmented linking

## Inputs

- Fine-mapped GWAS variant table with posterior probabilities (e.g., Trubetskoy et al. SCZ FINEMAP variants, PP >= 0.05, GRCh38/hg38 coordinates)
- Unprocessed RNA-seq and ATAC-seq AnnData objects from the 10x Multiome human corticogenesis dataset, with gene TSS annotations (chr_no, txstart) and peak coordinates (chr_no, start, end) in adata.var
- SHARE-seq mouse skin eQTL peak-gene pairs (P < 1e-3) used as positive labels to calibrate the distance-weighted scoring model
- Trained GrID-Net peak-gene score table (Granger causality test statistics per candidate pair)

## Outputs

- Set of fine-mapped variants overlapping scATAC-seq peaks
- Variant-gene link table (variant_gene_links.csv) mapping each variant to up to 3 highest-scoring genes from top-50% distance-augmented peak-gene pairs
- Candidate SCZ gene set (132 genes in the paper, including KCNG2 and SLC12A6) with GWAS enrichment statistics (FDR = 0.037)

## How to apply

Retain all fine-mapped variants with FINEMAP PP >= 0.05 that overlap peaks from the scATAC-seq data (here, the 10x Multiome human corticogenesis dataset), after filtering peaks accessible in >= 0.1% of cells and genes expressed in >= 1% of cells, with candidate peak-gene pairs restricted to 250 kb around each TSS for 10x Multiome (default 1 Mb). Build the cell DAG from Schema-integrated RNA+ATAC representations (kNN k=15, diffusion pseudotime) and run GrID-Net (10-layer GNN, Adam, lr 0.001, mini-batch 1024) to score every candidate peak-gene pair by Granger causal strength. Then fit a logistic regression predicting SHARE-seq mouse skin eQTL peak-gene pairs (P < 1e-3) from GrID-Net score plus log genomic distance, so the score captures distance-dependent regulatory effects. Score the genes of each variant-overlapping peak with this model, retain only pairs in the top 50% of all peak-gene scores, and link each variant to up to 3 of its highest-scoring genes; write the result to variant_gene_links.csv. The rationale is that nearest-gene assignment overlooks distal locus-gene interactions, which the Granger-based, distance-augmented score recovers.

## Related tools

- **GrID-Net** (Computes Granger causality test statistics for candidate peak-gene pairs; these scores plus log distance feed the logistic model used to link variants to genes) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint RNA+ATAC cell representations from the 10x Multiome corticogenesis data used to build the kNN cell DAG for GrID-Net) — https://github.com/rs239/schema
- **PyTorch** (Framework in which the 10-layer GrID-Net GNN is implemented and trained (Adam, lr 0.001, mini-batch 1024))

## Examples

```
candidates_df = gridnet.identify_all_peak_gene_link_candidates(rna_adata, atac_adata, distance_thresh=250000)
```

## Evaluation signals

- Reproduce the reported 132 candidate SCZ-linked genes when Trubetskoy fine-mapped variants are intersected with 10x Multiome peaks
- Check variant_gene_links.csv assigns at most 3 genes per variant, drawn only from peak-gene pairs in the top 50% of distance-augmented scores
- Confirm the resulting gene set is enriched for genes implicated in the original SCZ GWAS (FDR = 0.037)
- Verify coverage of PP > 50% variants reaches 30/78 (38%) versus 22/78 (28%) for baseline nearest-gene assignment, with 9/12 agreement on shared variants
- Sanity-check that only variants with PP >= 0.05 overlapping filtered ATAC peaks (accessible in >= 0.1% of cells) enter the link table

## Limitations

- Relies on multiomic single-cell data from matching tissue undergoing a dynamic process; the 10x Multiome corticogenesis data may explain only a subset of SCZ phenotypes
- Inferred locus-gene associations are causal hypotheses that need perturbation-based validation; only mimicked via eQTL cross-referencing
- Inferred causal interactions are likely indirect and mediated by additional biophysical events such as transcription factor binding
- Computational burden of large single-cell datasets may exceed GPU memory capacity

## Evidence

- [discussion] We retained all fine-mapped variants with PP≥0.05 that overlapped peaks from the 10x Multiome human corticogenesis dataset.: [evidence span withheld — anchor status: not_found]
