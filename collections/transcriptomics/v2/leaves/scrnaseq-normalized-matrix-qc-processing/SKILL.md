---
name: scrnaseq-normalized-matrix-qc-processing
description: Use when the input is a barcoded scRNA-seq dataset (e.g. GEO accession
  GSE99915) needing a QC-filtered, normalized expression matrix plus cell-ID-to-barcode-ID
  lineage annotations before lineage inference or memory-gene analysis.
when_to_use_negative:
- The cells are only distantly related — GEMLI performs less robustly in populations
  containing only distant relatives, so the lineage-annotated matrix would not support
  reliable downstream inference
- The dataset already ships a QC'd, normalized expression matrix with matching barcode-based
  lineage annotations — reprocessing from BAM would duplicate the step
- The goal is long-term lineage tracing over weeks or months — expression-memory-based
  processing only resolves lineages over about 1–5 cell divisions
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Environment for running the matrix.count.celltags.R quantification script
    and the GEMLI package downstream
  repo: https://github.com/UPSUTER/GEMLI
- name: GEMLI
  role: Consumes the QC'd, normalized expression matrix plus lineage annotations to
    predict lineages and call memory genes
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Used to install the GEMLI R package from GitHub before running downstream
    analysis
  repo: https://github.com/UPSUTER/GEMLI
- name: xcalibr
  role: Alternative tool for extracting and counting barcoded reads from short reads,
    producing count tables matched to barcode index/tag sequences
- name: samtools/grep workflow (morris-lab/BiddyetalWorkflow)
  role: Extracts CellTag motif reads from the CellRanger BAM and parses CellTag/Cell.BC/UMI
    to build the lineage annotation matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:3f976d5a283af7cf4240bb13f5530a6b84a8cea34b0155e68daf09714ea82475
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  - samtools/grep workflow (morris-lab/BiddyetalWorkflow)
  - xcalibr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# scrnaseq-normalized-matrix-qc-processing

## Summary

Load and prepare a quality-controlled, normalized single-cell RNA-seq gene expression matrix together with cell-to-lineage annotations (e.g. barcode IDs), so that lineage-aware downstream analyses such as GEMLI memory-gene calling can be run. It converts raw CellRanger/CellTag outputs into the cell-by-gene matrix and annotation tables the analysis consumes.

## When to use

Use when the input is a barcoded scRNA-seq dataset (e.g. GEO accession GSE99915) needing a QC-filtered, normalized expression matrix plus cell-ID-to-barcode-ID lineage annotations before lineage inference or memory-gene analysis.

## When NOT to use

- The cells are only distantly related — GEMLI performs less robustly in populations containing only distant relatives, so the lineage-annotated matrix would not support reliable downstream inference
- The dataset already ships a QC'd, normalized expression matrix with matching barcode-based lineage annotations — reprocessing from BAM would duplicate the step
- The goal is long-term lineage tracing over weeks or months — expression-memory-based processing only resolves lineages over about 1–5 cell divisions

## Inputs

- GEO-deposited barcoded scRNA-seq dataset (e.g. accession GSE99915), including the 10x CellRanger possorted_genome_bam.bam
- Filtered cell barcodes list from CellRanger (e.g. hf1.d15.barcodes.tsv)
- CellTag-containing reads parsed to TSV (Read.ID, Read.Seq, Cell.BC, UMI, Cell.Tag, Gene), e.g. v1.celltag.parsed.tsv
- Quality-controlled, normalized gene expression matrix (cells x genes)

## Outputs

- Cell Barcode x CellTag UMI count matrix linking cell IDs to barcode IDs (lineage annotations)
- Quality-controlled, normalized expression matrix keyed by the same cell barcodes, ready for GEMLI lineage/memory-gene analysis
- Downstream memory gene table (memory_genes_MEF.csv) with CV2, p-value, skewness, and class columns

## How to apply

Start from the publicly deposited dataset (e.g. GEO GSE99915) and take the filtered 10x CellRanger outputs — the filtered cell barcodes list (e.g. hf1.d15.barcodes.tsv) and the possorted_genome_bam.bam — keeping only cells present in the filtered CellRanger results. For lineage annotations, extract barcode-bearing reads with samtools/grep using the version-specific CellTag motifs (e.g. MEF: 'GGT[ACTG]{8}GAATTC' with the stricter capture regex 'CCGGT([ACTG]{8})GAATTC'), parse them with the workflow's custom gawk script into Read.ID/Cell.BC/UMI/Cell.Tag columns, and build a cell-barcode x cell-tag UMI count matrix with matrix.count.celltags.R so each cell ID maps to a barcode ID. For the expression matrix, use the quality-controlled, normalized gene expression matrix (counts per UMI per gene per cell) that feeds GEMLI's memory-gene workflow; downstream steps aggregate expression per barcoded lineage and compute CV2 of lineage-mean expression against a random lineage-size-matched null (memory genes called at p ≤ 0.05, split by skewness < 3 quantitative vs ≥ 3 qualitative). Correct application is judged by consistent cell identifiers between the expression matrix and the barcode annotation table and by downstream counts falling in the reported validation ranges (total 3668 ± 1131; quantitative 2286 ± 890; qualitative 1382 ± 413).

## Related tools

- **R** (Environment for running the matrix.count.celltags.R quantification script and the GEMLI package downstream) — https://github.com/UPSUTER/GEMLI
- **GEMLI** (Consumes the QC'd, normalized expression matrix plus lineage annotations to predict lineages and call memory genes) — https://github.com/UPSUTER/GEMLI
- **devtools** (Used to install the GEMLI R package from GitHub before running downstream analysis) — https://github.com/UPSUTER/GEMLI
- **xcalibr** (Alternative tool for extracting and counting barcoded reads from short reads, producing count tables matched to barcode index/tag sequences)
- **samtools/grep workflow (morris-lab/BiddyetalWorkflow)** (Extracts CellTag motif reads from the CellRanger BAM and parses CellTag/Cell.BC/UMI to build the lineage annotation matrix)

## Examples

```
Rscript ./scripts/matrix.count.celltags.R ./cell.barcodes/hf1.d15.barcodes.tsv v1.celltag.parsed.tsv hf1.d15.v1
```

## Evaluation signals

- Every cell barcode in the normalized expression matrix maps to a barcode ID in the lineage annotation table (consistent cell identifiers between the two tables)
- Only cell barcodes present in the filtered 10x CellRanger results are retained in the CellTag count matrix (explicit filter in matrix.count.celltags.R)
- Downstream memory-gene counts fall within reported validation ranges: total 3668 ± 1131, quantitative 2286 ± 890, qualitative 1382 ± 413
- Called memory genes have lineage-mean CV2 significantly higher than the random lineage-size-matched null at p ≤ 0.05
- Memory gene classes respect the skewness cutoff: quantitative (skewness < 3), qualitative (skewness ≥ 3)

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease GEMLI's performance (cannot be excluded)
- Expression-memory lineage analysis is limited to lineages over about 1–5 cell divisions; long-term tracing needs other approaches

## Evidence

- [abstract] abstract: "GEMLI is available as an R package on GitHub"
- [intro] intro: "While cellular characterization by scRNA-seq provides rich molecular information, it is intrinsically devoid of lineage information."
- [readme] readme: [evidence span withheld — anchor status: not_found]
- [readme] readme2: [evidence span withheld — anchor status: not_found]
- [readme] readme3: [evidence span withheld — anchor status: not_found]
- [other] task_workflow: [evidence span withheld — anchor status: not_found]
- [other] task_workflow2: [evidence span withheld — anchor status: not_found]
