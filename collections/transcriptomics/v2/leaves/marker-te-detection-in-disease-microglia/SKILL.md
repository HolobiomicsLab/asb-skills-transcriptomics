---
name: marker-te-detection-in-disease-microglia
description: Use when scRNA-Seq from a disease-vs-control mouse model needs locus-level
  TE activity assessed in annotated cell clusters (e.g. DAM/microglia), beyond gene-level
  markers.
when_to_use_negative:
- If the goal is whole-transcriptome gene-level marker discovery rather than TE locus-specific
  activity — standard gene expression pipelines suffice.
- If the question is family-level TE metagene summary rather than genomic locus of
  expression — scTE-family approaches suffice (though they lose locus information
  and missed the 71 marker TEs here).
- If reads are already assigned as a gene-TE feature matrix (e.g. prior CellRanger/SoloTE
  output) — skip to Seurat/edgeR stages.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SoloTE
  role: 'Core pipeline: selects unassigned reads, overlaps with TE BED, summarizes
    locus-level (MAPQ>=255) and family-level TE expression, outputs Seurat-compatible
    matrix'
  repo: https://github.com/bvaldebenitom/SoloTE
- name: STAR
  role: Aligner for scRNA-Seq reads with multi-mapping options (--winAnchorMultimapNmax
    100, --outFilterMultimapNmax 100, --outSAMmultNmax 1)
- name: BEDTools
  role: Assesses overlap of unassigned reads with TE annotation; required v2.29.2
    or higher
  repo: https://github.com/arq5x/bedtools2
- name: Samtools
  role: Dependency for BAM handling within the SoloTE pipeline
- name: Seurat
  role: Normalization, clustering, UMAP, DAM annotation (Cst7, Lpl, Clec7a), and FindAllMarkers
    for marker TEs
- name: scTE
  role: 'Contrast tool: family-level metagene TE summary used as comparison (detected
    0 of the marker TEs)'
- name: edgeR
  role: Pseudobulk differential expression between disease and control to identify
    disease-upregulated TEs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:0539c9c292c0d86e43af9893e1cf51c9348554d27f67e3d769ad9e2472665a14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - STAR
  - Samtools
  - Seurat
  - SoloTE
  - edgeR
  - scTE
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Marker TE detection in disease microglia

## Summary

Detect locus-specific transposable elements that mark disease-associated microglia in scRNA-Seq data by running the SoloTE pipeline to build a combined gene+TE expression matrix, then using Seurat FindAllMarkers and edgeR pseudobulk differential expression between disease and control samples.

## When to use

Use when scRNA-Seq from a disease-vs-control mouse model needs locus-level TE activity assessed in annotated cell clusters (e.g. DAM/microglia), beyond gene-level markers.

## When NOT to use

- If the goal is whole-transcriptome gene-level marker discovery rather than TE locus-specific activity — standard gene expression pipelines suffice.
- If the question is family-level TE metagene summary rather than genomic locus of expression — scTE-family approaches suffice (though they lose locus information and missed the 71 marker TEs here).
- If reads are already assigned as a gene-TE feature matrix (e.g. prior CellRanger/SoloTE output) — skip to Seurat/edgeR stages.

## Inputs

- scRNA-Seq FASTQ data from disease and control samples (e.g. APP/PS1 AD mouse model from the AD Knowledge Portal)
- RepeatMasker rm.out file for the reference genome (mm10 from UCSC)
- STAR-aligned BAM files
- SoloTE-format TE annotation BED (chr, start, end, locus|Subfamily:Family:Class ID, strand, score)

## Outputs

- Combined gene + TE expression matrix compatible with Seurat
- Cluster-level marker TE table from FindAllMarkers (adjusted P-value <= 0.05, avg log2FC >= 1)
- Pseudobulk disease-vs-control TE differential expression table from edgeR (e.g. 9 disease-upregulated TEs)

## How to apply

