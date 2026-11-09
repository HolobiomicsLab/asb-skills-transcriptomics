---
name: single-cell-multiome-count-preprocessing
description: Use when raw paired RNA-seq + ATAC-seq single-cell multimodal counts
  (e.g. 10x Multiome, SHARE-seq, SNARE-seq, sci-CAR) must be normalized and filtered
  before peak-gene link analysis.
when_to_use_negative:
- The input data are already normalized, log-transformed, or feature-filtered — re-running
  preprocess_multimodal would double-transform the counts
- Only one modality is available (single-cell RNA-seq or ATAC-seq alone), since the
  function requires paired RNA and ATAC AnnData objects
- The goal is correlation-free pseudotime or trajectory analysis without peak-gene
  Granger inference, where alternate preprocessing conventions may be preferable
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0769
tools:
- name: GrID-Net
  role: Provides gridnet.preprocess_multimodal, which applies count normalization
    and log(1+x) transformation to the paired RNA and ATAC AnnData objects
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Consumes the preprocessed RNA and ATAC matrices to learn joint multimodal
    cell representations in the next pipeline step
  repo: https://github.com/rs239/schema
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:eef1f03795bc8498cda63abc3bcff94a99ccfbac0736b9a89fd150f4f44d9913
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
  - scanpy
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# single-cell-multiome-count-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalizes paired single-cell multiome count matrices (RNA-seq and ATAC-seq) with count normalization followed by log(1+x) transformation, then filters genes expressed in fewer than 1% of cells and peaks accessible in fewer than 0.1% of cells. This prepares AnnData objects for GrID-Net's multimodal integration, DAG construction, and Granger causal peak-gene inference.

## When to use

Use when raw paired RNA-seq + ATAC-seq single-cell multimodal counts (e.g. 10x Multiome, SHARE-seq, SNARE-seq, sci-CAR) must be normalized and filtered before peak-gene link analysis.

## When NOT to use

- The input data are already normalized, log-transformed, or feature-filtered — re-running preprocess_multimodal would double-transform the counts
- Only one modality is available (single-cell RNA-seq or ATAC-seq alone), since the function requires paired RNA and ATAC AnnData objects
- The goal is correlation-free pseudotime or trajectory analysis without peak-gene Granger inference, where alternate preprocessing conventions may be preferable

## Inputs

- Unprocessed AnnData object for RNA-seq (gene x cell counts; var must include chr_no and txstart TSS annotations)
- Unprocessed AnnData object for ATAC-seq (peak x cell counts; var must include chr_no, start, end peak coordinates)
- Filter thresholds: rna_filter_gene_percent=0.01 (1% of cells), atac_filter_peak_percent=0.001 (0.1% of cells)

## Outputs

- Normalized, log(1+x)-transformed RNA-seq AnnData filtered to genes expressed in ≥1% of cells
- Normalized, log(1+x)-transformed ATAC-seq AnnData filtered to peaks accessible in ≥0.1% of cells

## How to apply

Run gridnet.preprocess_multimodal(rna_adata, atac_adata), which performs count normalization and a log(1+x) transformation on both modalities in place. After normalization, remove genes represented in fewer than 1% of cells and peaks accessible in fewer than 0.1% of cells (matching the rna_filter_gene_percent=0.01 and atac_filter_peak_percent=0.001 defaults of the wrapper). Confirm that the RNA AnnData var contains gene TSS annotations (chr_no, txstart) and the ATAC AnnData var contains peak coordinates (chr_no, start, end), since downstream candidate peak-gene pair assembly (identify_all_peak_gene_link_candidates with distance_thresh=1e6, or 250 kb for 10x Multiome) depends on them. The preprocessed matrices feed Schema joint cell representations and gridnet.construct_dag (kNN k=15, diffusion pseudotime) in subsequent steps.

## Related tools

- **GrID-Net** (Provides gridnet.preprocess_multimodal, which applies count normalization and log(1+x) transformation to the paired RNA and ATAC AnnData objects) — https://github.com/alexw16/gridnet
- **Schema** (Consumes the preprocessed RNA and ATAC matrices to learn joint multimodal cell representations in the next pipeline step) — https://github.com/rs239/schema

## Examples

```
gridnet.preprocess_multimodal(rna_adata, atac_adata)
```

## Evaluation signals

- Both AnnData objects contain non-negative log-transformed values consistent with log(1+x) normalization (no raw integer counts remaining)
- Every retained gene is expressed in at least 1% of cells; every retained peak is accessible in at least 0.1% of cells
- The RNA AnnData var retains chr_no and txstart columns and the ATAC var retains chr_no, start, end columns required for candidate pair assembly
- RNA and ATAC AnnData share the same set of N cells in matched row order, so downstream X (ATAC) and Y (RNA) matrices are row-paired per observation
- Subsequent gridnet.schema_representations and gridnet.construct_dag (n_neighbors=15, pseudotime_algo='dpt') run without shape errors on the preprocessed objects

## Limitations

- GrID-Net relies on multiomic single-cell data from matching tissue undergoing a dynamic process (e.g. differentiation) to provide pseudotemporal context, so static or mismatched-tissue samples may preprocess fine but yield weak downstream causal inference
- Large single-cell datasets impose a computational burden that may exceed GPU memory capacity after preprocessing
- Default filter thresholds (1% genes, 0.1% peaks) and 1 Mb distance threshold (250 kb for 10x Multiome) are dataset-tuned; blindly applying them to sparse datasets may over-filter features

## Evidence
