---
name: gene-body-coverage-skewness-classification
description: Use when you have per-cell gene body coverage data (Coverage_means_DF_Clust
  derived from coverage.r in the SkewC workflow) and need to partition cells into
  typical vs. skewed groups for quality filtering.
when_to_use_negative:
- Input is raw barcoded BAM that has not yet been split by cell barcode and run through
  gene body coverage computation (2_SkewC.sh expects the coverage outputs).
- You need to pre-filter cells before analysis — that is handled separately by bash
  3_filter.sh $filter $indir $matchdir $unmatchdir, not by the trim clustering step.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: tclust
  role: R function providing the trim clustering that partitions cells into typical
    and skewed groups
  repo: https://github.com/LSBDT/SkewC.git
- name: SkewC
  role: overall pipeline (2_SkewC.sh / SkewC_TrimClustering.Rmd) that prepares inputs
    and runs the trim clustering on gene body coverage
  repo: https://github.com/LSBDT/SkewC.git
- name: R
  role: implementation language in which the trim clustering R Markdown is executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:68d77937f6764907900ca6e588ad1e2a82f7473f61ca048bf1e7b00006718dd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - R
  - SkewC
  - tclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# gene-body-coverage-skewness-classification

## Summary

Classifies individual scRNA-seq cells into typical and skewed groups based on their gene body coverage profiles using R tclust trim clustering, so that cells with skewed coverage (poor quality) can be excluded from downstream analysis.

## When to use

Use when you have per-cell gene body coverage data (Coverage_means_DF_Clust derived from coverage.r in the SkewC workflow) and need to partition cells into typical vs. skewed groups for quality filtering.

## When NOT to use

- Input is raw barcoded BAM that has not yet been split by cell barcode and run through gene body coverage computation (2_SkewC.sh expects the coverage outputs).
- You need to pre-filter cells before analysis — that is handled separately by bash 3_filter.sh $filter $indir $matchdir $unmatchdir, not by the trim clustering step.

## Inputs

- Coverage_means_DF_Clust data frame derived from gene body coverage results stored in coverage.r
- alpha trimming level value in [0..1] (user-specified or auto-approximated optimal)
- per-cell BAM files and gene body coverage outputs from 0_split10XbyBarcode.sh and 1_geneBodyCoverage.sh

## Outputs

- TypicalCellsID.tsv — list of typical cells
- SkewedCellsID.tsv — list of skewed cells
- CLUSTResult clustering plot as a PDF
- HTML report under a directory skewc/

## How to apply

Load the clustering input data frame Coverage_means_DF_Clust, which is derived from the gene body coverage results stored in coverage.r. Supply an alpha trimming level constrained to [0..1] — for large 10X Genomics datasets, e.g. start with alpha = 0.099 and evaluate the result; if the auto-approximation fails the article reports 'No alpha value computation found', so a manual value is needed. Execute the trim clustering implemented in the R tclust function via the R Markdown 3-SkewC_TrimClustering.Rmd, run in the order the SkewC README prescribes. The clustering splits cells into two types: typical cells with prototypical gene body coverage profiles and skewed cells with skewed profiles. Write TypicalCellsID.tsv and SkewedCellsID.tsv and generate the CLUSTResult clustering plot as a PDF.

## Related tools

- **tclust** (R function providing the trim clustering that partitions cells into typical and skewed groups) — https://github.com/LSBDT/SkewC.git
- **SkewC** (overall pipeline (2_SkewC.sh / SkewC_TrimClustering.Rmd) that prepares inputs and runs the trim clustering on gene body coverage) — https://github.com/LSBDT/SkewC.git
- **R** (implementation language in which the trim clustering R Markdown is executed)

## Examples

```
bash 2_SkewC.sh $prjname $indir $outdir $alpha
```

## Evaluation signals

- TypicalCellsID.tsv and SkewedCellsID.tsv together partition all cells present in Coverage_means_DF_Clust.
- No cell is assigned to both TypicalCellsID.tsv and SkewedCellsID.tsv (the two lists are disjoint).
- The alpha value supplied is within the constrained range [0 .. 1].
- A CLUSTResult clustering plot PDF is generated and shows the typical/skewed separation.
- Final HTML output is stored under a directory skewc/ as the README specifies.

## Limitations

- For large 10X Genomics datasets the alpha must be chosen and evaluated manually (e.g. start with alpha = 0.099); the workflow can fail with 'No alpha value computation found'.
- An R package error (stringi/libicui18n.so.57) terminates SkewC and is not fixable from the user's side.
- A mismatch between barcode.tsv and the barcoded BAM produces an empty gene body coverage plot PDF, corrupting downstream clustering input.

## Evidence

- [abstract] The R Markdown SkewC_TrimClustering.Rmd performs the trim clustering implemented in R tclust function.: "The R Markdown SkewC_TrimClustering.Rmd performs the trim clustering implemented in R tclust function."
- [abstract] SkewC distinguishes between two types of single cells: typical cells with prototypical gene body coverage profiles and skewed cells with skewed gene body coverage profiles.: "SkewC distinguishes between two types of single cells: typical cells with prototypical gene body coverage profiles and skewed cells with skewed gene body coverage profiles."
- [methods] bash 2_SkewC.sh $prjname $indir $outdir $alpha: "bash 2_SkewC.sh $prjname $indir $outdir $alpha"
- [readme] This folder contains several R Markdowns and should be run in the same order as listed below: "This folder contains several R Markdowns and should be run in the same order as listed below:"
- [discussion] No alpha value computation found: [evidence span withheld — anchor status: not_found]
