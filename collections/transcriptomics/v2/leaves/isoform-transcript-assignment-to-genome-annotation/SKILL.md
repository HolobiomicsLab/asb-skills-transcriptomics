---
name: isoform-transcript-assignment-to-genome-annotation
description: Use when raw long-read FASTQ reads from barcode-deconvolved ScISOrSeq
  libraries must be aligned, filtered for full-length spliced poly-exonic reads, and
  assigned intron/exon chains relative to GENCODE annotation.
when_to_use_negative:
- Reads have not yet undergone barcode deconvolution — run scisorseqr::GetBarcodes
  first, since unassigned reads cannot be linked to cell types
- The goal is short-read gene-level expression quantification rather than isoform/transcript-structure
  assignment from long reads
- Analysis targets a genome/annotation without matching CAGE peak and PolyA-site reference
  BED files, since TSS/PolyA site assignment (cp_distance = 50) depends on them
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scisorseqr
  role: R package providing MapAndFilter (full-length spliced read filtering against
    GENCODE GTF, CAGE and PolyA references) and InfoPerLongRead (per-read intron/exon
    chain concatenation)
  repo: https://github.com/noush-joglekar/scisorseqr
- name: STARlong
  role: Splice-aware long-read aligner used via scisorseqr::STARalign for PacBio reads
  repo: https://github.com/alexdobin/STAR
- name: Minimap2
  role: Alternative splice-aware long-read aligner used via scisorseqr::MMalign if
    not using STARlong
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: BAM processing dependency for the alignment/mapping step
- name: bedtools
  role: Genomic interval operations used when intersecting reads with annotation BED
    files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:3d80bd58df73bb20683d2d9af34cb26708ed777d8a54d2d58360429f0da56118
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# isoform-transcript-assignment-to-genome-annotation

## Summary

Assigns single-cell long reads (ScISOrSeq PacBio/ONT) to transcript structures by splice-aware alignment to the mm10 genome and annotation-guided filtering against GENCODE GTF, FANTOM CAGE and PolyA-site references, yielding full-length spliced read records with intron-chain and exon-chain assignments per read.

## When to use

Use when raw long-read FASTQ reads from barcode-deconvolved ScISOrSeq libraries must be aligned, filtered for full-length spliced poly-exonic reads, and assigned intron/exon chains relative to GENCODE annotation.

## When NOT to use

- Reads have not yet undergone barcode deconvolution — run scisorseqr::GetBarcodes first, since unassigned reads cannot be linked to cell types
- The goal is short-read gene-level expression quantification rather than isoform/transcript-structure assignment from long reads
- Analysis targets a genome/annotation without matching CAGE peak and PolyA-site reference BED files, since TSS/PolyA site assignment (cp_distance = 50) depends on them

## Inputs

- Barcode-filtered FASTQ files (OutputFiltered from scisorseqr::GetBarcodes)
- mm10 reference genome FASTAs, per chromosome (seqDir)
- GENCODE annotation GTF (.gtf.gz, annoGZ)
- FANTOM CAGE peaks BED files (mm10) for TSS assignment
- PolyA sites atlas BED files (polyasite.unibas.ch) for 3' end assignment
- Barcode-celltype assignment file from short-read Seurat analysis

## Outputs

- Aligned BAM files (STARoutput) for each filtered FASTQ
- LongReadInfo/AllInfo.gz — per-read record for complete reads with ReadName, Gene, Celltype, Barcode, UMI, IntronChain, ExonChain, Status
- LongReadInfo/AllInfo_Incomplete.gz — per-read record with relaxed 5'/3' completeness
- OutputFiltered/FilteredDeconvBC_AllFiles.csv — filtered per-read barcode assignment table

## How to apply

After scisorseqr::GetBarcodes retains only barcoded reads (OutputFiltered), align them with scisorseqr::STARalign (STARlong, for PacBio) or MMalign (Minimap2, for ONT/PacBio) into STARoutput BAMs. Then run scisorseqr::MapAndFilter with filterFullLength = TRUE, supplying the GENCODE annotation GTF (annoGZ), per-chromosome genome FASTAs (seqDir), FANTOM CAGE peak BEDs and PolyA-site BEDs, cp_distance = 50 (cutoff distance for matching TSS/PolyA sites to references), numThreads = 24, and genomeVersion = 'mm10'. This filters for full-length, spliced, poly-exonic reads and annotates each read's splice junctions. Finally, scisorseqr::InfoPerLongRead (minTimesIsoObserve = 5, rmTmpFolder = TRUE) concatenates per-read information into LongReadInfo/AllInfo.gz for complete reads and AllInfo_Incomplete.gz with relaxed 5'/3' completeness criteria, giving ReadName, Gene, Celltype, Barcode, UMI, IntronChain, ExonChain and Status columns for downstream IsoQuant quantification.

## Related tools

- **scisorseqr** (R package providing MapAndFilter (full-length spliced read filtering against GENCODE GTF, CAGE and PolyA references) and InfoPerLongRead (per-read intron/exon chain concatenation)) — https://github.com/noush-joglekar/scisorseqr
- **STARlong** (Splice-aware long-read aligner used via scisorseqr::STARalign for PacBio reads) — https://github.com/alexdobin/STAR
- **Minimap2** (Alternative splice-aware long-read aligner used via scisorseqr::MMalign if not using STARlong) — https://github.com/lh3/minimap2
- **samtools** (BAM processing dependency for the alignment/mapping step)
- **bedtools** (Genomic interval operations used when intersecting reads with annotation BED files)

## Examples

```
MapAndFilter(annoGZ='annotation.gtf.gz', numThreads=24, seqDir='genomes/chr*.fa.gz', filterFullLength=TRUE, cp_distance=50, genomeVersion='mm10')
```

## Evaluation signals

- LongReadInfo/AllInfo.gz and AllInfo_Incomplete.gz both exist after InfoPerLongRead with rmTmpFolder = TRUE
- Every row of the AllInfo table carries non-empty IntronChain and ExonChain values plus ReadName, Gene, Celltype, Barcode, UMI and Status fields
- All reads in OutputFiltered retain a valid barcode-celltype assignment from the chemistry v2 deconvolution
- Filtered reads are full-length, spliced and poly-exonic (Status reflects completeness), consistent with filterFullLength = TRUE
- Only reads whose isoform is observed at least minTimesIsoObserve = 5 times enter the AllInfo quantification input

## Limitations

- Long-read clustering may be affected by sparsity of the isoform matrix due to lower long-read throughput
- Reads failing full-length/spliced/poly-exonic criteria are only recoverable via the relaxed AllInfo_Incomplete.gz track
- TSS and PolyA site assignment accuracy is bounded by the cp_distance = 50 cutoff against FANTOM CAGE and PolyA-site references
- The scisorseqr package has only been tested on a CentOS x86_64 machine

## Evidence

- [methods] MapAndFilter step and parameters: "MapAndFilter(annoGZ='[PATH_TO_annotation.gtf.gz]',numThreads=24, 
             seqDir = '[PATH_TO_GENOME_DIR/chr*.fa.gz]',"
- [methods] Filter target described: [evidence span withheld — anchor status: not_found]
- [methods] InfoPerLongRead concatenation: "InfoPerLongRead(barcodeOutputFile = 
                  'OutputFiltered/FilteredDeconvBC_AllFiles.csv',"
- [methods] CAGE reference data: "annotated [CAGE peaks](https://fantom.gsc.riken.jp/5/datafiles/reprocessed/mm10_latest/extra/CAGE_peaks/)"
- [readme] Readme description of mapping/filtering role: [evidence span withheld — anchor status: external_doc]
