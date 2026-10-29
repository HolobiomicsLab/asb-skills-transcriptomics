---
name: cell-partition-consistency-verification
description: Use when SkewC_TrimClustering.Rmd (R tclust) has produced TypicalCellsID.tsv
  and SkewedCellsID.tsv and you must verify the two lists cover every cell exactly
  once.
when_to_use_negative:
- The clustering outputs have not yet been generated (e.g. 3-SkewC_TrimClustering.Rmd
  failed with 'No alpha value computation found' or the stringi/libicui18n.so.57 R
  package error)
- The cell identifiers in the .tsv files do not match the barcode conventions of Coverage_means_DF_Clust
  (e.g. barcode mismatch between barcode.tsv and the barcoded BAM file, which yields
  empty/incorrect outputs)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SkewC
  role: Produces the typical/skewed cell lists that this verification step checks
    for a disjoint, complete partition
  repo: https://github.com/LSBDT/SkewC.git
- name: tclust
  role: R trim clustering function whose alpha-trimmed assignment of Coverage_means_DF_Clust
    yields the two cell groups being verified
- name: R
  role: Language in which the clustering and output verification are implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:efb1c2641bc38ed0d3627bbe7ac8d6f1dd886444c8d988eb1d1daf5840d9114e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LSBDT/SkewC.git
  tools:
  - R
  - SkewC
  - tclust
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cell-partition-consistency-verification

## Summary

Validation step for SkewC trim clustering that confirms the outputs TypicalCellsID.tsv and SkewedCellsID.tsv form a disjoint partition of all cells in the input clustering data frame Coverage_means_DF_Clust.

## When to use

Use when SkewC_TrimClustering.Rmd (R tclust) has produced TypicalCellsID.tsv and SkewedCellsID.tsv and you must verify the two lists cover every cell exactly once.

## When NOT to use

- The clustering outputs have not yet been generated (e.g. 3-SkewC_TrimClustering.Rmd failed with 'No alpha value computation found' or the stringi/libicui18n.so.57 R package error)
- The cell identifiers in the .tsv files do not match the barcode conventions of Coverage_means_DF_Clust (e.g. barcode mismatch between barcode.tsv and the barcoded BAM file, which yields empty/incorrect outputs)

## Inputs

- TypicalCellsID.tsv (list of typical cells from SkewC_TrimClustering.Rmd)
- SkewedCellsID.tsv (list of skewed cells from SkewC_TrimClustering.Rmd)
- Coverage_means_DF_Clust clustering input data frame (derived from coverage.r gene body coverage results)

## Outputs

- Pass/fail verification that TypicalCellsID.tsv and SkewedCellsID.tsv partition all cells in Coverage_means_DF_Clust with no overlap

## How to apply

After running the R tclust trim clustering in 3-SkewC_TrimClustering.Rmd on Coverage_means_DF_Clust (derived from coverage.r gene body coverage results), load the two .tsv outputs TypicalCellsID.tsv and SkewedCellsID.tsv. Check that the union of cell identifiers in the two files equals the full set of cell identifiers (row names/cell barcodes) in Coverage_means_DF_Clust, and that the intersection is empty — no cell may appear in both lists. This confirms the alpha-trimmed clustering (alpha in [0..1], e.g. alpha = 0.099 for large 10X Genomics datasets) assigned every cell to exactly one of the typical or skewed groups. If the check fails, re-run the clustering or revisit the alpha value before using the partition to exclude skewed cells from downstream scRNA-seq analysis.

## Related tools

- **SkewC** (Produces the typical/skewed cell lists that this verification step checks for a disjoint, complete partition) — https://github.com/LSBDT/SkewC.git
- **tclust** (R trim clustering function whose alpha-trimmed assignment of Coverage_means_DF_Clust yields the two cell groups being verified)
- **R** (Language in which the clustering and output verification are implemented)

## Examples

```
Rscript -e 'typ <- read.delim("TypicalCellsID.tsv"); skew <- read.delim("SkewedCellsID.tsv"); load("coverage.r"); stopifnot(setequal(c(typ$x, skew$x), rownames(Coverage_means_DF_Clust)), length(intersect(typ$x, skew$x)) == 0)'
```

## Evaluation signals

- set(TypicalCellsID) ∪ set(SkewedCellsID) == set of all cell identifiers in Coverage_means_DF_Clust
- set(TypicalCellsID) ∩ set(SkewedCellsID) == ∅ (no cell assigned to both lists)
- Number of rows across both .tsv files equals the number of cells in Coverage_means_DF_Clust
- Clustering-result plot (PDF) from 3-SkewC_TrimClustering.Rmd renders two separated groups consistent with the two .tsv lists

## Limitations

- Verification cannot detect that the clustering itself was biologically correct — only that the partition is complete and disjoint
- A barcode mismatch between barcode.tsv and the barcoded BAM file can produce an empty PDF output, which would make the partition check vacuous
- Choice of alpha (constrained to [0..1]; e.g. alpha = 0.099 suggested for large 10X datasets) affects group membership, so a passing partition check does not guarantee an optimal trimming level

## Evidence

- [other] check that TypicalCellsID.tsv and SkewedCellsID.tsv together partition all cells present in Coverage_means_DF_Clust, with no cell assigned to both lists: [evidence span withheld — anchor status: not_found]
- [other] Write the two output text files (.tsv) containing the list of Typical cells and the list of Skewed cells (TypicalCellsID.tsv and SkewedCellsID.tsv): [evidence span withheld — anchor status: not_found]
- [other] Supply an alpha value constrained to [0 .. 1] (for large 10X Genomics datasets, e.g. start with alpha = 0.099 and evaluate the result): [evidence span withheld — anchor status: not_found]
- [abstract] The R Markdown SkewC_TrimClustering.Rmd performs the trim clustering implemented in R tclust function.: "The R Markdown SkewC_TrimClustering.Rmd performs the trim clustering implemented in R tclust function."
