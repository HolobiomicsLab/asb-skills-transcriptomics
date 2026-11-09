---
name: genomic-interval-overlap-analysis
description: 'Use when you already have a scHiCDiff DCI results data frame (bin pairs
  with pvalue.adj.FDR) and need downstream functional annotation: epigenetic mark
  association at DCI bins and GO enrichment of DCI-linked genes.'
when_to_use_negative:
- Input is raw scHi-C contact matrices needing normalization and DCI detection — run
  the scHiCDiff.norm/scHiCDiff.ZINB pipeline first; this skill only annotates already-detected
  DCIs
- Comparison is bulk Hi-C or involves more than two conditions — the scHiCDiff detection
  functions and downstream DCI annotation are defined for a two-condition single-cell
  comparison via a 'group' factor
- Goal is to recompute interaction-level statistics (chi2LR1, norm_foldChange) rather
  than to functionally/epigenetically interpret existing DCI bins
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: scHiCDiff
  role: Produces the DCI results data frame (scHiCDiff.ZINB output with bin_1/bin_2,
    pvalue.adj.FDR) whose significant bin pairs are converted to genomic intervals
    for overlap and enrichment analysis
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Implementation environment in which scHiCDiff is installed and run, and in
    which the interval-overlap/GO-enrichment downstream workflow is executed
  repo: https://github.com/wmalab/scHiCDiff
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:4c16293f1a89408215fce1446b043735383c789e2c56ef9e22086cdf14708594
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# genomic-interval-overlap-analysis

## Summary

Annotates differential chromatin interactions (DCIs) detected by scHiCDiff by converting interacting bin pairs (bin_1/bin_2) into genomic intervals, overlapping them with differential epigenetic mark tracks, and mapping them to genes for Gene Ontology enrichment — as demonstrated for Astro vs Endo scHi-C comparisons.

## When to use

Use when you already have a scHiCDiff DCI results data frame (bin pairs with pvalue.adj.FDR) and need downstream functional annotation: epigenetic mark association at DCI bins and GO enrichment of DCI-linked genes.

## When NOT to use

- Input is raw scHi-C contact matrices needing normalization and DCI detection — run the scHiCDiff.norm/scHiCDiff.ZINB pipeline first; this skill only annotates already-detected DCIs
- Comparison is bulk Hi-C or involves more than two conditions — the scHiCDiff detection functions and downstream DCI annotation are defined for a two-condition single-cell comparison via a 'group' factor
- Goal is to recompute interaction-level statistics (chi2LR1, norm_foldChange) rather than to functionally/epigenetically interpret existing DCI bins

## Inputs

- scHiCDiff.ZINB DCI results data frame (rows = bin pairs; columns = bin_1, bin_2, mu_1/mu_2, theta_1/theta_2, pi_1/pi_2, norm_total_mean_1/norm_total_mean_2, norm_foldChange, chi2LR1, pvalue, pvalue.adj.FDR)
- Differential epigenetic mark tracks between the two compared cell types (e.g., Astro vs Endo)
- Gene annotation for mapping DCI bins to genes

## Outputs

- functional_analysis_results.csv combining epigenetic-mark association at DCI bins and GO enrichment terms with Benjamini & Hochberg adjusted p-values
- Genomic intervals derived from significant bin_1/bin_2 DCI regions
- GO enrichment table (e.g., neurotransmitter/synaptic membrane terms) with BH-adjusted p-values

## How to apply

Start from the ZINB DCI results data frame produced by scHiCDiff.ZINB, whose rows are bin pairs and whose columns include bin_1, bin_2, mu/theta/pi per condition, norm_foldChange, chi2LR1, pvalue and pvalue.adj.FDR. Select significant DCIs using the Benjamini & Hochberg adjusted p-value column (pvalue.adj.FDR), then convert the bin_1/bin_2 interacting regions into genomic intervals. Overlap these DCI intervals with differential epigenetic mark tracks and test whether bins in DCIs show significant epigenetic mark changes between the two cell types (e.g., Astro vs Endo). Map DCI bins to genes and run Gene Ontology enrichment — for the Astro vs Endo comparison, focused on neurotransmitter/synaptic membrane terms — adjusting enrichment p-values with Benjamini & Hochberg FDR. Write the combined epigenetic-mark association and GO enrichment results to functional_analysis_results.csv.

## Related tools

- **scHiCDiff** (Produces the DCI results data frame (scHiCDiff.ZINB output with bin_1/bin_2, pvalue.adj.FDR) whose significant bin pairs are converted to genomic intervals for overlap and enrichment analysis) — https://github.com/wmalab/scHiCDiff
- **R** (Implementation environment in which scHiCDiff is installed and run, and in which the interval-overlap/GO-enrichment downstream workflow is executed) — https://github.com/wmalab/scHiCDiff

## Examples

```
count.table <- readRDS(paste("path/sampledata/sim 1 .rds")); result.zinb <- scHiCDiff.ZINB(count.table,group)
```

## Evaluation signals

- The output table (functional_analysis_results.csv) lists GO enrichment terms with Benjamini & Hochberg adjusted p-values, consistent with the pvalue.adj.FDR convention of the scHiCDiff DCI data frame
- Only bins passing the BH-adjusted p-value significance threshold are converted to genomic intervals and annotated
- For the Astro vs Endo comparison, both the epigenetic mark association at DCI bins and the neurotransmitter/synaptic GO terms are reported as significant
- Genomic intervals are traceable back to the bin_1/bin_2 columns of the input DCI results data frame
- Each significant DCI interval maps to at least one gene, and enrichment is computed over the mapped gene set

## Limitations

- Downstream annotation depends on the upstream normalization (effective length, GC content, mappability) and detection steps; errors in normalization propagate into the DCI bins being annotated
- Only DCIs significant by Benjamini & Hochberg adjusted p-value should be converted to intervals; non-significant bin pairs will dilute the enrichment signal
- The scHiCDiff documentation provides no version numbers for scHiCDiff or R, so reproducibility of the downstream workflow requires pinning environments separately
- The zero-inflated negative binomial option is specifically designed for single-cell interaction counts with excessive zeros; bulk-like data may yield bin pairs less suited to this annotation workflow

## Evidence

- [readme] DCI output schema with bin pairs: "The detection function will return a data frame containing the differential chromatin interaction (DCI) analysis results, rows are bin pairs and columns lists the related statistics."
- [readme] bin_1/bin_2 define interacting regions: "bin_1,bin_2          The interacting region of the bin pair."
- [readme] BH-adjusted p-value column used for significance selection: "pvalue.adj.FDR       Adjusted P value of H0's pvalue using Benjamini & Hochberg's method."
- [intro] ZINB rationale for single-cell DCI inputs: "zero-inflated Negative Binomial(ZINB) regression models is specially designated for the interaction comparison at single-cell level"
- [readme] Group factor defines the two compared conditions: "group            A vector of factor which mentions the two condition to be compared, corresponding to the columns in the count table."
