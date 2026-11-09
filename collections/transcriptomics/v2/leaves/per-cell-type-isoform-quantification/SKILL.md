---
name: per-cell-type-isoform-quantification
description: Use when you have PacBio/ONT single-cell long-read fastq.gz files plus
  a barcode-to-cell-type assignment table (e.g. from Seurat short-read analysis) and
  need per-cell-type isoform, TSS, or polyA counts.
when_to_use_negative:
- Input reads lack single-cell barcodes or a barcode-to-celltype assignment file —
  the quantification depends on assigning every long read to a cell type via deconvolved
  barcodes
- You only need gene-level expression counts, not isoform/TSS/polyA resolution — short-read
  quantification pipelines are more appropriate
- Differential exon-level splicing tests are the sole goal and isoform-level testing
  is unnecessary (note the paper found isoform tests perform better for DIE, but ExonQuant
  exists for that narrower need)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0203
tools:
- name: scisorseqr
  role: R-package orchestrating barcode deconvolution (GetBarcodes), read filtering
    (MapAndFilter), per-read info concatenation (InfoPerLongRead), and isoform quantification
    (IsoQuant)
  repo: https://github.com/noush-joglekar/scisorseqr
- name: STARlong
  role: splice-aware alignment of PacBio long reads to the mm10 reference genome via
    STARalign
  repo: https://github.com/alexdobin/STAR
- name: Minimap2
  role: alternative splice-aware long-read aligner via MMalign when not using STARlong
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: downstream BAM/alignment handling required by the package
- name: bedtools
  role: genomic interval operations supporting the mapping/filtering step
- name: pandas
  role: Python library for assembling the per-cell-type isoform count matrix from
    IsoQuantOutput files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:7a8b1c3643c81c110928de540bd81c81552fa49295c158160abb07a51b23f933
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noush-joglekar/scisorseqr
  tools:
  - Minimap2
  - Python
  - R
  - STARlong
  - bedtools
  - pandas
  - samtools
  - scisorseqr
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# per-cell-type-isoform-quantification

## Summary

Quantify isoform, TSS, and polyA-site usage per cell type from ScISOrSeq long reads by chaining scisorseqr barcode deconvolution, STARlong/Minimap2 alignment, full-length spliced-read filtering, and IsoQuant, producing cell-type-resolved isoform count matrices.

## When to use

Use when you have PacBio/ONT single-cell long-read fastq.gz files plus a barcode-to-cell-type assignment table (e.g. from Seurat short-read analysis) and need per-cell-type isoform, TSS, or polyA counts.

## When NOT to use

- Input reads lack single-cell barcodes or a barcode-to-celltype assignment file — the quantification depends on assigning every long read to a cell type via deconvolved barcodes
- You only need gene-level expression counts, not isoform/TSS/polyA resolution — short-read quantification pipelines are more appropriate
- Differential exon-level splicing tests are the sole goal and isoform-level testing is unnecessary (note the paper found isoform tests perform better for DIE, but ExonQuant exists for that narrower need)

## Inputs

- Packaged PacBio or ONT fastq.gz long-read files
- Barcode-to-celltype assignment file from short-read Seurat clustering
- GENCODE annotation GTF (.gtf.gz)
- Per-chromosome mm10 genome FASTAs (chr*.fa.gz)
- FANTOM CAGE peaks BED (mm10)
- PolyA sites atlas BED
- STARlong genome index (e.g. ~/starIndex_gencode10/)

## Outputs

- OutputFiltered/FilteredDeconvBC_AllFiles.csv (barcoded, filtered reads)
- STARoutput BAM alignment files
- LongReadInfo/AllInfo.gz and AllInfo_Incomplete.gz per-read tables (ReadName, Gene, Celltype, Barcode, UMI, IntronChain, ExonChain, Status)
- IsoQuantOutput per-cell-type isoform, TSS, and polyA-site counts
- Per-cell-type isoform count matrix CSV (cell_type, isoform/gene id, counts)

## How to apply

