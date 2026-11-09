---
name: subclone-mutual-exclusivity-analysis
description: Use when single-cell RNA-seq from a heterogeneous tumor (multiple cells
  per patient/sample) should be decomposed into CNV-defined subclones, and mutually
  exclusive vs co-occurring CNV events plus a clonal tree are the target outputs.
when_to_use_negative:
- Input is bulk RNA-seq or DNA data without single-cell resolution — clonal decomposition
  per cell is not possible; use bulk CaSpER workflows instead.
- 'Only 3''-end (e.g. DropSeq-style) scRNA-seq with sparse variants: BAF shift is
  measured only near 3'' gene ends and signal may be too degraded for reliable subclonal
  LOH integration.'
- A DNA- or SNV-based clonal inference tool (e.g. clonealign-style integration) is
  required — this skill infers clones from RNA-seq-derived CNVs and does not use high-quality
  heterozygous variant call sets.
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: CaSpER
  role: Multiscale HMM CNV caller from RNA-seq; provides CreateCasperObject, runCaSpER,
    extractLargeScaleEvents, extractMUAndCooccurence, plotSCellTree for subclone and
    clonal-tree inference
  repo: https://github.com/akdess/CaSpER
- name: BAFExtract
  role: Generates B-allele frequency shift profiles from mapped reads without a heterozygous
    variant call set, feeding the LOH/allele-shift channel used for subclonal CNV
    states
  repo: https://github.com/akdess/BAFExtract
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:bc62b9aa5f94027c8d3f19b9ae1e3a0ab8c5479edc9ba85d3dbb3d002b1d029f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/CaSpER
  tools:
  - BAFExtract
  - CaSpER
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# subclone-mutual-exclusivity-analysis

## Summary

Infers subclonal CNV architecture from single-cell RNA-seq using CaSpER's iterative HMM segmentation with patient-pooled BAF shift signal, then tests mutually exclusive and co-occurring CNV events (Fisher's exact test) and reconstructs a CNV-based clonal tree, as demonstrated on the 430-cell GBM dataset (MGH28/MGH30/MGH31).

## When to use

Use when single-cell RNA-seq from a heterogeneous tumor (multiple cells per patient/sample) should be decomposed into CNV-defined subclones, and mutually exclusive vs co-occurring CNV events plus a clonal tree are the target outputs.

## When NOT to use

- Input is bulk RNA-seq or DNA data without single-cell resolution — clonal decomposition per cell is not possible; use bulk CaSpER workflows instead.
- Only 3'-end (e.g. DropSeq-style) scRNA-seq with sparse variants: BAF shift is measured only near 3' gene ends and signal may be too degraded for reliable subclonal LOH integration.
- A DNA- or SNV-based clonal inference tool (e.g. clonealign-style integration) is required — this skill infers clones from RNA-seq-derived CNVs and does not use high-quality heterozygous variant call sets.

## Inputs

- Single-cell RNA-seq expression matrix (e.g. scell_gbm, 430 cells; normalized matrix, matrix.type='normalized')
- Mapped single-cell RNA-seq BAM files for BAF extraction
- Per-patient pooled BAFExtract output files (read via readBAFExtractOutput)
- hg19 cytoband table, centromere coordinates, and gene annotation (Ensembl gene IDs via generateAnnotation)
- loh.name.mapping data frame mapping each cell to its patient-pooled BAF signal

## Outputs

- Cell-by-chromosome-arm large-scale CNV event matrix (finalChrMat)
- Segment-level CNV summary (segment.summary, gamma=6 consistency threshold)
- Cell-by-gene CNV matrix (gene.matrix)
- Mutually exclusive and co-occurring CNV event pairs with Fisher's exact test statistics (e.g. MGH31: 5q amp vs 14q del; 1p amp vs 13q del)
- CNV-based clonal phylogenetic tree (plotSCellCNVTree)
- Heatmaps of large-scale events, smoothed expression signal, and BAF shift per sample

## How to apply

