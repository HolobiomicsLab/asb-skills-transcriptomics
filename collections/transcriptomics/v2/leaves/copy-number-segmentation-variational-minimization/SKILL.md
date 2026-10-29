---
name: copy-number-segmentation-variational-minimization
description: Use when a smoothed, gene-coordinate-sorted scRNA-seq expression matrix
  (e.g. MGH106 from GSE131928) must be segmented into copy number regions to infer
  clonal CNA profiles, as in SCEVAN steps D and G.
when_to_use_negative:
- The input matrix is a raw count matrix that has not been filtered (cells <200 genes
  / genes <1% cells), cell-cycle (REACTOME) filtered, normal-cell baseline-subtracted,
  and TV-smoothed — segmentation on unsmoothed counts produces artificial segments
- The tumor is expected to have minimal genomic alterations (e.g. leukemias, pediatric
  cancers, ependymomas), where the aneuploidy-based segmentation assumption is unsuitable
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: SCEVAN
  role: R package exposing the greedy Mumford–Shah multichannel segmenter as getBreaksVegaMC
    within pipelineCNA/multiSampleComparisonClonalCN
  repo: github.com/AntonioDeFalco/SCEVAN
- name: parallelDist
  role: Parallelized distance computation supporting the segmentation/clustering computations
- name: parallel
  role: R parallel backend for running the pipeline across cores (par_cores, default
    20)
- name: pheatmap
  role: Heatmap visualization of copy number profiles across cells/clones
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:f348edda4c2d8ba90a1977665dbe01fc038bb239fb9fc5bd68198432ddf1a8ad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCEVAN
  - parallel
  - parallelDist
  - pheatmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# copy-number-segmentation-variational-minimization

## Summary

Greedy piecewise-constant Mumford–Shah minimization (exported in SCEVAN as getBreaksVegaMC) that segments a smoothed, chromosome-indexed expression matrix into a copy number matrix, exploiting shared breakpoints across copy number clones via multichannel segmentation.

## When to use

Use when a smoothed, gene-coordinate-sorted scRNA-seq expression matrix (e.g. MGH106 from GSE131928) must be segmented into copy number regions to infer clonal CNA profiles, as in SCEVAN steps D and G.

## When NOT to use

- The input matrix is a raw count matrix that has not been filtered (cells <200 genes / genes <1% cells), cell-cycle (REACTOME) filtered, normal-cell baseline-subtracted, and TV-smoothed — segmentation on unsmoothed counts produces artificial segments
- The tumor is expected to have minimal genomic alterations (e.g. leukemias, pediatric cancers, ependymomas), where the aneuploidy-based segmentation assumption is unsuitable

## Inputs

- Smoothed, edge-preserving TV (logcosh) smoothed relative expression matrix (genes x cells)
- Vector of chromosome assignments for genes (chr_vect) derived from Ensembl annotation sorted by genomic coordinates
- Sample name (e.g. MGH106)
- Segmentation regularization parameter beta_vega (default 0.5)
- Output directory path

## Outputs

- Copy number matrix (CNA matrix of regions x cells)
- Segmentation output files (.seg) written to the output directory
- Breakpoint coordinates defining copy number regions shared across clonal subpopulations

## How to apply

Load the smoothed relative expression matrix with genes annotated by Ensembl genomic locations and sorted by chromosome coordinates, forming a chromosome-indexed matrix (chr_vect). Implement the greedy Mumford–Shah minimizer: initialize n regions (one per gene) and iteratively merge adjacent regions along the pyramid, recomputing region means and updating the regularization weight lambda dynamically at each level; stop when delta-lambda <= beta*nu. Expose beta as the single segmentation parameter (beta_vega, default 0.5 in SCEVAN; higher beta yields a more coarse-grained segmentation) and wrap the module with the signature getBreaksVegaMC(mtx, chr_vect, sample, beta_vega, output_dir) so it can run independently of the SCEVAN pipeline. Run the multichannel (all-cells-shared-breakpoints) setting so that cells in a given copy number clone share the same breakpoints, and write segmentation output files (e.g. .seg and CNA matrices) to the output directory.

## Related tools

- **SCEVAN** (R package exposing the greedy Mumford–Shah multichannel segmenter as getBreaksVegaMC within pipelineCNA/multiSampleComparisonClonalCN) — github.com/AntonioDeFalco/SCEVAN
- **parallelDist** (Parallelized distance computation supporting the segmentation/clustering computations)
- **parallel** (R parallel backend for running the pipeline across cores (par_cores, default 20))
- **pheatmap** (Heatmap visualization of copy number profiles across cells/clones)

## Examples

```
breaks <- SCEVAN::getBreaksVegaMC(mtx, chr_vect, sample = "MGH106", beta_vega = 0.5, output_dir = "output/")
```

## Evaluation signals

- Termination under the stopping rule delta-lambda <= beta*nu on the MGH106 (GSE131928) matrix without manual intervention
- Segmentation output files are written to the specified output_dir for the named sample
- Number of regions decreases monotonically from the initial n regions (one per gene) as adjacent regions merge along the pyramid
- Varying beta_vega produces the expected granularity effect: higher beta yields more coarse-grained segmentation (fewer, longer regions)
- Multichannel mode yields breakpoints shared by all cells within a copy number clone, consistent with segments at genomic distances <10 Mb being merged within each clone

## Limitations

- beta is the only segmentation parameter and its positive constant value is not fully specified in the article text; results are sensitive to the chosen beta_vega
- Like all aneuploidy-based approaches, it may not be suited to tumors with minimal genomic alterations
- Ground truth copy number from WES/WGS has different resolution than single-cell data and requires resampling before comparison

## Evidence

- [intro] Mumford-Shah energy model basis: "making use of the Mumford–Shah energy model12 that has already been successfully applied to detect copy number alterations in matched tumor–normal pairs"
- [discussion] Greedy initialization procedure: "we follow a greedy procedure. We start with a segmentation having n regions, one for each gene."
- [discussion] beta is the sole segmentation parameter: "where β is a positive constant, representing the only parameter of the segmentation algorithm."
- [methods] Exported function signature: "`getBreaksVegaMC(mtx, chr_vect, sample, beta_vega, output_dir)`"
- [readme] beta_vega semantics and default from README: [evidence span withheld — anchor status: external_doc]
- [readme] Greedy multichannel speed claim from README: [evidence span withheld — anchor status: external_doc]
