---
name: shifted-z-score-calculation
description: Use when permutation test results show a significant colocalization between
  region sets (e.g., m6A methylation sites and stop codon regions) and you must verify
  the association peaks exactly at the original region positions rather than being
  a diffuse positional effect.
when_to_use_negative:
- The region sets have clear isoform association and a standard transcriptome-space
  permTestTx without positional-shift diagnostics suffices
- The input is genome-coordinate-only colocalization analysis (e.g., BEDTools, GREAT,
  LOLA) where transcriptome/exon-level shifting is not applicable
- The permutation results are not significant and no positional refinement question
  exists
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0099
tools:
- name: RgnTX
  role: Provides shiftedZScoreTx and plotShiftedZScoreTx to compute and visualize
    shifted z-scores over mRNA (exon) space, plus permTestTxIA_customPick for the
    upstream permutation test on isoform-ambiguous m6A sites vs stop codons
  repo: https://bioconductor.org/packages/RgnTX
- name: R
  role: Execution environment for running the RgnTX workflow and saving the plot
  repo: https://github.com/yue-wang-biomath/RgnTX
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:4d3a1e177706ae108cf0e74d381f4e63d585956e95d7ecfa4cba87749e0cb711
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RgnTX
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# shifted-z-score-calculation

## Summary

Calculates shifted z-scores over transcriptome windows with RgnTX's shiftedZScoreTx to test whether an observed colocalization (e.g., m6A sites vs stop codon regions) is tied to the exact positions of the regions of interest on mature mRNA, then visualizes the profile with plotShiftedZScoreTx.

## When to use

Use when permutation test results show a significant colocalization between region sets (e.g., m6A methylation sites and stop codon regions) and you must verify the association peaks exactly at the original region positions rather than being a diffuse positional effect.

## When NOT to use

- The region sets have clear isoform association and a standard transcriptome-space permTestTx without positional-shift diagnostics suffices
- The input is genome-coordinate-only colocalization analysis (e.g., BEDTools, GREAT, LOLA) where transcriptome/exon-level shifting is not applicable
- The permutation results are not significant and no positional refinement question exists

## Inputs

- RgnTX permutation test results object (e.g., from permTestTxIA_customPick) containing observed/random overlap counts, p-value and z-score
- m6A RNA methylation sites with isoform ambiguity (e.g., miCLIP-seq sites loaded from extdata/m6A_sites_data.rds)
- TxDb transcript annotation (e.g., TxDb.Hsapiens.UCSC.hg19.knownGene) providing exons/stop codon regions

## Outputs

- Shifted z-score values over shifted region windows (window=500, step=50, type='mature')
- Shifted z-score profile plot (e.g., shifted_zscore_plot.png) showing enrichment curve around the original region positions

## How to apply

First run a permutation test in RgnTX (e.g., permTestTxIA_customPick with getStopCodon as the customPick_function and overlapCountsTxIA/overlapCountsTx as evaluation functions) to obtain permutation results. Then call shiftedZScoreTx on those results with window=500, step=50, and type='mature' to recompute overlap-based z-scores as the regions of interest are shifted over mRNA (exonic) space across the window. Plot the resulting z-score profile with plotShiftedZScoreTx and inspect whether the curve shows a clear central peak at the original position; for m6A–stop codon data, a peak deviated toward the 3' direction indicates m6A enrichment more on the 3'UTR than on the CDS. Save the diagnostic (e.g., shifted_zscore_plot.png) and interpret a sharp, position-specific peak as evidence that the association is linked to the exact stop codon positions.

## Related tools

- **RgnTX** (Provides shiftedZScoreTx and plotShiftedZScoreTx to compute and visualize shifted z-scores over mRNA (exon) space, plus permTestTxIA_customPick for the upstream permutation test on isoform-ambiguous m6A sites vs stop codons) — https://bioconductor.org/packages/RgnTX
- **R** (Execution environment for running the RgnTX workflow and saving the plot) — https://github.com/yue-wang-biomath/RgnTX

## Examples

```
library(RgnTX); shifted_results <- shiftedZScoreTx(permTestTx_results, window = 500, step = 50, type = 'mature'); plotShiftedZScoreTx(shifted_results)
```

## Evaluation signals

- The shifted z-score profile shows a clear central peak at the original (unshifted) region position
- For m6A–stop codon data, the peak is deviated toward the 3' direction, consistent with m6A enrichment more on the 3'UTR than on the CDS
- The workflow parameters match the protocol: window=500, step=50, type='mature'
- The plot is saved (shifted_zscore_plot.png) and the upstream permTestTxIA_customPick results used ev_function_1=overlapCountsTxIA and ev_function_2=overlapCountsTx with customPick_function=getStopCodon
- Z-scores away from the center of the profile fall toward the null expectation rather than staying elevated across the whole window

## Limitations

- Shifting is performed over mRNA (exon) space only; shiftedZScoreTx supports shifting ROI over mRNA space (exons), so the interpretation is restricted to mature transcripts
- Isoform ambiguity of input features (m6A sites lacking specific isoform association) must be handled by the IA permutation functions; results depend on how many isoforms a region aligns to
- No version number or dependency versions are stated for RgnTX in the article, so reproducibility of exact behavior across package versions is not documented

## Evidence

- [abstract] easy to be utilized by users for visualizing permutation results, calculating shifted z-scores and conducting multiple hypothesis testing under Benjamini-Hochberg correction.: "easy to be utilized by users for visualizing permutation results, calculating shifted z-scores and conducting multiple hypothesis testing under Benjamini-Hochberg correction."
- [results] The shifted z-score plot for the m6A–stop codon association shows a clear central peak: [evidence span withheld — anchor status: not_found]
- [readme] Function `shiftedZScoreTx` is updated supprting the shifting regions of interest (ROI) over mRNA space (exons).: [evidence span withheld — anchor status: external_doc]
- [results] m6A RNA methylation sites (with isoform ambiguity) from miCLIP-seq [36] and stop codon regions (without isoform ambiguity) derived from UCSC transcriptome annotation: "m6A RNA methylation sites (with isoform ambiguity) from miCLIP-seq [36] and stop codon regions (without isoform ambiguity) derived from UCSC transcriptome annotation"
