---
name: epigenetic-mark-enrichment-analysis
description: Use when a scHiCDiff DCI results data frame is available (bin_1/bin_2
  pairs with pvalue.adj.FDR) and the goal is to characterize detected DCIs by epigenetic
  mark changes and GO enrichment, e.g. Astro vs Endo.
when_to_use_negative:
- When no scHiCDiff DCI detection has been performed yet — this skill operates on
  the DCI results data frame, not raw single-cell Hi-C contact matrices
- When comparing conditions other than the two-cell-type contrast for which differential
  epigenetic mark tracks exist (e.g. no Astro/Endo epigenetic data available), since
  the mark-change test requires matched tracks
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3940
tools:
- name: scHiCDiff
  role: Produces the ZINB DCI results data frame (bin pairs with pvalue.adj.FDR) that
    this skill annotates with epigenetic marks and GO terms
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Implementation environment for scHiCDiff and for the downstream overlap/gene-mapping/GO
    enrichment steps
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:28cbc2cf70d5c5aa5617140e7b6777d263003e24eeaf625b6572993b4a4c4ac7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3940
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# epigenetic-mark-enrichment-analysis

## Summary

Downstream functional annotation of differential chromatin interactions (DCIs) detected by scHiCDiff: overlapping significant DCI bins with differential epigenetic mark tracks and running Gene Ontology enrichment on DCI-associated genes, as demonstrated for the Astro vs Endo single-cell Hi-C comparison.

## When to use

Use when a scHiCDiff DCI results data frame is available (bin_1/bin_2 pairs with pvalue.adj.FDR) and the goal is to characterize detected DCIs by epigenetic mark changes and GO enrichment, e.g. Astro vs Endo.

## When NOT to use

- When no scHiCDiff DCI detection has been performed yet — this skill operates on the DCI results data frame, not raw single-cell Hi-C contact matrices
- When comparing conditions other than the two-cell-type contrast for which differential epigenetic mark tracks exist (e.g. no Astro/Endo epigenetic data available), since the mark-change test requires matched tracks

## Inputs

- ZINB DCI results data frame from scHiCDiff.ZINB (rows = bin pairs; columns = bin_1, bin_2, mu_1/mu_2, theta_1/theta_2, pi_1/pi_2, norm_total_mean_1/norm_total_mean_2, norm_foldChange, chi2LR1, pvalue, pvalue.adj.FDR, Remark)
- Differential epigenetic mark tracks between Astro and Endo cells
- Genome bin annotations for the scHi-C resolution used (e.g. 200 kb bins) for interval conversion and gene mapping

## Outputs

- functional_analysis_results.csv combining epigenetic-mark association at DCI bins and GO enrichment terms with Benjamini & Hochberg adjusted p-values
- List of significant GO terms (e.g. neurotransmitter/synaptic membrane terms) enriched at genes mapped from DCI bins

## How to apply

Load the DCI results data frame produced by scHiCDiff.ZINB (rows are bin pairs; columns include bin_1, bin_2, mu/theta/pi per condition, norm_foldChange, chi2LR1, pvalue, pvalue.adj.FDR) for the Astro vs Endo comparison. Select significant DCIs via the Benjamini & Hochberg adjusted p-value column (pvalue.adj.FDR) and convert bin_1/bin_2 interacting regions into genomic intervals. Overlap these DCI bins with differential epigenetic mark tracks and test whether bins in DCIs show significant epigenetic mark changes between Astro and Endo cells. Map DCI bins to genes and run GO enrichment focused on neurotransmitter/synaptic membrane terms, adjusting enrichment p-values with Benjamini & Hochberg FDR, then write combined results to functional_analysis_results.csv.

## Related tools

- **scHiCDiff** (Produces the ZINB DCI results data frame (bin pairs with pvalue.adj.FDR) that this skill annotates with epigenetic marks and GO terms) — https://github.com/wmalab/scHiCDiff
- **R** (Implementation environment for scHiCDiff and for the downstream overlap/gene-mapping/GO enrichment steps)

## Examples

```
count.table <- readRDS(paste("path/sampledata/sim 1 .rds")); result.zinb <- scHiCDiff.ZINB(as.matrix(count.table), factor(c(rep(1,50), rep(2,50))))
```

## Evaluation signals

- functional_analysis_results.csv exists and lists enrichment terms with Benjamini & Hochberg adjusted p-values consistent with the DCI results data frame convention (pvalue.adj.FDR column semantics)
- Epigenetic mark association at DCI bins is reported as significant for the Astro vs Endo comparison
- Neurotransmitter/synaptic membrane GO terms are reported as significantly enriched after Benjamini & Hochberg FDR adjustment
- DCI bins were derived from the significant subset of the scHiCDiff.ZINB output (filtered on pvalue.adj.FDR), not from all bin pairs
- Genomic intervals from bin_1/bin_2 convert back to the bin-pair coordinates present in the input DCI data frame

## Limitations

- The epigenetic-mark and GO findings are demonstrated for the Astro vs Endo scHi-C comparison; significance conclusions are specific to that contrast
- Downstream annotation inherits the limitations of the upstream DCI detection — the article notes scHiCDiff's ZINB option is what 'effectively detect[s] reliable and consistent single-cell DCIs', so results depend on using the appropriate detection model
- No version numbers are stated for scHiCDiff or R, so reproducibility requires recording the developmental version installed from the GitHub tarball

## Evidence

- [abstract] scHiCDiff, especially the zero-inflated negative binomial model option, can effectively detect reliable and consistent single-cell DCIs between two conditions: "Our results demonstrated that scHiCDiff, especially the zero-inflated negative binomial model option, can effectively detect reliable and consistent single-cell DCIs between two conditions"
- [readme] The detection function returns a data frame with bin pairs, statistics, and BH-adjusted p-values: "The detection function will return a data frame containing the differential chromatin interaction (DCI) analysis results, rows are bin pairs and columns lists the related statistics."
- [readme] pvalue.adj.FDR column is Benjamini & Hochberg adjusted, the convention this skill's enrichment output must match: "pvalue.adj.FDR       Adjusted P value of H0's pvalue using Benjamini & Hochberg's method."
- [abstract] scHiCDiff is implemented in R and freely available at GitHub: "scHiCDiff is implemented in R and freely available at GitHub"
