---
name: single-cell-multiomic-preprocessing
description: Use when raw unprocessed RNA-seq and ATAC-seq AnnData objects from a
  paired single-cell multimodal assay must be normalized and filtered to define valid
  candidate peak-gene pairs for downstream GrID-Net or pseudotime analysis.
when_to_use_negative:
- Data are already normalized and only a single modality (e.g., scRNA-seq alone) is
  available — pseudotime-oriented causal inference requires paired RNA+ATAC observations
  of the same cells
- The goal is linking variants to genes without single-cell multimodal data, since
  preprocessing and candidate-pair assembly presuppose peak coordinates and TSS annotations
- The dataset is static or lacks a dynamic process (e.g., differentiation), as the
  method relies on pseudotemporal context for the cell-state DAG
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3168
tools:
- name: GrID-Net
  role: Provides preprocess_multimodal and identify_all_peak_gene_link_candidates
    for normalization, log(1+x) transformation, and distance-based candidate peak-gene
    pair assembly
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Consumes the preprocessed RNA and ATAC AnnData objects to learn joint multimodal
    cell representations for downstream pseudotime and DAG construction
  repo: https://github.com/rs239/schema
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:416bff0c854d5bb6c2561c986c1abaf6e4e76e4122c4efa0f1946a58c339139a
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-multiomic-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess paired single-cell RNA-seq and ATAC-seq multimodal data (e.g., 10x Multiome human corticogenesis) with count normalization, log(1+x) transformation, and expression/accessibility filtering before causal peak-gene link inference with GrID-Net.

## When to use

Use when raw unprocessed RNA-seq and ATAC-seq AnnData objects from a paired single-cell multimodal assay must be normalized and filtered to define valid candidate peak-gene pairs for downstream GrID-Net or pseudotime analysis.

## When NOT to use

- Data are already normalized and only a single modality (e.g., scRNA-seq alone) is available — pseudotime-oriented causal inference requires paired RNA+ATAC observations of the same cells
- The goal is linking variants to genes without single-cell multimodal data, since preprocessing and candidate-pair assembly presuppose peak coordinates and TSS annotations
- The dataset is static or lacks a dynamic process (e.g., differentiation), as the method relies on pseudotemporal context for the cell-state DAG

## Inputs

- Unprocessed paired RNA-seq AnnData object with gene TSS annotations (chr_no, txstart) in adata.var
- Unprocessed paired ATAC-seq AnnData object with peak coordinates (chr_no, start, end) in adata.var
- distance_thresh parameter (e.g., 250,000 bp for 10x Multiome; default 1e6 bp)
- rna_filter_gene_percent (default 1% of cells)
- atac_filter_peak_percent (default 0.1% of cells)

## Outputs

- Normalized log(1+x)-transformed RNA and ATAC AnnData objects in place
- candidates_df: DataFrame of candidate peak-gene pairs passing distance and cell-frequency filters

## How to apply

Load the unprocessed RNA-seq and ATAC-seq AnnData objects and run gridnet.preprocess_multimodal to normalize counts with a log(1+x) transformation. Then assemble candidate peak-gene pairs with gridnet.identify_all_peak_gene_link_candidates, restricting peaks to within 250 kb of a gene's TSS for the 10x Multiome dataset (default distance_thresh is 1 Mb for other assays). Filter out genes expressed in fewer than 1% of cells (rna_filter_gene_percent) and peaks accessible in fewer than 0.1% of cells (atac_filter_peak_percent). The RNA AnnData must carry gene TSS annotations (chr_no, txstart) in adata.var, and the ATAC AnnData must carry peak coordinates (chr_no, start, end) in adata.var. The output candidate pair DataFrame feeds Schema integration, DAG construction, and gridnet.run_gridnet.

## Related tools

- **GrID-Net** (Provides preprocess_multimodal and identify_all_peak_gene_link_candidates for normalization, log(1+x) transformation, and distance-based candidate peak-gene pair assembly) — https://github.com/alexw16/gridnet
- **Schema** (Consumes the preprocessed RNA and ATAC AnnData objects to learn joint multimodal cell representations for downstream pseudotime and DAG construction) — https://github.com/rs239/schema

## Examples

```
import gridnet; gridnet.preprocess_multimodal(rna_adata, atac_adata); candidates_df = gridnet.identify_all_peak_gene_link_candidates(rna_adata, atac_adata, distance_thresh=250000)
```

## Evaluation signals

- Genes retained are expressed in >=1% of cells and peaks retained are accessible in >=0.1% of cells after filtering
- All candidate peak-gene pairs have peak centers within the specified genomic distance of a gene TSS (250 kb for 10x Multiome; 1 Mb default)
- RNA adata.var contains chr_no and txstart columns; ATAC adata.var contains chr_no, start, and end columns
- candidates_df is a DataFrame with atac_id and gene columns usable directly as candidate_XY_pairs for gridnet.run_gridnet
- Downstream DAG construction with n_neighbors=15 and 'dpt' pseudotime succeeds on the preprocessed joint Schema representations

## Limitations

- Large single-cell datasets may impose computational burden exceeding GPU memory capacity for downstream GrID-Net training
- The method relies on multiomic single-cell data from matching tissue undergoing a dynamic biological process to provide pseudotemporal context
- GrID-Net's mathematical formulation is deferred to Supplementary Information, so preprocessing edge-case behavior may not be fully documented in the main text

## Evidence