Convert the RepeatMasker rm.out file (mm10) to SoloTE BED format (chr, start, end, ID concatenating locus and Subfamily:Family:Class, strand, score) with SoloTE_RepeatMasker_to_BED.py -g mm10. Align reads with STAR using the recommended multi-mapping options --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, and --outSAMmultNmax 1, then run SoloTE_pipeline.py, which selects reads not assigned to genes, assesses TE overlap with BEDTools, and summarizes TE expression at locus level for reads with MAPQ >= 255 and family level for multi-mapping reads. Import the combined gene+TE matrix into Seurat, normalize, cluster, UMAP, and annotate disease-associated microglia using Cst7, Lpl, and Clec7a. Run FindAllMarkers keeping adjusted P-value <= 0.05 and average log2(Fold Change) >= 1 to identify marker TEs (e.g. the 71 DAM marker TEs in the APP/PS1 model). Finally, perform edgeR pseudobulk differential expression disease-vs-wild-type to find disease-upregulated TEs (e.g. Arthur1B, RLTR14-int, MMVL30-int); contrast against scTE family-level metagenes which detect none.

## Related tools

- **SoloTE** (Core pipeline: selects unassigned reads, overlaps with TE BED, summarizes locus-level (MAPQ>=255) and family-level TE expression, outputs Seurat-compatible matrix) — https://github.com/bvaldebenitom/SoloTE
- **STAR** (Aligner for scRNA-Seq reads with multi-mapping options (--winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1))
- **BEDTools** (Assesses overlap of unassigned reads with TE annotation; required v2.29.2 or higher) — https://github.com/arq5x/bedtools2
- **Samtools** (Dependency for BAM handling within the SoloTE pipeline)
- **Seurat** (Normalization, clustering, UMAP, DAM annotation (Cst7, Lpl, Clec7a), and FindAllMarkers for marker TEs)
- **scTE** (Contrast tool: family-level metagene TE summary used as comparison (detected 0 of the marker TEs))
- **edgeR** (Pseudobulk differential expression between disease and control to identify disease-upregulated TEs)

## Examples

```
python SoloTE_pipeline.py --threads 8 --bam sample.bam --teannotation mm10_TE.bed --outputprefix APP_PS1 --outputdir out_soloTE
```

## Evaluation signals

- Output table reports 71 marker TEs at adjusted P-value <= 0.05 and average log2(FC) >= 1 for the DAM clusters.
- Pseudobulk edgeR analysis yields 9 disease-upregulated TEs including Arthur1B, RLTR14-int, and MMVL30-int.
- Locus-level assignments restricted to reads with MAPQ >= 255 (uniquely mapped); family-level counts only for multi-mapping reads.
- DAM/microglia clusters annotated by marker genes Cst7, Lpl, and Clec7a before TE marker testing.
- Marker TEs detected by SoloTE are absent from scTE family-level results (scTE detects 0).

## Limitations

- Multi-mapping reads cannot be assigned to specific loci; only uniquely mapped (MAPQ >= 255) reads give locus-level expression.
- In organisms with a greater proportion of transpositionally-active TEs, most copies could be genetically intact and multi-mapping, limiting locus-level resolution.
- Not all TEs can be uniquely mappable with short 100 nucleotide reads; TEs acting as alternative transcription start sites cannot be detected due to 3' bias of 10X data.
- No experimental validation performed; gain/loss-of-function follow-up needed to confirm TE influence on gene expression.
- Accession IDs for analyzed datasets are not provided in the text (only a pointer to Supplementary Table 3).

## Evidence

- [discussion] Marker testing thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] Locus-level MAPQ rule: "TE expression is summarized at the locus-level if the alignments have a MAPQ equal to or greater than a user-defined threshold"
- [discussion] Application context: "we applied SoloTE to profile the TE transcriptome of the murine embryonic two-cell stage, during early GC, and in the APP/PS1 AD mouse model"
- [discussion] Multi-mapping alignment options: [evidence span withheld — anchor status: not_found]
- [readme] SoloTE dependencies: "SoloTE requires the following tools to be installed, and available in your PATH environment variable:
- Samtools v1.16 or higher
- BEDTools v2.29.2 or higher"
