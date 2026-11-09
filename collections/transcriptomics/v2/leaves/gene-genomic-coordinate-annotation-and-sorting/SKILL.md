---
name: gene-genomic-coordinate-annotation-and-sorting
description: Use when a filtered gene-by-cell count matrix (e.g. SCEVAN input after
  removing cells with <200 genes and genes in <1% of cells) lacks genomic coordinates
  or is not yet ordered by genomic position prior to CNV segmentation.
when_to_use_negative:
- The count matrix already carries seqnames/start/end columns and is sorted by genomic
  coordinate — re-annotation is redundant
- The analysis goal does not depend on genomic order (e.g. cell-type clustering or
  pathway enrichment) — coordinate sorting is only needed for copy number smoothing/segmentation
  along the genome
- Gene identifiers cannot be resolved by the available Ensembl annotation (e.g. non-standard
  IDs), leaving genes without coordinates
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: SCEVAN
  role: R package whose workflow annotates the remaining genes with genomic locations
    via an Ensembl-based annotation package and sorts them by genomic coordinates
    before smoothing and segmentation
  repo: github.com/AntonioDeFalco/SCEVAN
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:cfe3136c4667a60615adf5acf5f0de5305dfa0157baefe4125ef9bdf8ca7a11e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCEVAN
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# gene-genomic-coordinate-annotation-and-sorting

## Summary

Annotates the filtered gene-by-cell scRNA-seq count matrix with genomic locations (seqnames, start, end) using an Ensembl-based annotation package and reorders genes by genomic coordinate, so that downstream copy number smoothing and segmentation can detect breakpoints along the genome.

## When to use

Use when a filtered gene-by-cell count matrix (e.g. SCEVAN input after removing cells with <200 genes and genes in <1% of cells) lacks genomic coordinates or is not yet ordered by genomic position prior to CNV segmentation.

## When NOT to use

- The count matrix already carries seqnames/start/end columns and is sorted by genomic coordinate — re-annotation is redundant
- The analysis goal does not depend on genomic order (e.g. cell-type clustering or pathway enrichment) — coordinate sorting is only needed for copy number smoothing/segmentation along the genome
- Gene identifiers cannot be resolved by the available Ensembl annotation (e.g. non-standard IDs), leaving genes without coordinates

## Inputs

- Filtered gene-by-cell raw count matrix (genes as rows, Gene Symbol or Ensembl ID; e.g. from GSE131928 MGH106 sample)
- Ensembl-based genomic annotation package providing seqnames, start, end for each gene

## Outputs

- Annotated gene matrix with genomic location columns (seqnames, start, end), sorted by genomic coordinate (e.g. *_count_mtx_annot.RData)

## How to apply

After quality filtering and cell-cycle gene removal (REACTOME cell cycle pathway), attach each gene's genomic location to the matrix with an Ensembl-based annotation package (e.g. the package cited as ref. 35 in the SCEVAN workflow), then sort rows strictly by genomic coordinates. This ordering is essential because SCEVAN's subsequent edge-preserving total-variation smoothing (phi(x)=logcosh(x), beta_vega parameter) and greedy Mumford–Shah piecewise-constant segmentation assume breakpoints are detected along the genomic axis. The resulting annotated matrix (e.g. MGH106_count_mtx_annot.RData with seqnames/start/end) is what CNA-profile extraction uses when pulling regions such as chr3:158644278-194498364 via count_mtx_annot$seqnames, $start, $end filters.

## Related tools

- **SCEVAN** (R package whose workflow annotates the remaining genes with genomic locations via an Ensembl-based annotation package and sorts them by genomic coordinates before smoothing and segmentation) — github.com/AntonioDeFalco/SCEVAN

## Examples

```
load("output/MGH106_count_mtx_annot.RData"); head(count_mtx_annot[, c("seqnames", "start", "end")])
```

## Evaluation signals

- Every gene in the matrix has non-missing seqnames, start, and end after annotation
- Row order of the annotated matrix is monotonically non-decreasing by (seqnames/chromosome, start) coordinate
- The number of annotated genes equals the number of genes retained after the <200-gene/cell and <1%-expressed filters plus cell-cycle gene removal
- Downstream region queries succeed, e.g. mean CNA over count_mtx_annot$seqnames==3 & start>=158644278 & end<=194498364 yields a per-cell vector for plotting
- The sorted annotated matrix feeds correct piecewise-constant segmentation: breakpoints align with genomic coordinate order, not alphabetical gene order

## Limitations

- The specific Ensembl-based annotation package (ref. 35) and its version are not named in the text, so reproduction depends on inferring the package from the SCEVAN repository
- Genes without resolvable genomic coordinates must be dropped or handled explicitly before sorting, otherwise breakpoint detection along the genome is corrupted

## Evidence

- [discussion] Annotation and sorting step of the SCEVAN workflow: "The remaining genes are annotated by adding their genomic locations to the matrix using an Ensembl-based annotation package35 and then genes are sorted according to genomic coordinates."
- [readme] Annotated matrix used for region-level CNA extraction: [evidence span withheld — anchor status: external_doc]
- [methods] Entry point that triggers annotation and sorting internally: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
