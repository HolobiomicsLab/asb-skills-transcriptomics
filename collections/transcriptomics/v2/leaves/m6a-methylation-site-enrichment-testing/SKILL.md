---
name: m6a-methylation-site-enrichment-testing
description: Use when genomic region sets such as miCLIP-seq m6A sites lack isoform
  annotation and you need a permutation test of their association with transcriptome
  features (e.g. stop codons) under a transcriptome-wide null model.
when_to_use_negative:
- Regions already have clear isoform association (known transcript-level positions)
  — use permTestTx/permTestTx_customPick instead of the TxIA variant, which exists
  specifically for elements without clear isoform association
- You need genome-based (not transcriptome) colocalization — the paper shows null
  distributions on genome/exonic DNA/pre-mRNA/mRNA backgrounds differ significantly,
  so these transcriptome statistics are not interchangeable with genome-based tools
  run in default mode
- No TxDb transcriptome annotation matching the genome build of the sites is available
  — randomization and customPick ROI extraction (e.g. getStopCodon on hg19) cannot
  proceed
edam_operation: http://edamontology.org/operation_0240
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: RgnTX
  role: R/Bioconductor package providing permTestTxIA_customPick, getStopCodon, overlapCountsTxIA/overlapCountsTx
    and TxIA-aware randomization used to run the m6A-vs-stop-codon permutation test
    and extract weighted overlap, p-value and z-score
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: hg19 UCSC knownGene TxDb annotation supplying the transcript/exon structure
    for stop codon extraction (getStopCodon) and transcriptome-wide randomization
    of m6A sites
- name: R
  role: Runtime environment in which the RgnTX workflow is executed and permtest_txIA_m6A_results.csv
    is written
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:47b9529371018d6270cb468b89dc38891ba0c5349b0268afcb0ba7f23a298efc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  edam_operation: http://edamontology.org/operation_0240
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# m6a-methylation-site-enrichment-testing

## Summary

An RgnTX-based permutation-testing skill that quantifies the colocalization of miCLIP-seq m6A RNA methylation sites (which carry isoform ambiguity) against transcriptome regions of interest such as stop codons, using TxIA-aware randomization and weighted overlap counts over the hg19 UCSC transcriptome annotation.

## When to use

Use when genomic region sets such as miCLIP-seq m6A sites lack isoform annotation and you need a permutation test of their association with transcriptome features (e.g. stop codons) under a transcriptome-wide null model.

## When NOT to use

- Regions already have clear isoform association (known transcript-level positions) — use permTestTx/permTestTx_customPick instead of the TxIA variant, which exists specifically for elements without clear isoform association
- You need genome-based (not transcriptome) colocalization — the paper shows null distributions on genome/exonic DNA/pre-mRNA/mRNA backgrounds differ significantly, so these transcriptome statistics are not interchangeable with genome-based tools run in default mode
- No TxDb transcriptome annotation matching the genome build of the sites is available — randomization and customPick ROI extraction (e.g. getStopCodon on hg19) cannot proceed

## Inputs

- miCLIP-seq m6A RNA methylation site GRanges with isoform ambiguity (RgnTX extdata/m6A_sites_data.rds; 500-region subset or full 9181-site set)
- TxDb transcriptome annotation object: TxDb.Hsapiens.UCSC.hg19.knownGene (hg19)
- Custom pick function defining regions of interest, e.g. getStopCodon, supplied via customPick_function

## Outputs

- permTestTxIA_customPick result object containing ROI, random_region_list, observed weighted overlap, random_overlap_list, pvalue and zscore
- permtest_txIA_m6A_results.csv summarizing observed overlap, mean random overlap, p-value and z-score for both m6A variants (500-region subset and 9181-site set)

## How to apply

