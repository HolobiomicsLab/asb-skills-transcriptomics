---
name: tclust-alpha-parameter-tuning
description: Use when running SkewC's 2_SkewC.sh trim-clustering step and you need
  to choose or adjust the alpha trimming proportion (e.g., alpha=0.04 with k=1 for
  the 1k Brain Cells E18 Mouse dataset) to control the typical/skewed cell split.
when_to_use_negative:
- You have not yet computed per-cell gene body coverage (2_SkewC.sh requires the coverage
  output of 1_geneBodyCoverage.sh as input)
- You want to filter cells by criteria other than gene body coverage skewness — use
  SkewC's 3_filter.sh pre-filtering step instead
- Your cells' BAM files are not yet split per barcode and coverage cannot be computed
  per cell
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3577
tools:
- name: SkewC
  role: Pipeline whose 2_SkewC.sh script runs the trim clustering analysis of gene
    body coverage with the user-supplied alpha parameter
  repo: https://github.com/LSBDT/SkewC.git
- name: tclust
  role: R trim clustering function invoked by SkewC_TrimClustering.Rmd to separate
    typical from skewed cells using the alpha trimming proportion
  repo: https://github.com/LSBDT/SkewC.git
- name: R
  role: Implementation language hosting the SkewC R Markdowns that perform the trim
    clustering
  repo: https://github.com/LSBDT/SkewC.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:241d38859daa08286a392c587d5ef0aa5dcb908deb589d9d970d323eb24374a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - R
  - SkewC
  - tclust
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tclust-alpha-parameter-tuning

## Summary

Tuning the tclust trim-clustering alpha parameter in SkewC, which controls the proportion of cells trimmed as outliers when separating typical from skewed gene body coverage profiles in scRNA-seq quality assessment. The alpha value directly determines how many cells are flagged as skewed.

## When to use

Use when running SkewC's 2_SkewC.sh trim-clustering step and you need to choose or adjust the alpha trimming proportion (e.g., alpha=0.04 with k=1 for the 1k Brain Cells E18 Mouse dataset) to control the typical/skewed cell split.

## When NOT to use

- You have not yet computed per-cell gene body coverage (2_SkewC.sh requires the coverage output of 1_geneBodyCoverage.sh as input)
- You want to filter cells by criteria other than gene body coverage skewness — use SkewC's 3_filter.sh pre-filtering step instead
- Your cells' BAM files are not yet split per barcode and coverage cannot be computed per cell

## Inputs

- Per-cell gene body coverage files (.geneBodyCoverage, .geneBodyCoverage.txt, .log) in a coverage input directory
- Alpha trimming proportion (e.g., 0.04)
- Number of clusters k (e.g., k=1)
- Project name and output directory paths for 2_SkewC.sh

## Outputs

- TypicalCellsID.tsv listing cells with prototypical gene body coverage
- SkewedCellsID.tsv listing cells with skewed gene body coverage
- Index HTML report in the skewc/ directory with typical and skewed gene body coverage plots

## How to apply

Run bash 2_SkewC.sh $prjname $indir $outdir $alpha, which invokes the SkewC_TrimClustering.Rmd to apply the R tclust trim clustering function over per-cell gene body coverage profiles computed by geneBodyCoverage.pl. Alpha sets the fraction of cells tclust trims as outliers; the article's worked example uses alpha=0.04 with k=1 on the 1k Brain Cells from an E18 Mouse dataset (n = 930), yielding 897 typical and 33 skewed cells. Adjust alpha upward to flag more cells as skewed (larger trimmed fraction) or downward to be more permissive; note that a failed alpha computation surfaces the error 'No alpha value computation found'. Validate each run by checking that TypicalCellsID.tsv and SkewedCellsID.tsv counts sum to the number of input cells and inspecting the typical/skewed gene body coverage plots in the index HTML under the skewc/ output directory.

## Related tools

- **SkewC** (Pipeline whose 2_SkewC.sh script runs the trim clustering analysis of gene body coverage with the user-supplied alpha parameter) — https://github.com/LSBDT/SkewC.git
- **tclust** (R trim clustering function invoked by SkewC_TrimClustering.Rmd to separate typical from skewed cells using the alpha trimming proportion) — https://github.com/LSBDT/SkewC.git
- **R** (Implementation language hosting the SkewC R Markdowns that perform the trim clustering) — https://github.com/LSBDT/SkewC.git

## Examples

```
bash 2_SkewC.sh $prjname $indir $outdir $alpha
```

## Evaluation signals

- With alpha=0.04 and k=1 on the 1k Brain Cells E18 Mouse dataset (n = 930), tclust should classify 897 typical cells and 33 skewed cells
- Counts in TypicalCellsID.tsv and SkewedCellsID.tsv must sum to the total number of input cells (930)
- The skewc/ output directory must contain the index HTML with gene body coverage plots for both typical and skewed cells
- No 'No alpha value computation found' error should appear in the run logs
- Skewed cells should visibly show skewed gene body coverage profiles in the plotted output, distinct from prototypical profiles

## Limitations

- A mis-match between barcode names in barcodes.tsv and the barcoded BAM produces empty gene body coverage plots, which corrupts the clustering input
- The stringi/libicui18n.so.57 R package error terminates SkewC and is not fixable from the user's side
- SkewC requires git and at least one of docker, udocker, or singularity installed; gene body coverage computation is time-consuming (about 3 h for 500 cells on 10 cores)

## Evidence

- [abstract] Run bash 2_SkewC.sh to analyze gene body coverage.: [evidence span withheld — anchor status: not_found]
- [methods] bash 2_SkewC.sh $prjname $indir $outdir $alpha: [evidence span withheld — anchor status: not_found]
- [abstract] The R Markdown SkewC_TrimClustering.Rmd performs the trim clustering implemented in R tclust function.: "The R Markdown SkewC_TrimClustering.Rmd performs the trim clustering implemented in R tclust function."
- [discussion] No alpha value computation found".: [evidence span withheld — anchor status: not_found]
- [results] Running SkewC on the 1k Brain Cells from an E18 Mouse dataset (n = 930) yields 897 typical cells and 33 skewed cells: [evidence span withheld — anchor status: not_found]
- [readme] * Final HTML output will be stored under a directory skewc/: "* Final HTML output will be stored under a directory skewc/"
