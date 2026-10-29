---
name: permutation-result-visualization
description: Use when a permutation test from RgnTX (permTestTx, permTestTx_customPick,
  or permTestTxIA_customPick) has produced a null overlap distribution and you need
  to plot observed vs. random overlaps, p-values, or z-scores for a transcriptome
  colocalization result.
when_to_use_negative:
- When the colocalization question is genome-based only and isoform ambiguity/transcriptome
  shuffling is irrelevant — genome-based tools like regioneR or BEDTools are the appropriate
  comparison points.
- When no permutation test has been run yet — visualization requires an existing results
  object with a random overlap null distribution.
- When regions have clear isoform association and you need IA-mode accounting — use
  overlapCountsTx rather than overlapCountsTxIA as the evaluation function context.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3512
tools:
- name: RgnTX
  role: Provides the permutation test functions whose results are visualized, plus
    support for visualizing permutation results and calculating shifted z-scores
  repo: https://bioconductor.org/packages/RgnTX
- name: R
  role: Execution environment for running RgnTX permutation tests and producing the
    summary-statistics plots
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: UCSC hg19 transcriptome annotation used to extract 5'UTR and CDS backgrounds
    via customPick functions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:07900373fe3094c8621e3efb253a7d34e3fbe14cfd9a0ba97719c55f5c735309
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# permutation-result-visualization

## Summary

Visualize RgnTX permutation test results by plotting the observed colocalization overlap of a region set (e.g., m7G sites against 5'UTR or CDS) relative to the transcriptome-wide shuffling null distribution, supporting summary-statistics plots like Fig. S2.

## When to use

Use when a permutation test from RgnTX (permTestTx, permTestTx_customPick, or permTestTxIA_customPick) has produced a null overlap distribution and you need to plot observed vs. random overlaps, p-values, or z-scores for a transcriptome colocalization result.

## When NOT to use

- When the colocalization question is genome-based only and isoform ambiguity/transcriptome shuffling is irrelevant — genome-based tools like regioneR or BEDTools are the appropriate comparison points.
- When no permutation test has been run yet — visualization requires an existing results object with a random overlap null distribution.
- When regions have clear isoform association and you need IA-mode accounting — use overlapCountsTx rather than overlapCountsTxIA as the evaluation function context.

## Inputs

- permTestTx_results object from RgnTX (containing ROI, observed_overlap, random_overlap_list, pvalue, zscore)
- Region set RS1 (e.g., m7G sites from the RgnTX vignette example dataset, GRanges)
- TxDb.Hsapiens.UCSC.hg19.knownGene transcript annotation
- customPick_function extracting 5'UTR or CDS regions from transcripts

## Outputs

- Summary-statistics plot of observed overlap vs. null (random) overlap distribution, e.g., fig_S2_summary_statistics.png
- Reported permutation p-value and z-score for the tested association

## How to apply

Run the appropriate RgnTX permutation function first — for features lacking isoform association (e.g., m7G or m6A sites) use permTestTxIA_customPick with ev_function_1 = overlapCountsTx, otherwise use permTestTx or permTestTx_customPick. The returned permTestTx_results object carries the ROI, random_region_list, observed_overlap, random_overlap_list, pvalue, and zscore needed for plotting. Define a customPick_function (e.g., getCDS for CDS regions or a 5'UTR extractor) against TxDb.Hsapiens.UCSC.hg19.knownGene so the background is the transcriptome region of interest. Then plot the observed overlap against the random overlap list as a summary-statistics figure (reproducing Fig. S2 style for m7G–5'UTR and m7G–CDS associations) and, optionally, use the package's shifted z-score and visualization support; for multiple tests apply the package's Benjamini-Hochberg correction before displaying adjusted p-values.

## Related tools

- **RgnTX** (Provides the permutation test functions whose results are visualized, plus support for visualizing permutation results and calculating shifted z-scores) — https://bioconductor.org/packages/RgnTX
- **R** (Execution environment for running RgnTX permutation tests and producing the summary-statistics plots)
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (UCSC hg19 transcriptome annotation used to extract 5'UTR and CDS backgrounds via customPick functions)

## Examples

```
permTestTx_results <- permTestTxIA_customPick(RS1 = m7G_sites_data[1:500], txdb = TxDb.Hsapiens.UCSC.hg19.knownGene, customPick_function = getFiveUTR, ntimes = 50, ev_function_1 = overlapCountsTxIA, ev_function_2 = overlapCountsTx)
```

## Evaluation signals

- The results object contains all plotting inputs: ROI, random_region_list, observed_overlap, random_overlap_list, pvalue, and zscore.
- The plotted null distribution contains ntimes random overlap values (e.g., ntimes = 50 in the README example; 100 permutations used in the paper's m6A group tests).
- The observed overlap marker lies within a plausible range of the null distribution; z-score magnitude should be consistent with the visual separation of the observed value from the random overlaps.
- The plot reproduces the Fig. S2 pattern: association of m7G sites with 5'UTR and with CDS assessed against the transcriptome-wide shuffling null.
- If multiple associations are plotted, p-values are adjusted with Benjamini-Hochberg correction as supported by RgnTX.

## Limitations

- The number of permutations (ntimes) is user-set and small values (e.g., 50) give a coarse null distribution for visualization and p-value stability.
- RgnTX has no stated version number or changelog in the article, so exact visualization function behavior may vary across Bioconductor releases.
- For features without isoform association, results depend on the IA-mode weighting (overlap divided by the K isoforms a region aligns to), which affects the plotted observed and random overlaps.

## Evidence

- [abstract] easy to be utilized by users for visualizing permutation results, calculating shifted z-scores and conducting multiple hypothesis testing under Benjamini-Hochberg correction.: "easy to be utilized by users for visualizing permutation results, calculating shifted z-scores and conducting multiple hypothesis testing under Benjamini-Hochberg correction."
- [readme] RETURN ROI, ramdom_region_list observed_overlap, random_overlap_list, pvalue, zscore: [evidence span withheld — anchor status: external_doc]
- [readme] The function permTestTx_customPick performs permutation test to evaluate association between features (RS) and transcriptome regions of interest (ROI).: [evidence span withheld — anchor status: not_found]
- [supplementary] The package documentation (Fig. S2) provides an example of the association between m7G sites and the 5'UTR, as well as the association between m7G sites and the CDS, demonstrating the visualization workflow to reproduce.: [evidence span withheld — anchor status: not_found]
