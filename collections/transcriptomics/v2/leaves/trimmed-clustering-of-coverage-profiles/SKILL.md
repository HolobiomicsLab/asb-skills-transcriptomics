---
name: trimmed-clustering-of-coverage-profiles
description: Use when per-cell gene body coverage profiles (e.g. .geneBodyCoverage
  outputs from geneBodyCoverage.pl) are available and cells must be classified into
  typical vs skewed groups with a robust, trimming-based clustering.
when_to_use_negative:
- Per-cell gene body coverage has not yet been computed (coverage.r / .geneBodyCoverage
  outputs are missing); run 0_split10XbyBarcode.sh and 1_geneBodyCoverage.sh first
- The goal is to pre-filter unwanted cells before coverage computation rather than
  classify cells after coverage — use the SkewC batch command 3_filter.sh instead
- The BAM files are already split per cell and downstream analysis only needs the
  coverage plots, not typical/skewed classification
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SkewC
  role: Framework whose SkewC_TrimClustering.Rmd runs the trim clustering and writes
    the typical/skewed cell ID files
  repo: https://github.com/LSBDT/SkewC.git
- name: tclust
  role: R function performing the trim clustering over the gene body coverage profiles
- name: R
  role: Language in which SkewC_TrimClustering.Rmd is implemented and executed
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:17511b11044a3360a6f6208b3bbce35f921dd8c6e8b4ad6fb41f43a88265f09c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - R
  - SkewC
  - tclust
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Trimmed clustering of coverage profiles

## Summary

SkewC's tclust-based trim clustering partitions single cells into typical and skewed groups based on their gene body coverage profiles, using a user-specified trimming level (alpha) to robustly down-weight outlier cells.

## When to use

Use when per-cell gene body coverage profiles (e.g. .geneBodyCoverage outputs from geneBodyCoverage.pl) are available and cells must be classified into typical vs skewed groups with a robust, trimming-based clustering.

## When NOT to use

- Per-cell gene body coverage has not yet been computed (coverage.r / .geneBodyCoverage outputs are missing); run 0_split10XbyBarcode.sh and 1_geneBodyCoverage.sh first
- The goal is to pre-filter unwanted cells before coverage computation rather than classify cells after coverage — use the SkewC batch command 3_filter.sh instead
- The BAM files are already split per cell and downstream analysis only needs the coverage plots, not typical/skewed classification

## Inputs

- Coverage_means_DF_Clust data frame derived from per-cell gene body coverage results stored in coverage.r
- User-specified alpha trimming level in [0..1] (or auto-approximated optimal alpha)
- Number of clusters k (e.g. k=1 for the 1k Brain Cells from an E18 Mouse test)

## Outputs

- TypicalCellsID.tsv — list of typical cells
- SkewedCellsID.tsv — list of skewed cells
- CLUSTResult clustering plot as a PDF

## How to apply

Load the clustering input data frame Coverage_means_DF_Clust, derived from the per-cell gene body coverage results stored in coverage.r, into the SkewC_TrimClustering.Rmd R Markdown. Choose a trimming level alpha constrained to [0..1] — for large 10X Genomics datasets such as pbmc_8k, starting with alpha = 0.099 is suggested, while the 1k Brain Cells from an E18 Mouse test used alpha = 0.04 — or supply an auto-approximated optimal trimming level if available. Run the R tclust trim clustering (k=1) over the coverage profiles, which treats cells far from the prototypical coverage shape as skewed rather than forcing them into a cluster. Write the two outputs TypicalCellsID.tsv (typical cells) and SkewedCellsID.tsv (skewed cells) alongside a CLUSTResult clustering plot as a PDF. Validate that the two lists partition all cells in Coverage_means_DF_Clust with no overlap — e.g. the E18 mouse run (n = 930) yields 897 typical and 33 skewed cells.

## Related tools

- **SkewC** (Framework whose SkewC_TrimClustering.Rmd runs the trim clustering and writes the typical/skewed cell ID files) — https://github.com/LSBDT/SkewC.git
- **tclust** (R function performing the trim clustering over the gene body coverage profiles)
- **R** (Language in which SkewC_TrimClustering.Rmd is implemented and executed)

## Examples

```
bash 2_SkewC.sh myproject input skewc 0.04
```

## Evaluation signals

- TypicalCellsID.tsv and SkewedCellsID.tsv together partition all cells present in Coverage_means_DF_Clust
- No cell appears in both TypicalCellsID.tsv and SkewedCellsID.tsv
- On the 1k Brain Cells from an E18 Mouse dataset (n = 930) with alpha=0.04 and k=1, expect 897 typical and 33 skewed cells
- A CLUSTResult clustering plot PDF is generated alongside the two .tsv files
- The supplied alpha value lies within the constrained range [0..1]

## Limitations

- For large 10X Genomics datasets the alpha value may need iterative evaluation (e.g. start with alpha = 0.099 and assess the result); a 'No alpha value computation found' error can occur when the optimal alpha cannot be determined
- Running the R Markdowns out of order will fail — they must be run in the same order as listed in the SkewC folder
- SkewC execution may terminate due to an R package error (stringi/libicui18n.so.57) that is not fixable from the user's side

## Evidence

- [abstract] SkewC_TrimClustering.Rmd performs the trim clustering implemented in R tclust function.: "The R Markdown SkewC_TrimClustering.Rmd performs the trim clustering implemented in R tclust function."
- [other] Running SkewC on the 1k Brain Cells from an E18 Mouse dataset (n = 930) yields 897 typical cells and 33 skewed cells.: [evidence span withheld — anchor status: not_found]
- [methods] bash 2_SkewC.sh $prjname $indir $outdir $alpha: "bash 2_SkewC.sh $prjname $indir $outdir $alpha"
- [readme] Final HTML output will be stored under a directory skewc/: "Final HTML output will be stored under a directory skewc/"
- [other] for large 10X Genomics datasets, e.g. start with alpha = 0.099 and evaluate the result: [evidence span withheld — anchor status: not_found]
