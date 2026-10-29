---
name: peak-gene-candidate-pair-filtering
description: Use when preprocessing scRNA-seq + scATAC-seq multiomic AnnData objects
  before GrID-Net-style causal inference, to define the bounded set of peak-gene pairs
  that will be scored.
when_to_use_negative:
- The AnnData objects lack TSS or peak-coordinate annotations in adata.var, so genomic
  distance cannot be computed
- The goal is all-vs-all gene regulatory inference without genomic proximity constraints
  (pair filtering by 1 Mb TSS distance would be inappropriate)
- The data are not paired multimodal single-cell measurements of the same cells (e.g.
  bulk ATAC + bulk RNA)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: GrID-Net
  role: Provides preprocess_multimodal and identify_all_peak_gene_link_candidates
    with rna_filter_gene_percent, atac_filter_peak_percent, and distance_thresh parameters
    that implement the filtering
  repo: https://github.com/alexw16/gridnet
- name: scanpy
  role: Used downstream for diffusion pseudotime on the kNN graph built from Schema
    joint representations (scanpy implementation of diffusion pseudotime)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:6cfbdbc140ce3d2bfc59749ddfe46eb49cb9cd79b8006aa0bf643eb01d2b0bd6
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
  - scanpy
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# peak-gene-candidate-pair-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assemble and filter the candidate peak-gene pair universe for causal regulatory inference from single-cell multimodal RNA+ATAC data, by removing genes expressed in <1% of cells, peaks accessible in <0.1% of cells, and restricting pairs to peaks within 1 Mb (250 kb for 10x Multiome) of a gene's TSS.

## When to use

Use when preprocessing scRNA-seq + scATAC-seq multiomic AnnData objects before GrID-Net-style causal inference, to define the bounded set of peak-gene pairs that will be scored.

## When NOT to use

- The AnnData objects lack TSS or peak-coordinate annotations in adata.var, so genomic distance cannot be computed
- The goal is all-vs-all gene regulatory inference without genomic proximity constraints (pair filtering by 1 Mb TSS distance would be inappropriate)
- The data are not paired multimodal single-cell measurements of the same cells (e.g. bulk ATAC + bulk RNA)

## Inputs

- Preprocessed RNA-seq AnnData with gene TSS annotations (chr_no, txstart columns in adata.var)
- Preprocessed ATAC-seq AnnData with peak coordinates (chr_no, start, end columns in adata.var)
- distance_thresh parameter (default 1e6 bp; 250 kb for 10x Multiome)
- rna_filter_gene_percent (default 1%) and atac_filter_peak_percent (default 0.1%) thresholds

## Outputs

- candidates_df: DataFrame of candidate peak-gene pairs passing distance and prevalence filters, with atac_id and gene columns
- candidate_XY_pairs: list of (peak, gene) tuples feeding run_gridnet and baseline methods

## How to apply

First normalize counts and apply log(1+x) transformation to both modalities with gridnet.preprocess_multimodal. Then filter low-prevalence features: drop genes represented in fewer than 1% of cells and peaks represented in fewer than 0.1% of cells, implemented via the rna_filter_gene_percent and atac_filter_peak_percent parameters. Assemble candidate pairs with gridnet.identify_all_peak_gene_link_candidates using distance_thresh=1e6 (peaks whose centers lie within 1 Mb of the gene TSS); for the 10x Multiome human corticogenesis dataset, tighten this to 250 kb. The RNA AnnData must carry per-gene TSS annotations (chr_no, txstart in adata.var) and the ATAC AnnData per-peak coordinates (chr_no, start, end) so genomic distances can be computed. This filtered pair list defines the candidate_XY_pairs input to downstream scoring (run_gridnet) and must be applied identically when computing baseline scores (Pearson, Granger, GVAR, ABC) for fair benchmarking.

## Related tools

- **GrID-Net** (Provides preprocess_multimodal and identify_all_peak_gene_link_candidates with rna_filter_gene_percent, atac_filter_peak_percent, and distance_thresh parameters that implement the filtering) — https://github.com/alexw16/gridnet
- **scanpy** (Used downstream for diffusion pseudotime on the kNN graph built from Schema joint representations (scanpy implementation of diffusion pseudotime))

## Examples

```
gridnet.preprocess_multimodal(rna_adata, atac_adata); candidates_df = gridnet.identify_all_peak_gene_link_candidates(rna_adata, atac_adata, distance_thresh=1e6)
```

## Evaluation signals

- Every retained candidate pair has peak center within distance_thresh (1e6 bp, or 250e3 for 10x Multiome) of the gene TSS
- All retained genes are expressed in ≥1% of cells and all retained peaks are accessible in ≥0.1% of cells
- The output candidates_df contains the atac_id and gene columns expected by run_gridnet's candidate_XY_pairs input
- The same filtered candidate set is reused verbatim for baseline methods (Pearson, Linear Granger, GVAR, ABC) so AUROC comparisons are on identical pair sets
- Preprocessing output is log(1+x)-transformed normalized counts in both AnnData objects

## Limitations

- The 1 Mb / 250 kb distance cutoff ignores 3D genome structure (Hi-C, CTCF topology), which could refine or falsify candidate membership
- Filtering thresholds are defaults; datasets with sparse accessibility may lose true pairs below the 0.1% accessibility floor
- Requires paired multiomic data from a dynamic process for downstream pseudotime context; static tissue data yield weaker candidate evaluation

## Evidence

- [discussion] we assembled a set of candidate peak-gene pairs by selecting all peaks with centers within 1 Mb from each gene’s TSS. For the 10x Multiome dataset, we only considered peak-gene pairs that were: [evidence span withheld — anchor status: not_found]
