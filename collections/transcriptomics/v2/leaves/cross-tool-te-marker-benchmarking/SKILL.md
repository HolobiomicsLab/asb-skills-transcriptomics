---
name: cross-tool-te-marker-benchmarking
description: Use when evaluating whether a locus-resolved TE quantification tool outperforms
  a family-level TE tool for detecting cell-type marker TEs in scRNA-Seq clusters.
when_to_use_negative:
- Input is already a gene-only or TE-only expression matrix not derived from the same
  underlying BAM, making the tool comparison invalid
- The dataset has no defined cell clusters or known cell-type structure to serve as
  the basis for marker detection
- Only a single TE tool's output exists and no comparator (e.g., scTE) can be run
  on the same data for benchmarking
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0160
tools:
- name: SoloTE
  role: Locus-level TE expression quantification; produces combined gene+TE matrix
    and the primary marker set being benchmarked
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Family-level comparator tool whose marker output (3 TEs) is contrasted against
    SoloTE's 24 markers
- name: Seurat
  role: Clustering, UMAP, and FindAllMarkers-based marker TE identification with adjusted
    P-value <= 0.05 and avg log2FC >= 1
- name: STAR
  role: Multi-mapping-permissive read alignment producing the BAM input to SoloTE
- name: BEDTools
  role: Assessment of unassigned read overlap with the TE BED annotation within the
    SoloTE pipeline
  repo: https://github.com/arq5x/bedtools2
- name: Samtools
  role: Dependency for BAM handling in the SoloTE pipeline
  repo: http://www.htslib.org/download/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:6049b3b2a6204fb46c64e97b7e18891502d33909b17db627b4a6eaff0e5b807f
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
  - scTE
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-tool-te-marker-benchmarking

## Summary

Benchmarks locus-specific TE analysis (SoloTE) against a family-level comparator (scTE) on the same scRNA-Seq dataset by running both, performing Seurat FindAllMarkers, and comparing the number and resolution of detected cluster marker TEs (e.g., 24 locus-resolved SoloTE markers vs 3 family-level scTE markers in the early gastric cancer dataset).

## When to use

Use when evaluating whether a locus-resolved TE quantification tool outperforms a family-level TE tool for detecting cell-type marker TEs in scRNA-Seq clusters.

## When NOT to use

- Input is already a gene-only or TE-only expression matrix not derived from the same underlying BAM, making the tool comparison invalid
- The dataset has no defined cell clusters or known cell-type structure to serve as the basis for marker detection
- Only a single TE tool's output exists and no comparator (e.g., scTE) can be run on the same data for benchmarking

## Inputs

- STAR-aligned scRNA-Seq BAM file (multi-mapping-permissive alignment)
- hg38 RepeatMasker TE annotation converted to BED format (SoloTE_RepeatMasker_to_BED.py)
- Same dataset processed with scTE for the comparator matrix
- Seurat cell clustering (6 clusters of the early gastric cancer dataset)

## Outputs

- SoloTE locus-level marker TE list (24 markers with adjusted P-value <= 0.05 and avg log2FC >= 1)
- scTE family-level marker TE list (3 markers)
- Genomic coordinates of locus-resolved cluster markers (e.g., L1PA7 chrY:18989629-18990627, THE1D chr8:61579930-61580458)
- Benchmark comparison of marker counts and locus resolution

## How to apply

Run SoloTE on STAR-aligned BAM (multi-mapping-permissive: --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1) to obtain a combined gene+TE matrix, with locus-level assignment for MAPQ >= 255 alignments and family-level for multi-mappers. Run scTE on the same dataset as the family-level comparator. Load each matrix into Seurat, normalize, cluster (reproducing the 6 clusters of the early gastric cancer dataset), and call markers with FindAllMarkers keeping only adjusted P-value <= 0.05 and average log2(Fold Change) >= 1. Compare marker counts (SoloTE yields 24 marker TEs vs 3 for scTE) and confirm that locus-resolved cancer-cluster markers map to specific loci such as L1PA7 (chrY:18989629-18990627) and THE1D (chr8:61579930-61580458). The rationale is that family-level metagene summarization loses genomic location, so a higher count of specific, locus-anchored markers demonstrates the resolution advantage.

## Related tools

- **SoloTE** (Locus-level TE expression quantification; produces combined gene+TE matrix and the primary marker set being benchmarked) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Family-level comparator tool whose marker output (3 TEs) is contrasted against SoloTE's 24 markers)
- **Seurat** (Clustering, UMAP, and FindAllMarkers-based marker TE identification with adjusted P-value <= 0.05 and avg log2FC >= 1)
- **STAR** (Multi-mapping-permissive read alignment producing the BAM input to SoloTE)
- **BEDTools** (Assessment of unassigned read overlap with the TE BED annotation within the SoloTE pipeline) — https://github.com/arq5x/bedtools2
- **Samtools** (Dependency for BAM handling in the SoloTE pipeline) — http://www.htslib.org/download/

## Examples

```
python SoloTE_pipeline.py --threads 8 --bam earlyGC_aligned.bam --teannotation hg38_rm.bed --outputprefix earlyGC --outputdir solote_out
```

## Evaluation signals

- Seurat clustering reproduces the expected 6 cell clusters of the early gastric cancer dataset after UMAP
- SoloTE marker filter (adjusted P-value <= 0.05 and average log2FC >= 1) yields exactly 24 marker TEs
- scTE on the same data yields only 3 marker TEs, confirming the locus-resolution advantage
- The two cancer-cluster markers resolve to specific loci: L1PA7 at chrY:18989629-18990627 and THE1D at chr8:61579930-61580458
- SoloTE marker IDs encode locus coordinates (locus|TE_Subfamily:TE_Family:TE_Class), proving locus-level rather than family-level assignment

## Limitations

- 3' end 10X data with short 100 nt reads cannot uniquely map all TEs, potentially reducing marker sensitivity
- TEs acting as alternative transcription start sites cannot be detected due to the 3' bias
- scTE indexes do not filter non-TE repeats such as Satellites, which can contaminate the comparator matrix
- In organisms with a greater proportion of transpositionally-active TEs, most copies may be intact and multi-mapping, weakening locus-level assignment

## Evidence

- [other] SoloTE identified L1PA7 as a locus-resolved TE marker (located at chrY:18989629-18990627) with higher expression in the Cancer cell cluster: [evidence span withheld — anchor status: not_found]
- [other] Run scTE on the same dataset as the family-level comparator, which yields only 3 marker TEs, confirming the locus-resolution advantage: [evidence span withheld — anchor status: not_found]
- [discussion] marker analysis was carried out using the "FindAllMarkers" function of Seurat, only keeping the results having adjusted P-value ≤0.05 and average log2(Fold Change) ≥1: [evidence span withheld — anchor status: not_found]
- [intro] which summarizes TE expression on metagenes corresponding to TE families, thus, losing the genomic location from which each TE is expressed: "which summarizes TE expression on metagenes corresponding to TE families, thus, losing the genomic location from which each TE is expressed"
- [discussion] TE expression is summarized at the locus-level if the alignments have a MAPQ equal to or greater than a user-defined threshold: "TE expression is summarized at the locus-level if the alignments have a MAPQ equal to or greater than a user-defined threshold"
- [readme] python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory: "python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory"
