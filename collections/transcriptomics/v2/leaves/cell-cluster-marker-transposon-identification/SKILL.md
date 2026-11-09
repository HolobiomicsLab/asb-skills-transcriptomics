---
name: cell-cluster-marker-transposon-identification
description: Use when you have a processed scRNA-Seq BAM file and need to find which
  TEs (at locus resolution) are differentially expressed markers of specific cell
  clusters, e.g. cancer vs normal cell types.
when_to_use_negative:
- You only need family-level TE abundance and do not care about the genomic locus
  — scTE's family-level metagenes may suffice
- Your reads are 3'-end 10X short reads and the TEs of interest are very young/recently
  active copies that cannot be uniquely mappable with ~100 nt reads
- Your expression matrix is already a final gene+TE feature table — this skill covers
  building that matrix, not consuming it
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: SoloTE
  role: 'Core pipeline: selects unassigned reads, intersects with TE annotation via
    BEDTools, and outputs the combined gene+TE expression matrix'
  repo: https://github.com/bvaldebenitom/SoloTE
- name: STAR
  role: Aligner producing the input BAM with multi-mapping-permissive options
- name: BEDTools
  role: Assesses overlap of gene-unassigned reads with the TE annotation (v2.29.2
    or higher)
  repo: https://github.com/arq5x/bedtools2
- name: Samtools
  role: Dependency for BAM handling in the SoloTE pipeline (v1.16 or higher)
- name: Seurat
  role: Normalization, clustering, UMAP, and FindAllMarkers marker-TE identification
    on the combined matrix
- name: SoloTE_RepeatMasker_to_BED.py
  role: Helper to download RepeatMasker files from UCSC and convert to SoloTE BED
    format
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Family-level comparator benchmark for marker TE detection
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:236560d8867997ff468a8faa05f58ec833ed96ecc2e1c6706065f5e5f113771e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - Pandas
  - Pysam
  - R
  - STAR
  - Samtools
  - Seurat
  - SoloTE
  - SoloTE_RepeatMasker_to_BED.py
  - scTE
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 21
---

# cell-cluster-marker-transposon-identification

## Summary

Identifies transposable element markers for cell clusters in scRNA-Seq data by building a combined gene+TE expression matrix with SoloTE (locus-level for uniquely mapped reads, family-level for multi-mappers) and running Seurat's FindAllMarkers with adjusted P-value ≤0.05 and average log2(FC) ≥1.

## When to use

Use when you have a processed scRNA-Seq BAM file and need to find which TEs (at locus resolution) are differentially expressed markers of specific cell clusters, e.g. cancer vs normal cell types.

## When NOT to use

- You only need family-level TE abundance and do not care about the genomic locus — scTE's family-level metagenes may suffice
- Your reads are 3'-end 10X short reads and the TEs of interest are very young/recently active copies that cannot be uniquely mappable with ~100 nt reads
- Your expression matrix is already a final gene+TE feature table — this skill covers building that matrix, not consuming it

## Inputs

- BAM file from STAR or CellRanger alignment of scRNA-Seq reads (multi-mapping-permissive STAR options recommended)
- RepeatMasker TE annotation in BED format (chrom, start, end, locus|TE_Subfamily:TE_Family:TE_Class ID, strand, optional score) for hg38 or other genomes
- Seurat object of cell clusters (from normalization, clustering, and UMAP on the combined gene+TE matrix)

## Outputs

- Combined gene and TE expression matrix per cell, compatible with Seurat
- Locus-resolved cluster marker TE list with adjusted P-values and average log2 fold changes (filtered at adj. P ≤0.05, log2FC ≥1)
- Genomic loci of marker TEs (e.g. L1PA7 at chrY:18989629-18990627, THE1D at chr8:61579930-61580458 in the early gastric cancer dataset)

## How to apply

