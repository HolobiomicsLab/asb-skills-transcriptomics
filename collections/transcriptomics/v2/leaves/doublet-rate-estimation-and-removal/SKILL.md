---
name: doublet-rate-estimation-and-removal
description: Use when preprocessing raw scRNA-seq data with scKB/COPILOT and quality-filtered
  gene-by-cell matrices must be purged of doublets without manually specifying a doublet
  rate.
when_to_use_negative:
- Input is already a quality-filtered matrix produced by Cell Ranger or another tool
  rather than raw spliced/unspliced scKB matrices
- The user has a trusted externally computed doublet rate and only needs rate estimation
  toggled off via doublet.rate=NULL semantics unrelated to this step
- Running cell type/developmental stage annotation on non-Arabidopsis data, since
  COPILOT annotation only supports Arabidopsis thaliana root
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: COPILOT
  role: Runs quality filtering (mt.threshold=5, filtering.ratio=1, top.percent=1,
    min.UMI thresholds 100/300) and performs doublet rate estimation and removal via
    estimate.doublet.rate and remove.doublet arguments
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: DoubletFinder
  role: R package dependency used by COPILOT to detect and remove doublets after gene
    and cell quality filtering
- name: scKB
  role: Produces the raw spliced and unspliced count matrices from FASTQs via kallisto/bustools
    that feed COPILOT's filtering and doublet removal
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:352cd4ccbb44f8dd7af0382bdde3b32f1d3e0b6d2aa7e8eeeb9f7e5a0555a246
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  repo_url: https://github.com/Hsu-Che-Wei/COPILOT
  tools:
  - BSgenome
  - COPILOT
  - DoubletFinder
  - bustools
  - scKB
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# doublet-rate-estimation-and-removal

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates the expected doublet rate from the 10X Genomics loading estimation and removes heterotypic/homotypic doublets from gene-by-cell UMI matrices during COPILOT quality filtering, using DoubletFinder downstream of the noise-model-based cell filtering.

## When to use

Use when preprocessing raw scRNA-seq data with scKB/COPILOT and quality-filtered gene-by-cell matrices must be purged of doublets without manually specifying a doublet rate.

## When NOT to use

- Input is already a quality-filtered matrix produced by Cell Ranger or another tool rather than raw spliced/unspliced scKB matrices
- The user has a trusted externally computed doublet rate and only needs rate estimation toggled off via doublet.rate=NULL semantics unrelated to this step
- Running cell type/developmental stage annotation on non-Arabidopsis data, since COPILOT annotation only supports Arabidopsis thaliana root

## Inputs

- Raw spliced gene-by-cell UMI count matrix from scKB (column- and row-named)
- Raw unspliced gene-by-cell UMI count matrix from scKB
- Mitochondrial gene list/pattern supplied via mt.pattern (e.g. genes grepped from chrM in genes.gtf)

## Outputs

- Quality-filtered spliced and unspliced count matrices with doublets removed (written to filtered.mtx.output.dir)
- Seurat object with normalized, PCA-, UMAP- and cluster-analyzed filtered cells (do.seurat=TRUE)
- COPILOT summary file reporting doublet removal results, UMI histogram, and filtering statistics

## How to apply

Run the copilot() function on the raw spliced and unspliced count matrices produced by the scKB (kallisto/bustools) pipeline, leaving estimate.doublet.rate=TRUE and remove.doublet=TRUE so the doublet rate is derived from 10X Genomics' estimation rather than user guesswork; a user-specified doublet.rate can override this (default NULL). Doublet removal happens after the quality filtering steps that use mt.threshold=5 (mitochondrial percentage), filtering.ratio=1 (lenient), top.percent=1 (high-UMI cells), and min.UMI.low.quality=100 / min.UMI.high.quality=300. Removal is executed via the DoubletFinder R package dependency. Confirm results in the COPILOT summary file, which reports doublet removal outcomes alongside UMI histograms and filtering thresholds.

## Related tools

- **COPILOT** (Runs quality filtering (mt.threshold=5, filtering.ratio=1, top.percent=1, min.UMI thresholds 100/300) and performs doublet rate estimation and removal via estimate.doublet.rate and remove.doublet arguments) — https://github.com/Hsu-Che-Wei/COPILOT
- **DoubletFinder** (R package dependency used by COPILOT to detect and remove doublets after gene and cell quality filtering)
- **scKB** (Produces the raw spliced and unspliced count matrices from FASTQs via kallisto/bustools that feed COPILOT's filtering and doublet removal)

## Examples

```
copilot(sample.name="pbmc_1k_v2", mt.pattern=mt_genes, mt.threshold=5, filtering.ratio=1, top.percent=1, min.UMI.low.quality=100, min.UMI.high.quality=300, estimate.doublet.rate=TRUE, remove.doublet=TRUE, do.seurat=TRUE, do.annotation=FALSE, filtered.mtx.output.dir="pbmc_1k_v2_filtered")
```

## Evaluation signals

- COPILOT summary file explicitly reports the doublet removal results and the number of retained high-quality cells
- Estimated doublet rate matches 10X Genomics' loading-based estimation for the sequenced cell number (estimate.doublet.rate=TRUE)
- Retained cell count is sensible given the min.UMI.high.quality=300 fallback rule and lenient filtering.ratio=1
- Filtered matrices written to filtered.mtx.output.dir contain fewer barcodes than the raw input matrices, consistent with mt.threshold, top.percent, and doublet filtering

## Limitations

- Removing doublets is performed only after quality filtering of gene and cell; cells already flagged as dying (mitochondrial percentage > 5) or low-UMI are handled first
- COPILOT's annotation step only supports the root of Arabidopsis thaliana, so human datasets (e.g. pbmc_1k_v2) require do.annotation=FALSE
- The required genome may not be available as a BSgenome object, blocking upstream scKB index building for custom organisms

## Evidence

- [readme] DoubletFinder dependency: [evidence span withheld — anchor status: not_found]
