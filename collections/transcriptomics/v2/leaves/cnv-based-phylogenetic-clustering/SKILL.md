---
name: cnv-based-phylogenetic-clustering
description: Use when scRNA-seq data (e.g., the 430-cell single-cell GBM dataset)
  require reconstruction of a CNV-based clonal tree and testing of mutually exclusive
  vs co-occurring subclone CNV events across cells.
when_to_use_negative:
- Bulk RNA-seq cohorts (e.g., yale_meningioma, TCGA-GBM/BRCA demos) — the CNV-based
  clonal evolution tree and single-cell mutual exclusivity analysis require per-cell
  CNV profiles, not bulk samples.
- 3′-only sequencing (e.g., DropSeq-style) single-cell data where the BAF shift signal
  is degraded because shifts are measured only for variants near gene 3′ ends, weakening
  the allelic component that supports CNV calls.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0622
tools:
- name: CaSpER
  role: Multiscale HMM CNV calling from scRNA-seq expression plus BAF allelic shift;
    provides gene.matrix, extractMUAndCooccurence (Fisher's exact test), and plotSCellCNVTree
    for the clonal phylogeny.
  repo: https://github.com/akdess/CaSpER
- name: BAFExtract
  role: Generates B-allele frequency shift profiles from mapped RNA-seq reads (no
    heterozygous variant call set needed); supplies the allelic signal CaSpER integrates
    to correct CNV calls before clonal inference.
  repo: https://github.com/akdess/BAFExtract
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:303493e0f4526d4f9571d3373696a1af6f3d8f270bae7a4334410105b14a258a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/CaSpER
  tools:
  - BAFExtract
  - CaSpER
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cnv-based-phylogenetic-clustering

## Summary

Infers CNV-based clonal evolution from single-cell RNA-seq using CaSpER: the per-cell gene-level CNV matrix from multiscale HMM segmentation (integrated with BAF allele-shift signal) is converted into a clonal phylogenetic tree, and subclonal CNV architecture is tested for mutual exclusivity/co-occurrence with Fisher's exact test.

## When to use

Use when scRNA-seq data (e.g., the 430-cell single-cell GBM dataset) require reconstruction of a CNV-based clonal tree and testing of mutually exclusive vs co-occurring subclone CNV events across cells.

## When NOT to use

- Bulk RNA-seq cohorts (e.g., yale_meningioma, TCGA-GBM/BRCA demos) — the CNV-based clonal evolution tree and single-cell mutual exclusivity analysis require per-cell CNV profiles, not bulk samples.
- 3′-only sequencing (e.g., DropSeq-style) single-cell data where the BAF shift signal is degraded because shifts are measured only for variants near gene 3′ ends, weakening the allelic component that supports CNV calls.

## Inputs

- Normalized single-cell RNA-seq expression matrix (e.g., scell_gbm, 430 cells)
- BAFExtract per-sample BAF shift output files with a loh.name.mapping (one-to-many cell-to-patient-pooled-BAF mapping)
- hg19 cytoband file and gene annotation (e.g., from generateAnnotation with ensembl_gene_id)
- final.objects list from runCaSpER (per-scale HMM segments integrated with BAF allele-shift states)

## Outputs

- Cell-by-gene CNV matrix (gene.matrix; rows are cells, columns are genes)
- Chromosome-arm large-scale event matrix (finalChrMat from extractLargeScaleEvents, thr=0.75)
- CNV-based clonal phylogenetic tree of cells (plotSCellCNVTree)
- Mutual exclusivity / co-occurrence event pairs with Fisher's exact test p-values (extractMUAndCooccurence, plotMUAndCooccurence)

## How to apply

Run CaSpER on the normalized single-cell expression matrix (CreateCasperObject with cnv.scale=3, loh.scale=3, method='iterative', matrix.type='normalized') together with patient-pooled BAFExtract BAF shift signal via loh.name.mapping, then runCaSpER(removeCentromere=T, cytoband=hg19 cytoband, method='iterative') to obtain final.objects. Summarize calls with extractLargeScaleEvents(final.objects, thr=0.75) for chromosome-arm events, extractSegmentSummary (gamma=6 consistent-call threshold), and gene.matrix to build the cell-by-gene CNV matrix. Feed the CNV calls to plotSCellCNVTree to generate the CNV-based clonal phylogenetic tree of cells. Detect mutually exclusive and co-occurring CNV alterations with extractMUAndCooccurence (Fisher's exact test) and render with plotMUAndCooccurence. In the GBM data this resolves mutually exclusive subclones within patients — MGH31 (5q amplification vs 14q deletion; 1p amp vs 13q del), MGH28 (8q:20p, 5q:19p), MGH30 (6p:7p) — while normal oligodendrocytes separate from tumor clones.

## Related tools

- **CaSpER** (Multiscale HMM CNV calling from scRNA-seq expression plus BAF allelic shift; provides gene.matrix, extractMUAndCooccurence (Fisher's exact test), and plotSCellCNVTree for the clonal phylogeny.) — https://github.com/akdess/CaSpER
- **BAFExtract** (Generates B-allele frequency shift profiles from mapped RNA-seq reads (no heterozygous variant call set needed); supplies the allelic signal CaSpER integrates to correct CNV calls before clonal inference.) — https://github.com/akdess/BAFExtract

## Examples

```
final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative"); finalChrMat <- extractLargeScaleEvents(final.objects, thr=0.75)
```

## Evaluation signals

- Large-scale CNV calls are consistent across ≥75% (≥7 of 9) pairwise smoothing-scale comparisons (extractLargeScaleEvents thr=0.75).
- Tumor cells show the expected broad aneuploidy (chr7 amplification, chr10 deletion) while normal oligodendrocytes cluster distinctly in the CNV heatmap/tree.
- Within-patient mutually exclusive pairs are recovered: MGH31 (5q amp vs 14q del; 1p amp vs 13q del), MGH28 (8q:20p, 5q:19p), MGH30 (6p:7p), confirmed by Fisher's exact test in extractMUAndCooccurence.
- Segment-level calls agree across scales at the gamma=6 consistency threshold used in extractSegmentSummary.
- The plotSCellCNVTree topology matches the reported clonal architecture derived from the CNV matrix.

## Limitations

- In scRNA-seq, cells with substantially large numbers of reads may dominate the pooled BAF shift signal and create biases in the CNV calls underlying the tree.
- 3′-only sequencing (e.g., DropSeq) degrades BAF shift signal because shifts are measured only for variants located close to the 3′ ends of genes; pooling of reads is believed but not validated to soothe this.
- RNA editing, PCR duplication artefacts, and sequencing errors may impact the BAF shift signal and bias CNV detection in complex ways.
- Ground-truth CNV calls for validating the inferred clone structure may be unknown (as for the MM135 10x dataset), so concordance must be assessed against alternative methods such as HONEYBADGER.

## Evidence

- [intro] CaSpER identifies and visualizes mutually exclusive and co-occurring CNV alterations. For scRNA-seq experiments, it infers the CNV-based clonal evolution of the cells using the detected CNVs.: "CaSpER identifies and visualizes mutually exclusive and co-occurring CNV alterations. For scRNA-seq experiments, it infers the CNV-based clonal evolution of the cells using the detected CNVs."
- [methods] Gene-based CNV summarization builds the cell-by-gene matrix that underpins clustering and tree inference.: "we generate a matrix where rows are the samples (cells) and columns are the genes"
- [discussion] Single-cell GBM analysis empirically unraveled mutually exclusive and co-occurring CNV subclones.: "Analyzing single-cell GBM RNA-seq data using CaSpER, unraveled mutual exclusive and co-occurring CNV subclones"
- [discussion] CaSpER CNV calls can be validated for concordance against HONEYBADGER when ground truth is unavailable.: "we show the concordance of CaSpER CNV calls with HONEYBADGER"
- [methods] The runCaSpER call with iterative method on the casper object produces the final per-scale objects feeding the tree and exclusivity analyses.: "final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative")"