Align reads with STAR using multi-mapping-permissive options (--winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1), convert the hg38 RepeatMasker annotation to BED with SoloTE_RepeatMasker_to_BED.py (ID = locus|TE_Subfamily:TE_Family:TE_Class|strand), then run SoloTE_pipeline.py to select reads not assigned to genes, assess overlap with the TE annotation via BEDTools, and summarize TE expression at locus level for MAPQ ≥ 255 alignments and at family level for multi-mappers. Load the resulting combined gene+TE matrix into Seurat, normalize, cluster, and perform UMAP reduction. Run FindAllMarkers keeping only results with adjusted P-value ≤0.05 and average log2(Fold Change) ≥1 to obtain cluster marker TEs (e.g. 24 marker TEs in the early gastric cancer dataset vs only 3 with the family-level scTE comparator).

## Related tools

- **SoloTE** (Core pipeline: selects unassigned reads, intersects with TE annotation via BEDTools, and outputs the combined gene+TE expression matrix) — https://github.com/bvaldebenitom/SoloTE
- **STAR** (Aligner producing the input BAM with multi-mapping-permissive options)
- **BEDTools** (Assesses overlap of gene-unassigned reads with the TE annotation (v2.29.2 or higher)) — https://github.com/arq5x/bedtools2
- **Samtools** (Dependency for BAM handling in the SoloTE pipeline (v1.16 or higher))
- **Seurat** (Normalization, clustering, UMAP, and FindAllMarkers marker-TE identification on the combined matrix)
- **SoloTE_RepeatMasker_to_BED.py** (Helper to download RepeatMasker files from UCSC and convert to SoloTE BED format) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Family-level comparator benchmark for marker TE detection)

## Examples

```
python SoloTE_pipeline.py --threads 8 --bam aligned.bam --teannotation hg38_rm.bed --outputprefix EGC --outputdir solote_out
```

## Evaluation signals

- The combined matrix contains both gene and TE rows and loads into Seurat without error
- In the early gastric cancer dataset, the analysis reproduces 6 cell clusters after UMAP
- FindAllMarkers with adj. P ≤0.05 and avg log2FC ≥1 yields 24 marker TEs (vs only 3 with scTE family-level analysis)
- Two cancer-cluster markers resolve to L1PA7 (chrY:18989629-18990627) and THE1D (chr8:61579930-61580458)
- Locus-level TE summaries come only from alignments with MAPQ ≥ 255

## Limitations

- In organisms with a larger proportion of transpositionally-active TEs, most copies could be genetically intact and multi-mapping, weakening locus resolution
- With 3' end 10X ~100 nt reads, not all TEs can be uniquely mappable, and TEs acting as alternative transcription start sites cannot be detected due to the 3' bias
- Locus-level summaries are restricted to uniquely mapped reads at the default MAPQ 255 threshold; multi-mappers are only summarized at family level
- Marker TE findings are computational; no gain- or loss-of-function experimental validation was performed

## Evidence

- [abstract] SoloTE, a pipeline that outperforms the previous approach in terms of computational resources and by allowing the inclusion of locus-specific TE activity in scRNA-Seq expression matrixes: "SoloTE, a pipeline that outperforms the previous approach in terms of computational resources and by allowing the inclusion of locus-specific TE activity in scRNA-Seq expression matrixes"
- [discussion] marker analysis was carried out using the "FindAllMarkers" function of Seurat, only keeping the results having adjusted P-value ≤0.05 and average log2(Fold Change) ≥1: "only keeping the results having adjusted P-value ≤0.05 and average log2(Fold Change) ≥1"
- [discussion] we recommend using STAR with options --winAnchorMultimapNmax 100 ... --outFilterMultimapNmax 100 ... --outSAMmultNmax 1: "we recommend using STAR with options --winAnchorMultimapNmax 100"
- [readme] README command line for running the pipeline: "python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory"
- [readme] README on TE annotation ID format: "So, column 4, the ID, is a concatenation of the locus of the TE and its identifiers at the Subfamily, Family and Class level."
