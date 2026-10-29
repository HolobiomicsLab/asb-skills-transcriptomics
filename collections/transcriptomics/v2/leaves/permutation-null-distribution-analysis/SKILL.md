---
name: permutation-null-distribution-analysis
description: Use when testing whether two transcriptome region sets (e.g., m6A RNA
  methylation sites and stop codon regions) colocalize more than expected by chance,
  especially when region-to-isoform association is ambiguous.
when_to_use_negative:
- When the analysis target is the genome rather than the transcriptome (existing genome-based
  tools like LOLA or GenomeRunner suffice and RgnTX's transcriptome null model is
  unnecessary)
- When only two static feature tables need interval overlap counting without a statistical
  significance assessment
- When regions have unambiguous isoform association AND a simple genome-based permutation
  is acceptable — RgnTX's isoform-aware machinery adds cost without benefit
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0099
tools:
- name: RgnTX
  role: R/Bioconductor package providing permTestTx, permTestTx_customPick, permTestTxIA_customPick,
    randomizeTx, and shiftedZScoreTx for transcriptome permutation colocalization
    tests
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: R
  role: Runtime environment in which RgnTX permutation tests and downstream shifted
    z-score analyses are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:09f13246d0f0494ac01d782936efb25bf394a6387d184e0a080be4c746f1ac6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# permutation-null-distribution-analysis

## Summary

Generates empirical null distributions of overlaps between transcriptome region sets using RgnTX permutation tests, enabling colocalization assessment of elements (e.g., m6A sites vs stop codons) that accounts for isoform heterogeneity and isoform ambiguity.

## When to use

Use when testing whether two transcriptome region sets (e.g., m6A RNA methylation sites and stop codon regions) colocalize more than expected by chance, especially when region-to-isoform association is ambiguous.

## When NOT to use

- When the analysis target is the genome rather than the transcriptome (existing genome-based tools like LOLA or GenomeRunner suffice and RgnTX's transcriptome null model is unnecessary)
- When only two static feature tables need interval overlap counting without a statistical significance assessment
- When regions have unambiguous isoform association AND a simple genome-based permutation is acceptable — RgnTX's isoform-aware machinery adds cost without benefit

## Inputs

- Region set RS1 (GRanges of transcriptome features, e.g., m6A sites from miCLIP-seq in .rds format via system.file(package='RgnTX','extdata','m6A_sites_data.rds'))
- Region set RS2 or a customPick_function (e.g., getStopCodon) defining regions of interest
- TxDb transcriptome annotation object (e.g., TxDb.Hsapiens.UCSC.hg19.knownGene)
- Permutation parameters: ntimes, type ('mature'/'full'/'fiveUTR'/'CDS'/'threeUTR'), random_num, random_length

## Outputs

- permTestTx results object containing observed_overlap, random_region_list, random_overlap_list (null distribution), p-value and z-score
- Randomized region sets (GRangesList) from randomizeTx used as the transcriptome background
- BH-adjusted p-values across multiple tested region groups
- Shifted z-score profile (via shiftedZScoreTx, e.g., window=500, step=50) plottable with plotShiftedZScoreTx

## How to apply

Run RgnTX in R with a TxDb transcriptome annotation (e.g., TxDb.Hsapiens.UCSC.hg19.knownGene). For region sets with known isoform association, use permTestTx or permTestTx_customPick with a customPick_function such as getStopCodon; for features known only by genomic positions (e.g., miCLIP-seq m6A sites with isoform ambiguity), use permTestTxIA_customPick with ev_function_1 = overlapCountsTxIA and ev_function_2 = overlapCountsTx. Specify ntimes (e.g., 50–100 permutations), and type ('mature', 'full', 'fiveUTR', 'CDS', 'threeUTR') to control the background against which regions are randomized. RgnTX returns observed_overlap, random_overlap_list, p-value and z-score; apply Benjamini-Hochberg correction when testing many region groups (the article divided m6A sites into groups of 100 non-repeated regions and ran permutation tests per group). Optionally follow with shiftedZScoreTx (window=500, step=50, type='mature') to profile z-scores over shifted region windows and confirm the signal peaks at the original position.

## Related tools

- **RgnTX** (R/Bioconductor package providing permTestTx, permTestTx_customPick, permTestTxIA_customPick, randomizeTx, and shiftedZScoreTx for transcriptome permutation colocalization tests) — https://github.com/yue-wang-biomath/RgnTX
- **R** (Runtime environment in which RgnTX permutation tests and downstream shifted z-score analyses are executed)

## Examples

```
permTestTx_results <- permTestTxIA_customPick(RS1 = m6A_sites_data[1:500], txdb = txdb, customPick_function = getStopCodon, ntimes = 50, ev_function_1 = overlapCountsTxIA, ev_function_2 = overlapCountsTx)
```

## Evaluation signals

- The results object exposes all expected components: observed_overlap, random_region_list, random_overlap_list, pvalue and zscore
- With a true m6A–stop codon association, the shifted z-score profile (shiftedZScoreTx with window=500, step=50, type='mature') shows a clear peak at the original position deviated toward the 3' direction
- Null distributions generated on transcriptome backgrounds should be significantly tighter/different from genome-based backgrounds (verified in the article by two-sample t-test over 1000 random region-set pairs of 500 equal-length regions)
- BH-corrected p-values across region groups (e.g., 100 non-repeated m6A regions per group) should reduce false positives versus raw p-values
- Randomized regions from randomizeTx must respect the specified type (e.g., 'mature') and region widths (random_length)

## Limitations

- No version number stated for RgnTX and no R/dependency versions specified, so reproducibility requires pinning environments manually
- Genomic features with isoform ambiguity (e.g., POSTAR3 RBP target sites lacking isoform information) can only be tested via the IA (isoform-ambiguous) mode, which averages overlaps over the K compatible isoforms per region
- Null distributions of random overlaps differ significantly between genome and transcriptome backgrounds (DNA, exonic DNA, pre-mRNA, mRNA), so results are not interchangeable with genome-based permutation methods

## Evidence

- [readme] RgnTX supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations.: "Importantly, RgnTX supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations."
- [results] m6A sites divided into groups of 100 regions with per-group permutation tests: "We randomly shuffled all the m6A sites and divided them into independent groups. Each independent group contains 100 regions"
- [results] BH correction reduces false positives: "RgnTX provided Benjamini-Hochberg correction to adjust the p-values, reducing the number of false positives"
- [results] null distributions differ across genome/transcriptome backgrounds: "Although all the backgrounds were restricted to only 300 genes, the overlapping counts in each background were significantly different"
- [abstract] supports visualization, shifted z-scores, and BH correction: "easy to be utilized by users for visualizing permutation results, calculating shifted z-scores and conducting multiple hypothesis testing under Benjamini-Hochberg correction."
