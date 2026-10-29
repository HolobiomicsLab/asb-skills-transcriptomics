---
name: m6a-colocalization-visualization
description: Use when you have RgnTX permutation-test results linking m6A sites to
  stop codon regions and need to determine whether enrichment is localized at the
  original region positions or is an artifact, by examining the shifted z-score profile.
when_to_use_negative:
- The regions of interest have a known, unambiguous isoform association and a simple
  permTestTx analysis suffices — isoform-ambiguity handling (overlapCountsTxIA) is
  unnecessary.
- The analysis goal is genome-based colocalization without transcriptome annotation
  context, where genome-based tools (GREAT, LOLA, GenomeRunner, regioneR) are the
  article's stated comparison baselines.
- You only need a global p-value for association; the shifted z-score profile is specifically
  for assessing position-specificity of the signal.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: RgnTX
  role: Provides shiftedZScoreTx, shiftExonTx and plotShiftedZScoreTx for sliding
    ROI over mRNA space and visualizing the shifted z-score profile; also provides
    permTestTxIA_customPick for the underlying isoform-ambiguous permutation test.
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: R
  role: Runtime environment; RgnTX is an R/Bioconductor package installed via BiocManager
    or devtools.
  repo: https://bioconductor.org/packages/RgnTX
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:6ad9d74301dbe463ce35bf28b50107f16f0467adee77ff070ec8a807e4c156f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# m6a-colocalization-visualization

## Summary

Visualizes and statistically validates the spatial association between m6A RNA methylation sites (with isoform ambiguity) and stop codon regions by computing shifted z-scores over mature mRNA space with RgnTX's shiftedZScoreTx and rendering the enrichment profile with plotShiftedZScoreTx.

## When to use

Use when you have RgnTX permutation-test results linking m6A sites to stop codon regions and need to determine whether enrichment is localized at the original region positions or is an artifact, by examining the shifted z-score profile.

## When NOT to use

- The regions of interest have a known, unambiguous isoform association and a simple permTestTx analysis suffices — isoform-ambiguity handling (overlapCountsTxIA) is unnecessary.
- The analysis goal is genome-based colocalization without transcriptome annotation context, where genome-based tools (GREAT, LOLA, GenomeRunner, regioneR) are the article's stated comparison baselines.
- You only need a global p-value for association; the shifted z-score profile is specifically for assessing position-specificity of the signal.

## Inputs

- RgnTX permutation test result object (e.g., from permTestTxIA_customPick with customPick_function = getStopCodon) containing observed_overlap, random_overlap_list, pvalue and zscore
- m6A RNA methylation sites from miCLIP-seq (isoform-ambiguous genomic positions, e.g., m6A_sites_data.rds grouped into sets of 100 non-repeated regions)
- Transcriptome annotation (TxDb.Hsapiens.UCSC.hg19.knownGene) for stop codon region definition

## Outputs

- Shifted z-score table/profile over sliding windows (window=500, step=50, type='mature')
- shifted_zscore_plot.png: enrichment curve showing the z-score peak relative to the original region positions

## How to apply

Load the m6A–stop codon permutation test results produced by RgnTX (typically from permTestTxIA_customPick with customPick_function = getStopCodon on TxDb.Hsapiens.UCSC.hg19.knownGene, using overlapCountsTxIA since m6A sites lack clear isoform association). Call shiftedZScoreTx on these results with window=500, step=50, and type='mature' to slide the regions of interest over mRNA space (exons) and recompute z-scores at each shifted window. Render the resulting profile with plotShiftedZScoreTx and save as shifted_zscore_plot.png. A clear central peak at the original position, shifted toward the 3' direction (right), indicates the m6A–stop codon association is tied to the exact stop codon positions and that m6A is enriched more on the 3'UTR than the CDS.

## Related tools

- **RgnTX** (Provides shiftedZScoreTx, shiftExonTx and plotShiftedZScoreTx for sliding ROI over mRNA space and visualizing the shifted z-score profile; also provides permTestTxIA_customPick for the underlying isoform-ambiguous permutation test.) — https://github.com/yue-wang-biomath/RgnTX
- **R** (Runtime environment; RgnTX is an R/Bioconductor package installed via BiocManager or devtools.) — https://bioconductor.org/packages/RgnTX

## Examples

```
shifted_zscores <- shiftedZScoreTx(permTestTx_results, window = 500, step = 50, type = 'mature'); plotShiftedZScoreTx(shifted_zscores)
```

## Evaluation signals

- The plotted shifted z-score profile exhibits a single clear peak centered at the original (zero-shift) window position.
- The peak is deviated to the right (3' direction), consistent with m6A enrichment on the 3'UTR relative to the CDS near stop codons.
- The underlying permTestTxIA_customPick run used overlapCountsTxIA for the isoform-ambiguous feature set and returned significant zscore/pvalue before shifting.
- The shift parameters match the protocol: window=500, step=50, type='mature' (regions shifted over mature mRNA exons).
- The saved plot file shifted_zscore_plot.png is produced and matches the on-screen profile from plotShiftedZScoreTx.

## Limitations

- Transcriptome elements such as POSTAR3-collected m6A sites lack information about the specific isoform they originate from; the shifted analysis must handle this ambiguity rather than assume transcript association.
- Null distributions of random overlaps differ significantly between genome-based and transcriptome backgrounds (DNA, exonic DNA, pre-mRNA, mRNA), so shifted z-scores computed on a transcriptome background are not comparable to genome-based shifting approaches.
- No version number, R version, or dependency versions are stated for RgnTX, and no explicit code/data availability statement beyond the GitHub/Bioconductor links, so reproducibility of exact numeric profiles may vary across environments.
- Permutation count (ntimes) and grouping of m6A sites into independent groups of 100 regions affect z-score stability; too few permutations yield noisy shifted profiles.

## Evidence

- [abstract] easy to be utilized by users for visualizing permutation results, calculating shifted z-scores and conducting multiple hypothesis testing under Benjamini-Hochberg correction.: "easy to be utilized by users for visualizing permutation results, calculating shifted z-scores and conducting multiple hypothesis testing under Benjamini-Hochberg correction."
- [readme] Function `shiftedZScoreTx` is updated supprting the shifting regions of interest (ROI) over mRNA space (exons).: [evidence span withheld — anchor status: external_doc]
- [readme] The function `shiftExonTx` is able to shift regions over mRNA space, i.e., spanning multiple intervals (exons).: [evidence span withheld — anchor status: external_doc]
- [results] We randomly shuffled all the m6A sites and divided them into independent groups. Each independent group contains 100 regions... association between m6A sites and the stop codon regions were evaluated: "We randomly shuffled all the m6A sites and divided them into independent groups. Each independent group contains 100 regions"
- [results] m6A RNA methylation sites (with isoform ambiguity) from miCLIP-seq [36] and stop codon regions (without isoform ambiguity) derived from UCSC transcriptome annotation: "m6A RNA methylation sites (with isoform ambiguity) from miCLIP-seq [36] and stop codon regions (without isoform ambiguity) derived from UCSC transcriptome annotation"
