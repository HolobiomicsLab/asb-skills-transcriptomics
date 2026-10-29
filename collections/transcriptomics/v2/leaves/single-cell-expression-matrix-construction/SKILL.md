---
name: single-cell-expression-matrix-construction
description: Use when starting from CellRanger/STAR BAM files from scRNA-Seq and you
  need locus-specific TE expression counts appended to the gene expression matrix
  for downstream single-cell analysis.
when_to_use_negative:
- You only need gene expression quantification — standard CellRanger/STAR output already
  provides the gene matrix without TE annotation
- You require family-level TE metagene summarization without genomic location (that
  is what scTE produces; SoloTE exists specifically to preserve locus information)
- Analysis targets 3' end 10X data with TE-derived isoforms or TEs acting as alternative
  transcription start sites — 3' bias and short 100 nt reads prevent detection (unassessed
  EM integration is future work)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SoloTE
  role: 'Main pipeline: selects unassigned reads, overlaps with TE annotation, summarizes
    locus/family TE expression, and emits the gene+TE matrix'
  repo: github.com/bvaldebenitom/SoloTE
- name: BEDTools
  role: Assesses overlap of gene-unassigned reads with the TE BED annotation (v2.29.2
    or higher required)
  repo: https://github.com/arq5x/bedtools2
- name: samtools
  role: Required in PATH for BAM handling (v1.16 or higher)
- name: pysam
  role: Python module used by SoloTE for BAM parsing (Python ≥3.9.5)
- name: pandas
  role: Python module (≥1.5.0) used by SoloTE for matrix handling
- name: Seurat
  role: Downstream single-cell analysis tool consuming the combined gene+TE expression
    matrix
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:0db89636c76341f487f8c69b1169cce45d0b4a758af499083484ba2843b46fed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BEDTools
  - Seurat
  - SoloTE
  - pandas
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# single-cell-expression-matrix-construction

## Summary

Constructs a combined gene + transposable-element (TE) expression matrix from single-cell RNA-Seq BAM files by assigning reads not assigned to genes to TE loci (MAPQ ≥ threshold) or TE families (multi-mappers), yielding a Seurat-compatible matrix.

## When to use

Use when starting from CellRanger/STAR BAM files from scRNA-Seq and you need locus-specific TE expression counts appended to the gene expression matrix for downstream single-cell analysis.

## When NOT to use

- You only need gene expression quantification — standard CellRanger/STAR output already provides the gene matrix without TE annotation
- You require family-level TE metagene summarization without genomic location (that is what scTE produces; SoloTE exists specifically to preserve locus information)
- Analysis targets 3' end 10X data with TE-derived isoforms or TEs acting as alternative transcription start sites — 3' bias and short 100 nt reads prevent detection (unassessed EM integration is future work)

## Inputs

- BAM file from CellRanger or STAR scRNA-Seq alignment (with STAR options --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1 recommended)
- TE annotation in BED format with ID column chr|start|end|TE_Subfamily:TE_Family:TE_Class|strand, score, and placeholder field (from UCSC RepeatMasker *rm.out)

## Outputs

- Combined gene and TE expression matrix per cell, with TE features encoded as Subfamily:Family:Class locus identifiers, compatible with Seurat

## How to apply

First convert a UCSC RepeatMasker annotation (*rm.out) to BED with SoloTE_RepeatMasker_to_BED.py -g GenomeVersion (e.g., hg38 or mm10), so column 4 encodes chr|start|end|TE_Subfamily:TE_Family:TE_Class|strand. Run SoloTE_pipeline.py with --threads, --bam, --teannotation, --outputprefix, and --outputdir, ensuring Samtools ≥1.16, BEDTools ≥2.29.2, R ≥4, Python ≥3.9.5 with Pysam and Pandas ≥1.5.0 are in PATH. The pipeline selects reads not assigned to known genes (discarding gene-assigned reads unless the 'dual' option is used, to avoid falsely reporting TEs fully contained within genes as expressed), assesses overlap with the TE annotation via BEDTools, then summarizes TE expression at the locus level for alignments with MAPQ ≥ a user-defined threshold (default 255, uniquely mapped reads) and aggregates multi-mapping reads at the TE family level. Validate that the output directory is created and contains the combined gene+TE per-cell matrix under the output prefix.

## Related tools

- **SoloTE** (Main pipeline: selects unassigned reads, overlaps with TE annotation, summarizes locus/family TE expression, and emits the gene+TE matrix) — github.com/bvaldebenitom/SoloTE
- **BEDTools** (Assesses overlap of gene-unassigned reads with the TE BED annotation (v2.29.2 or higher required)) — https://github.com/arq5x/bedtools2
- **samtools** (Required in PATH for BAM handling (v1.16 or higher))
- **pysam** (Python module used by SoloTE for BAM parsing (Python ≥3.9.5))
- **pandas** (Python module (≥1.5.0) used by SoloTE for matrix handling)
- **Seurat** (Downstream single-cell analysis tool consuming the combined gene+TE expression matrix)

## Examples

```
python SoloTE_pipeline.py --threads 8 --bam sample_aligned.bam --teannotation hg38_rm.bed --outputprefix sample1 --outputdir results/sample1
```

## Evaluation signals

- Output directory specified by --outputdir is created if absent and contains the combined gene+TE expression matrix named with --outputprefix
- TE features in the matrix carry Subfamily:Family:Class locus identifiers (e.g., L1MC5a:L1:LINE) consistent with column 4 of the input BED annotation
- Reads assigned to known genes are absent from TE counts by default; verify 'dual' behavior only if that option was intentionally set
- Locus-level TE assignments restricted to MAPQ ≥ user-defined threshold (default 255 = uniquely mapped reads); multi-mapping reads appear aggregated at family level
- Matrix loads into Seurat with both gene and TE features per cell

## Limitations

- For organisms with a greater proportion of transpositionally-active TEs, most copies could be genetically intact and multi-mapping, limiting locus-level resolution
- With 3' end 10X short 100 nucleotide reads, not all TEs can be uniquely mappable
- TEs acting as alternative transcription start sites cannot be detected due to 3' bias; EM-algorithm performance on 3' end 10X data and TE-derived isoform analysis are not yet integrated

## Evidence

- [discussion] The first step of SoloTE is the selection of reads not assigned to genes.: "The first step of SoloTE is the selection of reads not assigned to genes."
- [discussion] Reads assigned to genes are discarded to avoid false TE expression inside genes: "In turn, reads assigned to known genes are discarded, to avoid mistakenly reporting TEs fully contained within genes, as expressed."
- [discussion] MAPQ threshold governs locus-level vs family-level summarization: "TE expression is summarized at the locus-level if the alignments have a MAPQ equal to or greater than a user-defined threshold"
- [discussion] Output is a combined gene+TE per-cell matrix compatible with Seurat: "a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools"
- [readme] BED ID column format required by SoloTE: "column 4, the ID, is a concatenation of the locus of the TE and its identifiers at the Subfamily, Family and Class level."
- [readme] Pipeline command line from README: "python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory"
