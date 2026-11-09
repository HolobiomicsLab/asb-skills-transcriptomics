---
name: mature-mrna-region-shifting
description: Use when you have RgnTX permutation test results (e.g., m6A sites vs
  stop codon regions via permTestTxIA_customPick) and need to verify that the association
  peak sits exactly at the original ROI positions rather than reflecting generic 3'UTR
  enrichment.
when_to_use_negative:
- Input regions already have a clear transcript/isoform association and you only need
  a single permutation p-value, not a positional-shift profile
- You are performing genome-based colocalization (e.g., GREAT, LOLA, GenomeRunner-style
  tests) rather than transcriptome/mRNA-space analysis
- The question is whether two region sets overlap at all, not whether the association
  is tied to exact ROI positions
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: RgnTX
  role: Provides shiftedZScoreTx, shiftExonTx, and plotShiftedZScoreTx to shift ROI
    over mature mRNA space and compute/plot shifted z-scores from permutation results
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: R
  role: Runtime environment / Bioconductor ecosystem in which RgnTX functions are
    executed
  repo: https://bioconductor.org/packages/RgnTX
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:a9b03db0ceca2f1a050351a714ec919f7dc9fe63102edf4f911b47a730edc576
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# mature-mrna-region-shifting

## Summary

Applies RgnTX's shifted z-score machinery (shiftedZScoreTx, shiftExonTx) to slide regions of interest over mature mRNA space (exons) and profile how a permutation-test colocalization signal changes when ROI positions are shifted, distinguishing true positional association from background enrichment.

## When to use

Use when you have RgnTX permutation test results (e.g., m6A sites vs stop codon regions via permTestTxIA_customPick) and need to verify that the association peak sits exactly at the original ROI positions rather than reflecting generic 3'UTR enrichment.

## When NOT to use

- Input regions already have a clear transcript/isoform association and you only need a single permutation p-value, not a positional-shift profile
- You are performing genome-based colocalization (e.g., GREAT, LOLA, GenomeRunner-style tests) rather than transcriptome/mRNA-space analysis
- The question is whether two region sets overlap at all, not whether the association is tied to exact ROI positions

## Inputs

- RgnTX permutation test result object (from permTestTx, permTestTx_customPick, or permTestTxIA_customPick)
- TxDb transcript annotation (e.g., TxDb.Hsapiens.UCSC.hg19.knownGene) providing exon/transcript structures
- Region set RS1 (e.g., m6A sites from miCLIP-seq, possibly isoform-ambiguous genomic positions)
- Shift parameters: window, step, and type (e.g., window=500, step=50, type='mature')

## Outputs

- Shifted z-score profile over shifted region windows (data frame with shift offsets and z-scores)
- Shifted z-score plot (e.g., shifted_zscore_plot.png) showing enrichment curve around original ROI positions

## How to apply

First run an RgnTX permutation test (e.g., permTestTxIA_customPick with customPick_function = getStopCodon for isoform-ambiguous features like m6A miCLIP-seq sites) to obtain permutation results with p-value and z-score. Then call shiftedZScoreTx on those results with window=500, step=50, and type='mature' so the ROI is shifted over mature mRNA space, spanning multiple exon intervals via shiftExonTx; positive width moves start positions right (3'), negative moves left (5'). Plot the resulting z-score profile with plotShiftedZScoreTx and inspect whether a clear central peak occurs at the original positions. Interpretation: a central peak supports position-specific association (e.g., m6A–stop codon); a peak deviated to the right (3') indicates enrichment biased toward the 3'UTR rather than the exact stop codon position.

## Related tools

- **RgnTX** (Provides shiftedZScoreTx, shiftExonTx, and plotShiftedZScoreTx to shift ROI over mature mRNA space and compute/plot shifted z-scores from permutation results) — https://github.com/yue-wang-biomath/RgnTX
- **R** (Runtime environment / Bioconductor ecosystem in which RgnTX functions are executed) — https://bioconductor.org/packages/RgnTX

## Examples

```
shiftedZScoreTx_results <- shiftedZScoreTx(permTestTx_results, window = 500, step = 50, type = 'mature'); plotShiftedZScoreTx(shiftedZScoreTx_results)
```

## Evaluation signals

- The shifted z-score profile shows a clear central peak at the original ROI positions (shift offset 0) when the association is position-specific
- For the m6A–stop codon case study, the peak is deviated to the right (3' direction), consistent with m6A enrichment more on the 3'UTR than on the CDS
- z-scores at offsets far from the original position fall toward background levels compared with the central peak
- Shifted regions produced by shiftExonTx span multiple exon intervals and honor width sign conventions (positive = 3' shift, negative = 5' shift) on the mature mRNA
- The plot is saved (e.g., shifted_zscore_plot.png) and renders the enrichment curve around the original region positions

## Limitations

- Isoform ambiguity: m6A sites lack information about the specific isoform they originate from, so counts are averaged over all K alignable isoforms (as in permTestTxIA_customPick)
- The central-peak interpretation can be confounded by broad region-type enrichment (e.g., general 3'UTR preference), which manifests as a right-deviated peak rather than a sharp positional peak
- No version number or R dependency versions are stated for RgnTX, complicating reproducibility pinning

## Evidence

- [abstract] abstract workflow: "easy to be utilized by users for visualizing permutation results, calculating shifted z-scores and conducting multiple hypothesis testing under Benjamini-Hochberg correction."
- [readme] readme shiftedZScoreTx update: [evidence span withheld — anchor status: external_doc]
- [readme] readme shiftExonTx semantics: [evidence span withheld — anchor status: external_doc]
- [other] source card workflow: [evidence span withheld — anchor status: not_found]
- [results] case study finding: [evidence span withheld — anchor status: not_found]
