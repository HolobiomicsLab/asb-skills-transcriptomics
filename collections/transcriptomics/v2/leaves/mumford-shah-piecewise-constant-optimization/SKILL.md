---
name: mumford-shah-piecewise-constant-optimization
description: Use when you must segment a chromosome-ordered smoothed scRNA-seq expression
  matrix into copy number regions, exploiting shared breakpoints across cells/clones
  via multichannel Mumford–Shah minimization.
when_to_use_negative:
- Tumors with minimal genomic alterations (e.g. leukemias, pediatric cancers, Ependymomas),
  where the aneuploidy-based segmentation assumption is unsuited
- Unsorted gene expression matrices without genomic coordinate annotation, since the
  algorithm merges only adjacent regions along the genome
- Normal/tumor classification only, with no need for clonal copy number segmentation
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0204
tools:
- name: SCEVAN
  role: R package exposing the greedy Mumford–Shah segmentation as getBreaksVegaMC
    within the pipelineCNA copy number workflow
  repo: github.com/AntonioDeFalco/SCEVAN
- name: parallelDist
  role: Parallelized distance computation supporting the cell/clone clustering around
    the segmentation
- name: parallel
  role: Multi-core execution of the segmentation and classification pipeline (par_cores,
    default 20)
- name: pheatmap
  role: Heatmap visualization of the segmented copy number profiles
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:fa3ff026a500011b31272e4685abae4a4b631721482111ea6d17ed7a24730721
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCEVAN
  - parallel
  - parallelDist
  - pheatmap
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# mumford-shah-piecewise-constant-optimization

## Summary

Greedy piecewise-constant minimization of the Mumford–Shah energy from Computer Vision, used in SCEVAN as the joint multichannel segmentation step that converts the smoothed per-cell expression matrix into a copy number matrix with shared breakpoints across clonal subpopulations.

## When to use

Use when you must segment a chromosome-ordered smoothed scRNA-seq expression matrix into copy number regions, exploiting shared breakpoints across cells/clones via multichannel Mumford–Shah minimization.

## When NOT to use

- Tumors with minimal genomic alterations (e.g. leukemias, pediatric cancers, Ependymomas), where the aneuploidy-based segmentation assumption is unsuited
- Unsorted gene expression matrices without genomic coordinate annotation, since the algorithm merges only adjacent regions along the genome
- Normal/tumor classification only, with no need for clonal copy number segmentation

## Inputs

- Smoothed (edge-preserving TV logcosh) relative expression matrix from scRNA-seq counts, genes sorted by genomic coordinates
- Chromosome index vector (chr_vect) mapping each gene to its chromosome
- Sample name (e.g. MGH106)
- Segmentation parameter beta_vega (default 0.5)
- Output directory path

## Outputs

- Copy number matrix (CNAmtx) with regions as rows and cells as columns
- Segmentation output files (.seg) written to the output directory
- Chromosome-annotated matrix (e.g. MGH106_count_mtx_annot.RData)

## How to apply

Load the chromosome-indexed expression matrix (e.g. MGH106 from GSE131928) with genes annotated and sorted by genomic coordinates. Implement the greedy minimizer: initialize the segmentation with n regions (one per gene), then iteratively merge adjacent regions along the pyramid, recomputing region means and dynamically updating the regularization weight lambda at each level. Terminate when delta-lambda <= beta*nu, exposing beta as the single segmentation parameter (beta_vega; higher beta yields a more coarse-grained segmentation, default 0.5 per the README). Wrap the routine with the exported signature getBreaksVegaMC(mtx, chr_vect, sample, beta_vega, output_dir) so it runs independently of the rest of the SCEVAN pipeline; in the multichannel setting all cells in a copy number clone share the same breakpoints, which makes the procedure fast even for large datasets.

## Related tools

- **SCEVAN** (R package exposing the greedy Mumford–Shah segmentation as getBreaksVegaMC within the pipelineCNA copy number workflow) — github.com/AntonioDeFalco/SCEVAN
- **parallelDist** (Parallelized distance computation supporting the cell/clone clustering around the segmentation)
- **parallel** (Multi-core execution of the segmentation and classification pipeline (par_cores, default 20))
- **pheatmap** (Heatmap visualization of the segmented copy number profiles)

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE); segments <- getBreaksVegaMC(mtx, chr_vect, "MGH106", beta_vega = 0.5, output_dir)
```

## Evaluation signals

- The greedy loop terminates under the stopping rule delta-lambda <= beta*nu rather than diverging or running to a single region
- The module writes segmentation output files (.seg, CNAmtx) to the specified output directory when run on the MGH106 matrix in multichannel mode
- Resulting segments are piecewise constant: every gene in a region shares the same region mean
- Varying beta_vega produces the expected granularity shift — higher beta yields a more coarse-grained segmentation
- Breakpoints are shared across all cells of a given copy number clone (multichannel invariant)

## Limitations

- beta is the only segmentation parameter and its article-level value was not stated in the text (README gives default 0.5), so tuning may be required per dataset
- Aneuploidy-based segmentation is unsuitable for tumors with minimal genomic alterations such as leukemias, pediatric cancers, and Ependymomas
- Segments representing the same CNA at genomic distances < 10 Mb must be merged post hoc, so raw segments can be fragmented

## Evidence

- [intro] SCEVAN's joint segmentation is based on the Mumford–Shah energy model from Computer Vision, previously applied to copy number alteration detection: "making use of the Mumford–Shah energy model12 that has already been successfully applied to detect copy number alterations in matched tumor–normal pairs"
- [discussion] Greedy initialization with one region per gene: "we follow a greedy procedure. We start with a segmentation having n regions, one for each gene."
- [methods] getBreaksVegaMC signature: "`getBreaksVegaMC(mtx, chr_vect, sample, beta_vega, output_dir)`"
- [readme] beta_vega meaning and default: [evidence span withheld — anchor status: not_found]
- [abstract] Multichannel shared-breakpoint assumption: "It uses a multichannel segmentation algorithm exploiting the assumption that all the cells in a given copy number clone share the same breakpoints"
