---
name: weighted-overlap-colocalization-measure
description: Use when testing association between transcriptome region sets where
  the query features lack clear isoform association (e.g. miCLIP-seq m6A sites) and
  isoform-derived regions such as stop codons, and you need a TxIA-aware weighted
  overlap statistic rather than a raw genomic overlap count.
when_to_use_negative:
- Features already carry clear isoform/transcript association — use permTestTx or
  permTestTx_customPick with overlapCountsTx instead of the TxIA (isoform-ambiguity)
  variant
- The comparison is purely genome-based (no transcriptome annotation involved) — genome-only
  tools like regioneR or BEDTools overlap workflows are the appropriate scope
- The region of interest is directly provided as a region set rather than derived
  by a customPick function from transcripts
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: RgnTX
  role: R/Bioconductor package providing permTestTxIA_customPick, overlapCountsTxIA
    and getStopCodon used to compute the weighted overlap colocalization measure
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: R
  role: Runtime environment in which the RgnTX permutation tests and result extraction
    are executed
  repo: https://bioconductor.org/packages/RgnTX
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:c8d6b6b81ed1df9a584871702b35a0d24e8e4df41bd427f88888335605c50aab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# weighted-overlap-colocalization-measure

## Summary

A colocalization measure used with RgnTX permutation tests that counts overlaps between a query region set (e.g. miCLIP-seq m6A methylation sites with isoform ambiguity) and target transcriptome regions (e.g. UCSC hg19 stop codons), weighting counts when a region maps to multiple isoforms. It yields an observed weighted overlap, a random-overlap null mean, a permutation p-value, and a z-score.

## When to use

Use when testing association between transcriptome region sets where the query features lack clear isoform association (e.g. miCLIP-seq m6A sites) and isoform-derived regions such as stop codons, and you need a TxIA-aware weighted overlap statistic rather than a raw genomic overlap count.

## When NOT to use

- Features already carry clear isoform/transcript association — use permTestTx or permTestTx_customPick with overlapCountsTx instead of the TxIA (isoform-ambiguity) variant
- The comparison is purely genome-based (no transcriptome annotation involved) — genome-only tools like regioneR or BEDTools overlap workflows are the appropriate scope
- The region of interest is directly provided as a region set rather than derived by a customPick function from transcripts

## Inputs

- m6A RNA methylation site GRanges from RgnTX extdata m6A_sites_data.rds (500-region subset or full 9181-site set)
- TxDb.Hsapiens.UCSC.hg19.knownGene hg19 transcriptome annotation (txdb object)
- getStopCodon customPick function for deriving stop codon regions of interest

## Outputs

- Observed weighted overlap count (e.g. ~804.93 for the full 9181-site m6A set vs stop codons)
- Mean random overlap from permutations (~365.28) and the random overlap list
- Permutation p-value and z-score per test
- permtest_txIA_m6A_results.csv summarizing both m6A dataset variants

## How to apply

Load the RgnTX package with TxDb.Hsapiens.UCSC.hg19.knownGene and the package-shipped m6A_sites_data.rds (500-region subset or full 9181-site set). Call permTestTxIA_customPick with customPick_function = getStopCodon to extract stop codon regions from the hg19 transcriptome as the region of interest, ntimes = 50 permutations, ev_function_1 = overlapCountsTxIA (the weighted, isoform-ambiguity-aware measure) and ev_function_2 = overlapCountsTx. For each m6A region, the function finds the K isoforms it can align to, picks the stop-codon ROI and a same-width random region over each isoform, and averages the overlap counts by K before aggregation across permutations. Extract the observed weighted overlap (expected ~804.93 for the full 9181-site set versus a random mean ~365.28), the permutation p-value and z-score, and write the statistics for both dataset variants to permtest_txIA_m6A_results.csv.

## Related tools

- **RgnTX** (R/Bioconductor package providing permTestTxIA_customPick, overlapCountsTxIA and getStopCodon used to compute the weighted overlap colocalization measure) — https://github.com/yue-wang-biomath/RgnTX
- **R** (Runtime environment in which the RgnTX permutation tests and result extraction are executed) — https://bioconductor.org/packages/RgnTX

## Examples

```
permTestTx_results <- permTestTxIA_customPick(RS1 = RS1, txdb = txdb, customPick_function = getStopCodon, ntimes = 50, ev_function_1 = overlapCountsTxIA, ev_function_2 = overlapCountsTx)
```

## Evaluation signals

- Observed weighted overlap for the full 9181-site m6A dataset against hg19 stop codons should be ~804.93, far above the random-permutation mean of ~365.28
- Permutation p-value should be small (statistically significant association) and the z-score positive and large relative to the null distribution
- Each m6A region's overlap contribution is divided by the number K of isoforms it aligns to, so results must differ from a raw (unweighted) overlapCountsTx count
- Both the 500-region subset and the full 9181-site variant should be tested and their statistics written to permtest_txIA_m6A_results.csv
- Randomized regions must have the same width as the query regions and be sampled on the transcripts where the query regions can be aligned

## Limitations

- Handles only regions without clear isoform association — a TxIA-aware design is required because query features like POSTAR3-collected sites lack information about the specific isoform they originate from
- Null distribution of random overlaps depends on the background (DNA, exonic DNA, pre-mRNA, mRNA); genome-based backgrounds give significantly different overlap counts, so genome-based methods are not interchangeable
- No version number or R/dependency versions specified for RgnTX in the source, and no explicit code/data availability statement beyond the Bioconductor/GitHub links

## Evidence

- [results] m6A RNA methylation sites (with isoform ambiguity) from miCLIP-seq [36] and stop codon regions (without isoform ambiguity) derived from UCSC transcriptome annotation: "m6A RNA methylation sites (with isoform ambiguity) from miCLIP-seq [36] and stop codon regions (without isoform ambiguity) derived from UCSC transcriptome annotation"
- [readme] permTestTxIA_customPick tests the association between m6A sites (which specific isoform it is located on is not known) with the stop codons.: [evidence span withheld — anchor status: external_doc]
- [abstract] RgnTX supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations.: "Importantly, RgnTX supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations."