Start an R session, load RgnTX with TxDb.Hsapiens.UCSC.hg19.knownGene as txdb, and read the shipped m6A_sites_data.rds from extdata (either the 500-region subset or the full 9181-site variant). Call permTestTxIA_customPick with customPick_function = getStopCodon so stop codon ROI are picked per transcript, with ev_function_1 = overlapCountsTxIA (isoform-ambiguity-aware weighted overlap) and ev_function_2 = overlapCountsTx; the README example uses ntimes = 50, while the paper shuffled all m6A sites into independent groups of 100 regions with 100 permutations per group. The TxIA randomization finds the K isoforms each site can align to, picks equal-width random regions on each isoform, and averages overlap counts by K, so no prior isoform labels are needed. Extract the observed weighted overlap (approximately 804.93 for the full 9181-site set), the mean random overlap (approximately 365.28), the permutation p-value and the z-score from each test result. Judge success by the observed weighted overlap far exceeding the random-overlap null, with a significant p-value/z-score, then write both variants' statistics into permtest_txIA_m6A_results.csv.

## Related tools

- **RgnTX** (R/Bioconductor package providing permTestTxIA_customPick, getStopCodon, overlapCountsTxIA/overlapCountsTx and TxIA-aware randomization used to run the m6A-vs-stop-codon permutation test and extract weighted overlap, p-value and z-score) — https://github.com/yue-wang-biomath/RgnTX
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (hg19 UCSC knownGene TxDb annotation supplying the transcript/exon structure for stop codon extraction (getStopCodon) and transcriptome-wide randomization of m6A sites)
- **R** (Runtime environment in which the RgnTX workflow is executed and permtest_txIA_m6A_results.csv is written)

## Examples

```
permTestTxIA_customPick(RS1 = m6A_sites_data[1:500], txdb = txdb, customPick_function = getStopCodon, ntimes = 50, ev_function_1 = overlapCountsTxIA, ev_function_2 = overlapCountsTx)
```

## Evaluation signals

- Full 9181-site m6A dataset must yield an observed weighted overlap near 804.93 against stop codon regions, versus a mean random overlap near 365.28
- Permutation p-value and z-score must reflect the observed overlap being far above the random-overlap null distribution (statistically significant association)
- Result object must conform to the function contract: ROI, random_region_list, observed_overlap, random_overlap_list, pvalue and zscore fields
- ev_function_1 must be overlapCountsTxIA while ev_function_2 is overlapCountsTx — swapping the evaluation functions invalidates the TxIA-aware test
- Both m6A variants (500-region subset and 9181-site set) must be tested and their statistics present in permtest_txIA_m6A_results.csv

## Limitations

- Permutation count controls p-value granularity — the README example uses ntimes = 50, and the paper's design (groups of 100 regions, 100 permutations) should be matched for reproducible statistics
- TxIA handling assumes each isoform-ambiguous site maps to K isoforms and averages overlap by K; results are anchored to the hg19 UCSC knownGene transcriptome, so other genome builds change ROI and the null model
- RgnTX novel testing modes return distinct and more significant results than existing genome-based methods, so numbers are not directly comparable to genome-based tool defaults (GREAT, LOLA, GenomeRunner, etc.)

## Evidence

- [abstract] Importantly, it supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations.: "Importantly, it supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations."
- [results] m6A RNA methylation sites (with isoform ambiguity) from miCLIP-seq [36] and stop codon regions (without isoform ambiguity) derived from UCSC transcriptome annotation: "m6A RNA methylation sites (with isoform ambiguity) from miCLIP-seq [36] and stop codon regions (without isoform ambiguity) derived from UCSC transcriptome annotation"
- [readme] The following example shows `permTestTxIA_customPick` tests the association between m6A sites (which specific isoform it is located on is not known) with the stop codons.: [evidence span withheld — anchor status: external_doc]
- [results] We randomly shuffled all the m6A sites and divided them into independent groups. Each independent group contains 100 regions: "We randomly shuffled all the m6A sites and divided them into independent groups. Each independent group contains 100 regions"
