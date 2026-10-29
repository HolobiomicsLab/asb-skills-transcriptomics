---
name: isoform-count-table-processing
description: Use when per-cell-type long-read isoform count tables (IsoQuantOutput-style
  'Modality-ModalityID.csv', NumModalityPerCluster files, or an AllInfo file) must
  be converted into pairwise differential isoform expression results with dPI and
  FDR statistics.
when_to_use_negative:
- Input is raw PacBio/ONT fastq — barcode deconvolution, STARlong/Minimap2 alignment,
  and MapAndFilter must run first before count tables exist
- The goal is exon-level or gene-level differential expression rather than isoform/TSS
  usage differences (ExonQuant or a gene DE tool would be the wrong-fit substitute)
- Data are short-read only — isoform-resolved count tables require full-length spliced
  long reads; sparsity from low long-read throughput can invalidate comparisons
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scisorseqr
  role: R package supplying IsoQuant, ExonQuant, and DiffSplicingAnalysis functions
    that turn the AllInfo/count tables into DIE test results
  repo: github.com/noush-joglekar/scisorseqr
- name: IsoQuant
  role: Sister long-read annotation package; its output-style 'Modality-ModalityID.csv'
    count tables are the deposited per-cell-type input format
  repo: github.com/ablab/isoquant
- name: R
  role: Runtime environment for DiffSplicingAnalysis and downstream delta-PI classification
    of DIE genes
- name: pandas
  role: Python library for loading the results tables, computing pairwise DIE fractions,
    and writing CSV summaries
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:252693a85ef5a3b2d44697168f4728adb246c14e6684d1619514925578c469ad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DiffSplicingAnalysis
  - IsoQuant
  - Python
  - R
  - pandas
  - scisorseqr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# isoform-count-table-processing

## Summary

Processes long-read ScISOrSeq isoform count tables (per-cell-type isoform/TSS usage matrices) into differential isoform expression (DIE) test results using the scisorseqr R package, the paper's core method for quantifying region- and cell-type-specific isoform regulation in the P7 mouse HIPP vs PFC isoform atlas.

## When to use

Use when per-cell-type long-read isoform count tables (IsoQuantOutput-style 'Modality-ModalityID.csv', NumModalityPerCluster files, or an AllInfo file) must be converted into pairwise differential isoform expression results with dPI and FDR statistics.

## When NOT to use

- Input is raw PacBio/ONT fastq — barcode deconvolution, STARlong/Minimap2 alignment, and MapAndFilter must run first before count tables exist
- The goal is exon-level or gene-level differential expression rather than isoform/TSS usage differences (ExonQuant or a gene DE tool would be the wrong-fit substitute)
- Data are short-read only — isoform-resolved count tables require full-length spliced long reads; sparsity from low long-read throughput can invalidate comparisons

## Inputs

- LongReadInfo/AllInfo concatenated per-read information file from InfoPerLongRead
- Per-cell-type isoform count tables (IsoQuantOutput-style 'Modality-ModalityID.csv')
- NumModalityPerCluster files from the isoformAtlas deposit (www.isoformAtlas.com)
- Tab-separated config file in Comparison1/Celltypes/Comparison2/Celltypes format
- GENCODE/mm10 reference (gencode10 STAR index) used upstream for alignment

## Outputs

- Per-comparison DIE results tables with columns Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR
- Pairwise DIE-fraction tables per cell-type comparison at the deltaPI=0.1 threshold
- Per-gene model-classification tables (e.g. die_model_classification.csv, cpe_tss_direction_summary.csv)

## How to apply

Generate the per-modality count tables by running scisorseqr's IsoQuant on the concatenated LongReadInfo/AllInfo file with Iso=TRUE, TSS=TRUE, PolyA=TRUE. Build a tab-separated config file in the Comparison1/Celltypes/Comparison2/Celltypes format listing the cell types to compare (e.g. HIPP vs PFC, or CPE vs non-CPE subtypes). Run DiffSplicingAnalysis with typeOfTest='Iso' (or 'TSS' for transcription-start-site usage), minNumReads=25, and is.hier=FALSE to obtain per-comparison results tables with columns Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR. Call significant DIE at deltaPI=0.1 and, for cell-type attribution, classify each gene by the delta-PI(subtype)/delta-PI(bulk) ratio into Single-/Both-/No-Cell-Type models. Isoform-level tests are used rather than exon-level (ExonQuant) tests because the paper found isoform tests for DIE show better performance.

## Related tools

- **scisorseqr** (R package supplying IsoQuant, ExonQuant, and DiffSplicingAnalysis functions that turn the AllInfo/count tables into DIE test results) — github.com/noush-joglekar/scisorseqr
- **IsoQuant** (Sister long-read annotation package; its output-style 'Modality-ModalityID.csv' count tables are the deposited per-cell-type input format) — github.com/ablab/isoquant
- **R** (Runtime environment for DiffSplicingAnalysis and downstream delta-PI classification of DIE genes)
- **pandas** (Python library for loading the results tables, computing pairwise DIE fractions, and writing CSV summaries)

## Examples

```
DiffSplicingAnalysis(configFile = config, typeOfTest = 'Iso', minNumReads = 25, is.hier = FALSE)
```

## Evaluation signals

- Results tables contain the required columns Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR, one row per tested isoform/gene
- minNumReads=25 filter was applied and is.hier=FALSE was used, matching the published parameterization
- Significant DIE calls reproduce the paper's scale, e.g. 395 genes with DIE between hippocampus and PFC, and 151/81/26 genes in the Single-/Both-/No-Cell-Type-Model classes (75%/24.4%/0.3%)
- Downstream validation files exist and match reported values, e.g. cpe_tss_direction_summary.csv reporting 70/93 upstream-TSS with Bernoulli p <= 3e-7
- Classification fractions fall within reported ranges (Single-Cell-Type 75% +/- 2.3 SE) with class counts matching the input DIE result count

## Limitations

- Sparsity of the isoform matrix due to lower long-read throughput can cause clustering and testing discrepancies
- Genes below minNumReads=25 reads are excluded, so lowly expressed isoform shifts go untested
- Traditional transcriptional cell-type definitions may not correspond to isoform expression programs; DIE patterns in adult regions, disease, and other species are untested

## Evidence

- [methods] DiffSplicingAnalysis call parameters: "DiffSplicingAnalysis(configFile = config, 
                     typeOfTest = 'Iso', 
                     minNumReads = 25, is.hier = FALSE)"
- [methods] IsoQuant count-table generation parameters: "IsoQuant(AllInfoFile = 'LongReadInfo/AllInfo',
          Iso = TRUE, TSS = TRUE, PolyA = TRUE)"
- [readme] scisorseqr package capability statement: [evidence span withheld — anchor status: not_found]
- [abstract] Rationale for isoform-level testing: "Isoform tests for DIE show better performance than exon tests."
- [discussion] Published HIPP vs PFC DIE yield: "Region-specific isoform testing revealed 395 genes with DIE between hippocampus and PFC, partially caused by isoforms novel to GENCODE."
