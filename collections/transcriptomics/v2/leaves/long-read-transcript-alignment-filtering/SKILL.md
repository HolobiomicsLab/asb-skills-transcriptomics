---
name: long-read-transcript-alignment-filtering
description: Use when raw PacBio/ONT FASTQ reads with single-cell barcodes must be
  splice-aligned to a reference genome and filtered to high-confidence full-length
  spliced isoforms before per-cell-type isoform quantification.
when_to_use_negative:
- Input reads are short Illumina paired-end reads — minimap2/STARlong presets and
  MapAndFilter full-length criteria target long noisy reads
- Goal is genome-free transcript quantification directly from FASTQ without a reference
  genome and annotation
- Reads lack single-cell barcodes and only bulk transcriptome alignment (no cell-type
  deconvolution) is needed
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: scisorseqr
  role: R package orchestrating barcode deconvolution, STARalign/MMalign wrappers
    and MapAndFilter full-length read filtering
  repo: https://github.com/noush-joglekar/scisorseqr
- name: STARlong
  role: Splice-aware aligner used by STARalign for PacBio long reads against a GENCODE-indexed
    mm10 genome
  repo: https://github.com/alexdobin/STAR
- name: Minimap2
  role: Alternative splice-aware long-read aligner used via MMalign (e.g. for ONT
    reads), using spliced presets
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Supports BAM handling during alignment and filtering
- name: bedtools
  role: Interval overlap operations underlying the cp_distance=50 CAGE/polyA end assignments
    in MapAndFilter
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:b51a47ff81d1e75da5d8cf5acd528f8f0a9a4b88117433550a14cfa3167b34a2
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
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# long-read-transcript-alignment-filtering

## Summary

Align barcoded long-read (PacBio/ONT) isoform reads to the mm10 genome with STARlong or Minimap2 and retain only mapped, full-length, spliced, poly-exonic reads via scisorseqr MapAndFilter, using GENCODE annotation, FANTOM CAGE and polyASite 2.0 interval files (cp_distance=50).

## When to use

Use when raw PacBio/ONT FASTQ reads with single-cell barcodes must be splice-aligned to a reference genome and filtered to high-confidence full-length spliced isoforms before per-cell-type isoform quantification.

## When NOT to use

- Input reads are short Illumina paired-end reads — minimap2/STARlong presets and MapAndFilter full-length criteria target long noisy reads
- Goal is genome-free transcript quantification directly from FASTQ without a reference genome and annotation
- Reads lack single-cell barcodes and only bulk transcriptome alignment (no cell-type deconvolution) is needed

## Inputs

- Barcoded PacBio or ONT FASTQ files (fastq.gz) from ScISOrSeq libraries
- Barcode-celltype assignment file from Seurat short-read analysis
- GENCODE annotation GTF (gzipped)
- Per-chromosome mm10 reference genome FASTAs (seqDir, chr*.fa.gz)
- FANTOM CAGE peak BED for mm10
- polyASite 2.0 polyA-site BED
- STARlong genome index (e.g. ~/starIndex_gencode10/) or minimap2 binary

## Outputs

- Aligned long-read BAM files (STARoutput or MMoutput)
- OutputFiltered/FilteredDeconvBC_AllFiles.csv with per-read barcode, UMI, cell type, intron/exon chain and full-length status
- Filtered set of mapped, spliced, full-length, poly-exonic reads ready for InfoPerLongRead

## How to apply

First deconvolve barcodes from the fastq.gz files with scisorseqr::GetBarcodes (chemistry='v2', concatenate=TRUE, filterReads=TRUE) to produce the OutputFiltered barcode table. Then align with scisorseqr::STARalign pointing at the STARlong binary and a GENCODE (gencode10) genome index for PacBio reads, or scisorseqr::MMalign with the minimap2 binary for ONT reads, writing BAMs to STARoutput/MMoutput. Next run scisorseqr::MapAndFilter with annoGZ (GENCODE GTF.gz), numThreads=24, seqDir with per-chromosome mm10 FASTAs, the FANTOM CAGE peak BED and polyASite 2.0 BED, cp_distance=50 and filterFullLength=TRUE, so that 5' starts lie within 50 bp of an annotated CAGE peak and 3' ends within 50 bp of an annotated polyA site. The rationale is that only reads with confirmed, externally supported ends and splice junctions constitute full-length isoforms suitable for downstream InfoPerLongRead and IsoQuant quantification.

## Related tools

- **scisorseqr** (R package orchestrating barcode deconvolution, STARalign/MMalign wrappers and MapAndFilter full-length read filtering) — https://github.com/noush-joglekar/scisorseqr
- **STARlong** (Splice-aware aligner used by STARalign for PacBio long reads against a GENCODE-indexed mm10 genome) — https://github.com/alexdobin/STAR
- **Minimap2** (Alternative splice-aware long-read aligner used via MMalign (e.g. for ONT reads), using spliced presets) — https://github.com/lh3/minimap2
- **samtools** (Supports BAM handling during alignment and filtering)
- **bedtools** (Interval overlap operations underlying the cp_distance=50 CAGE/polyA end assignments in MapAndFilter)

## Examples

```
scisorseqr::MapAndFilter(annoGZ='annotation.gtf.gz', numThreads=24, seqDir='~/genomes/mm10_dir/chr*.fa.gz', cageBed='CAGE_peaks.bed', polyABed='polyASite.bed', cp_distance=50, filterFullLength=TRUE, genomeVersion='mm10')
```

## Evaluation signals

- OutputFiltered CSV exists and contains ReadName, Gene, Celltype, Barcode, UMI, IntronChain, ExonChain and Status columns with non-empty barcode assignments
- Filtered reads are spliced and poly-exonic: every retained read has at least a junction chain (intron chain) in its Status/IntronChain fields
- 5' and 3' end assignments fall within cp_distance=50 bp of FANTOM CAGE peaks and polyASite 2.0 sites respectively
- Downstream InfoPerLongRead produces LongReadInfo/AllInfo.gz (complete) and AllInfo_Incomplete.gz (relaxed) without errors

## Limitations

- Long-read throughput is lower than short-read, so the resulting isoform matrix is sparse, which can cause clustering discrepancies
- The scisorseqr package has only been tested on a CentOS x86_64 machine
- Reads lacking confirmed 5' and 3' end support are excluded from the high-confidence set; only relaxed-completeness information is retained in AllInfo_Incomplete.gz

## Evidence

- [methods] Alignment filters: "## Step 3: Map and filter for full-length, spliced reads"
- [methods] STARalign call: "STARalign(fqFolder = fastqFolder, starProgPath = '~/bin/Linux_x86_64/STARlong'"
- [methods] MapAndFilter parameters: "MapAndFilter(annoGZ='[PATH_TO_annotation.gtf.gz]',numThreads=24, 
             seqDir = '[PATH_TO_GENOME_DIR/chr*.fa.gz]',"
- [readme] Hardware requirements from README: "The package has only been tested on a CentOS x86\_64 machine. For
alignment and mapping, we recommend"
- [readme] Package scope from README: "The package includes functions for barcode deconvolution
from fastqs, integration with long read alignment tools, mapping and
filtering of high confidence, full-length spliced reads, and some"
