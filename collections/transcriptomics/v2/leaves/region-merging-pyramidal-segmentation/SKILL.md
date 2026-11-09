---
name: region-merging-pyramidal-segmentation
description: Use when a smoothed, gene-order-sorted expression matrix must be converted
  into a copy number segmentation with shared breakpoints across cells, as in SCEVAN's
  joint segmentation step.
when_to_use_negative:
- The input is a raw, unsmoothed count matrix — edge-preserving TV smoothing (logcosh
  prior) must be applied first
- Genes are not sorted by genomic coordinates, since merging assumes adjacent regions
  along the genome
- The goal is malignant vs non-malignant classification only, without clonal structure
  (SUBCLONES = FALSE)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
tools:
- name: SCEVAN
  role: Host R package exposing getBreaksVegaMC, the greedy Mumford–Shah multichannel
    segmentation module invoked via pipelineCNA
  repo: github.com/AntonioDeFalco/SCEVAN
- name: parallelDist
  role: Parallel distance computation supporting region merging across cells
- name: parallel
  role: R parallelization backend used by the pipeline (par_cores, e.g. 20)
- name: pheatmap
  role: Visualization of segmented copy number profiles
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:b245d6f657857a27f187465675f2704cdd3ebd9a7b5a6b659a3e59043f6fddf1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCEVAN
  - parallel
  - parallelDist
  - pheatmap
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Region-merging pyramidal segmentation

## Summary

A greedy piecewise-constant Mumford–Shah minimization, ported from Computer Vision, that jointly segments the smoothed scRNA-seq expression matrix into a copy number matrix by iteratively merging adjacent genomic regions along a pyramid with a dynamic regularization weight lambda.

## When to use

Use when a smoothed, gene-order-sorted expression matrix must be converted into a copy number segmentation with shared breakpoints across cells, as in SCEVAN's joint segmentation step.

## When NOT to use

- The input is a raw, unsmoothed count matrix — edge-preserving TV smoothing (logcosh prior) must be applied first
- Genes are not sorted by genomic coordinates, since merging assumes adjacent regions along the genome
- The goal is malignant vs non-malignant classification only, without clonal structure (SUBCLONES = FALSE)

## Inputs

- Smoothed, relative expression matrix (genes on rows sorted by genomic coordinates, cells on columns)
- Chromosome index vector (chr_vect) mapping each gene to its chromosome
- Sample name (e.g., MGH106)
- beta_vega regularization parameter (default 0.5)

## Outputs

- Copy number (CNA) matrix with segmented regions per cell
- Segmentation output files (.seg) written to output_dir with breakpoints

## How to apply

Load the count matrix (e.g., MGH106 from GSE131928), annotate genes with genomic coordinates, and sort them so regions are adjacent along chromosomes. Initialize the segmentation with n regions (one per gene) and iteratively merge adjacent regions along the pyramid, recomputing region means and dynamically updating the regularization weight lambda at each level. Stop when delta-lambda <= beta*nu, where beta (exposed as beta_vega, default 0.5 in pipelineCNA) is the single segmentation parameter — higher beta yields more coarse-grained segmentation. Wrap the procedure in a function matching the exported signature getBreaksVegaMC(mtx, chr_vect, sample, beta_vega, output_dir) so it can run independently, in the multichannel (all-cells-shared-breakpoints) setting so that cells in a copy number clone share the same breakpoints.

## Related tools

- **SCEVAN** (Host R package exposing getBreaksVegaMC, the greedy Mumford–Shah multichannel segmentation module invoked via pipelineCNA) — github.com/AntonioDeFalco/SCEVAN
- **parallelDist** (Parallel distance computation supporting region merging across cells)
- **parallel** (R parallelization backend used by the pipeline (par_cores, e.g. 20))
- **pheatmap** (Visualization of segmented copy number profiles)

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- Segmentation terminates under the stopping criterion delta-lambda <= beta*nu rather than exhausting regions
- Output .seg and CNA matrix files are written to output_dir with breakpoints shared across cells of the same clone
- Segmented regions are contiguous in genomic coordinates and respect chromosome boundaries via chr_vect
- Changing beta_vega monotonically changes segmentation coarseness (higher beta → fewer, coarser regions)
- Running on the MGH106 matrix with default beta_vega = 0.5 reproduces the expected clonal breakpoints

## Limitations

- beta is the only segmentation parameter and its value is not specified in the article text (default 0.5 comes from the README)
- Greedy pyramid merging may merge across breakpoints at coarse levels, trading sensitivity for speed
- Aneuploidy-based segmentation is unsuitable for tumors with minimal genomic alterations (leukemia, pediatric cancers, Ependymomas)

## Evidence

- [intro] SCEVAN's joint segmentation is based on the Mumford–Shah energy model from Computer Vision, previously applied to copy number alteration detection: "making use of the Mumford–Shah energy model12 that has already been successfully applied to detect copy number alterations in matched tumor–normal pairs"
- [discussion] Greedy procedure: start with one region per gene and merge iteratively: "we follow a greedy procedure. We start with a segmentation having n regions, one for each gene."
- [discussion] beta is the only segmentation parameter: "where β is a positive constant, representing the only parameter of the segmentation algorithm."
- [abstract] Multichannel segmentation assumes shared breakpoints within a clone: "It uses a multichannel segmentation algorithm exploiting the assumption that all the cells in a given copy number clone share the same breakpoints"
- [methods] getBreaksVegaMC signature and pipelineCNA invocation from methods: "`getBreaksVegaMC(mtx, chr_vect, sample, beta_vega, output_dir)`"
- [readme] beta_vega default and semantics from README: [evidence span withheld — anchor status: not_found]