Deconvolve cell barcodes with scisorseqr::GetBarcodes using bc_celltype_assignments, chemistry='v2', concatenate=TRUE and filterReads=TRUE so only barcoded reads land in OutputFiltered. Align filtered long reads to the mm10 reference genome with scisorseqr::STARalign (PacBio) or MMalign (ONT/PacBio), then run MapAndFilter with the GENCODE annotation GTF, per-chromosome FASTAs, FANTOM CAGE peaks, and PolyA-site BED files (cp_distance=50, numThreads=24, filterFullLength=TRUE) to keep full-length, spliced, poly-exonic reads. Concatenate per-read information with InfoPerLongRead (minTimesIsoObserve=5, rmTmpFolder=TRUE) to build LongReadInfo/AllInfo.gz, then quantify isoform, TSS, and polyA usage per cell type via IsoQuant(AllInfoFile='LongReadInfo/AllInfo', Iso=TRUE, TSS=TRUE, PolyA=TRUE). Finally assemble a per-cell-type isoform count matrix (cell_type, isoform/gene id, counts) from the IsoQuantOutput files, e.g. with a MakeMatrices script, for downstream differential isoform expression testing (DiffSplicingAnalysis with typeOfTest='Iso', minNumReads=25).

## Related tools

- **scisorseqr** (R-package orchestrating barcode deconvolution (GetBarcodes), read filtering (MapAndFilter), per-read info concatenation (InfoPerLongRead), and isoform quantification (IsoQuant)) — https://github.com/noush-joglekar/scisorseqr
- **STARlong** (splice-aware alignment of PacBio long reads to the mm10 reference genome via STARalign) — https://github.com/alexdobin/STAR
- **Minimap2** (alternative splice-aware long-read aligner via MMalign when not using STARlong) — https://github.com/lh3/minimap2
- **samtools** (downstream BAM/alignment handling required by the package)
- **bedtools** (genomic interval operations supporting the mapping/filtering step)
- **pandas** (Python library for assembling the per-cell-type isoform count matrix from IsoQuantOutput files)

## Examples

```
scisorseqr::IsoQuant(AllInfoFile = 'LongReadInfo/AllInfo', Iso = TRUE, TSS = TRUE, PolyA = TRUE)
```

## Evaluation signals

- The pipeline script runs end-to-end without errors through GetBarcodes, STARalign/MMalign, MapAndFilter, InfoPerLongRead, and IsoQuant
- Both output files exist: the per-cell-type isoform count matrix CSV and the filtered per-read assignment table (OutputFiltered CSV / LongReadInfo AllInfo table)
- The count matrix contains cell_type, isoform/gene id, and counts fields with the expected schema
- All counts in the matrix are non-negative integers
- InfoPerLongRead reports isoforms observed at least minTimesIsoObserve=5 times in LongReadInfo/AllInfo.gz, with a relaxed AllInfo_Incomplete.gz for 5'/3' incompleteness

## Limitations

- The isoform matrix is sparse due to lower long-read throughput, which can cause reproducibility issues in long-read clustering
- Only full-length, spliced, poly-exonic reads are retained in AllInfo; incomplete reads are separated into AllInfo_Incomplete.gz and not used for strict quantification
- Traditional transcriptional cell-type definitions may not correspond to isoform expression programs and may need re-definition
- Whether observed cell-type-specific DIE patterns persist in disease case-control studies, across adult brain regions, and across species is untested

## Evidence

- [methods] GetBarcodes call: "GetBarcodes(fqFolder = fastqFolder, BCClustAssignFile = bc_clust,
            chemistry = "v2", concatenate = TRUE, filterReads = FALSE)"
- [methods] IsoQuant call: "IsoQuant(AllInfoFile = 'LongReadInfo/AllInfo',
          Iso = TRUE, TSS = TRUE, PolyA = TRUE)"
- [readme] GetBarcodes description: "The package includes functions for barcode deconvolution
from fastqs, integration with long read alignment tools, mapping and
filtering of high confidence, full-length spliced reads"
- [abstract] DIE detection: "We detect hundreds of DIE events traceable to cell types, often corresponding to functionally distinct protein isoforms."
- [discussion] Sparsity limitation: "which could be partially due to the sparsity of the isoform matrix due to lower long-read throughput"