Merge single-cell BAM files per study and extract BAF values with BAFExtract; for each patient, pool the per-cell BAFExtract output so every cell maps to its patient-level BAF shift signal via loh.name.mapping (one-to-many mapping). Build the casper object with CreateCasperObject (cnv.scale=3, loh.scale=3, method='iterative', matrix.type='normalized') together with the hg19 cytoband, gene annotation (generateAnnotation), and BAF files, then run runCaSpER(object, removeCentromere=T, cytoband=cytoband, method='iterative') which runs HMM on all smoothed expression scales and integrates allele-frequency shift states. Summarize calls with extractLargeScaleEvents(final.objects, thr=0.75) for chromosome-arm events (requires consistency in ≥75%, i.e. ≥7 of 9, pairwise scale comparisons), extractSegmentSummary with gamma=6 for segment-level calls, and gene.matrix for the cell-by-gene CNV matrix. Detect subclones with extractMUAndCooccurence (Fisher's exact test) and visualize with plotMUAndCooccurence, plotSingleCellLargeScaleEventHeatmap, plotHeatmap, and plotBAFOneSample; finally render the CNV-based clonal phylogeny with plotSCellCNVTree.

## Related tools

- **CaSpER** (Multiscale HMM CNV caller from RNA-seq; provides CreateCasperObject, runCaSpER, extractLargeScaleEvents, extractMUAndCooccurence, plotSCellTree for subclone and clonal-tree inference) — https://github.com/akdess/CaSpER
- **BAFExtract** (Generates B-allele frequency shift profiles from mapped reads without a heterozygous variant call set, feeding the LOH/allele-shift channel used for subclonal CNV states) — https://github.com/akdess/BAFExtract

## Examples

```
final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative"); mu <- extractMUAndCooccurence(final.objects)
```

## Evaluation signals

- Large-scale event calls must be consistent across ≥75% (≥7 of 9) pairwise expression-scale comparisons (extractLargeScaleEvents thr=0.75).
- Tumor cells reproduce the reported GBM pattern: chr7 amplification and chr10 deletion; normal oligodendrocytes cluster distinctly from tumor cells.
- Recovered mutually exclusive pairs match published topology: MGH31 (5q amp vs 14q del; 1p amp vs 13q del), MGH28 (8q vs 20p, 5q vs 19p), MGH30 (6p vs 7p).
- Clonal tree from plotSCellCNVTree is concordant with the reported CNV-based clonal topology.
- plotBAFOneSample plots show BAF shift evidence supporting the CNV states assigned per chromosome arm.

## Limitations

- 3'-only sequencing (e.g. DropSeq) degrades the BAF shift signal because variants are measured only near 3' ends of genes.
- Cells with substantially large read counts may dominate the patient-pooled BAF shift signal, creating biases.
- RNA editing, PCR duplication artefacts, and sequencing errors may bias CNV/LOH detection in complex ways.
- Mutual exclusivity/co-occurrence analysis in single-cell sequencing has not been studied extensively, so interpretation lacks strong precedent.
- Some cells may lack sufficient reads; BAF pooling per patient assumes the pooled signal soothes 3' bias with a decent number of reads, which was not validated.

## Evidence

- [results] CaSpER identified two mutually exclusive subclones in patient MGH31: one subclone harbors chromosome 5q amplification, while the other harbors chromosome 14q deletion: "Analyzing single-cell GBM RNA-seq data using CaSpER, unraveled mutual exclusive and co-occurring CNV subclones"
- [methods] finalChrMat <- extractLargeScaleEvents(final.objects, thr=0.75): "The main functions you will need to use to identify CNV events are CreateCasperObject() and runCaSpER(casper_object)."
- [methods] We assign a large-scale CNV call to every chromosome arm for each of the N×M pairwise scale comparisons.: "Harmonization and Summarization of CNV calls from multiple scales and from multiple pairwise comparison of BAF and Expression Signals"
- [intro] CaSpER identifies and visualizes mutually exclusive and co-occurring CNV alterations. For scRNA-seq experiments, it infers the CNV-based clonal evolution of the cells using the detected CNVs.: "CaSpER identifies and visualizes mutually exclusive and co-occurring CNV alterations. For scRNA-seq experiments, it infers the CNV-based clonal evolution of the cells using the detected CNVs."
- [readme] BAFExtract generates the B-allele frequency shifts from mapped reads alone without the need for a heterozygous variant call set.: [evidence span withheld — anchor status: external_doc]
