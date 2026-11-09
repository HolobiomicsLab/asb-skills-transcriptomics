---
name: stop-codon-region-extraction-custompick
description: Use when a permutation colocalization test needs transcriptome regions
  of interest (e.g. stop codons) that are not provided directly as a GRanges set and
  must be picked per-transcript from UCSC hg19 TxDb annotation for features with unknown
  isoform association.
when_to_use_negative:
- RS2 is already available as an explicit region set — use permTestTx with a directly
  supplied RS2 instead of a customPick function.
- Each feature's specific isoform association is known — use permTestTx_customPick
  (non-IA) since the TxIA averaging over K isoforms is unnecessary.
- The question is genome-based colocalization without transcriptome context — genome-based
  tools (GREAT, LOLA, GenomeRunner, BEDTools) do not model transcriptome backgrounds
  and RgnTX's null distributions on mRNA space are significantly different from genomic
  backgrounds.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: RgnTX
  role: Provides the getStopCodon customPick function and permTestTxIA_customPick
    to extract stop codon ROI and run the isoform-ambiguity-aware permutation test.
  repo: https://bioconductor.org/packages/RgnTX
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: UCSC hg19 transcriptome annotation from which the getStopCodon function derives
    stop codon regions (the isoform-unambiguous reference set).
- name: R
  role: Runtime environment for the RgnTX package and TxDb annotation.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:8c9373ec26352d4b491dd090605743d2441fc1e553b27bd6a7b44ee26683d74b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# stop-codon-region-extraction-custompick

## Summary

Uses RgnTX's getStopCodon customPick function to derive stop codon regions from the UCSC hg19 transcriptome annotation (TxDb.Hsapiens.UCSC.hg19.knownGene) as transcriptome regions of interest (ROI), for use in colocalization permutation tests such as permTestTxIA_customPick against miCLIP-seq m6A sites that carry isoform ambiguity.

## When to use

Use when a permutation colocalization test needs transcriptome regions of interest (e.g. stop codons) that are not provided directly as a GRanges set and must be picked per-transcript from UCSC hg19 TxDb annotation for features with unknown isoform association.

## When NOT to use

- RS2 is already available as an explicit region set — use permTestTx with a directly supplied RS2 instead of a customPick function.
- Each feature's specific isoform association is known — use permTestTx_customPick (non-IA) since the TxIA averaging over K isoforms is unnecessary.
- The question is genome-based colocalization without transcriptome context — genome-based tools (GREAT, LOLA, GenomeRunner, BEDTools) do not model transcriptome backgrounds and RgnTX's null distributions on mRNA space are significantly different from genomic backgrounds.

## Inputs

- RgnTX m6A_sites_data.rds (miCLIP-seq m6A sites; 500-region subset or full 9181-site variant) as GRanges with only genomic positions
- TxDb.Hsapiens.UCSC.hg19.knownGene transcriptome annotation object (txdb)
- getStopCodon customPick function shipped with RgnTX

## Outputs

- Stop codon ROI region set (GRanges) picked per transcript from hg19 annotation
- permTestTxIA_customPick result object with observed weighted overlap (~804.93 for 9181 sites), random_overlap_list (mean ~365.28), p-value and z-score
- Statistics table (e.g. permtest_txIA_m6A_results.csv) comparing the 500-region and 9181-site variants

## How to apply

Load RgnTX with TxDb.Hsapiens.UCSC.hg19.knownGene as the txdb. Pass customPick_function = getStopCodon to permTestTxIA_customPick: for each input region RS1_i (e.g. one of the 9181 miCLIP-seq m6A sites, or the 500-region subset), the function finds the K isoforms RS1_i can be aligned to and picks the stop codon region over each isoform, so the ROI is generated rather than supplied directly. Rationale: stop codon regions have no isoform ambiguity themselves (they derive from the UCSC transcriptome annotation), while m6A sites only have genomic positions, so the TxIA-aware path averages observed overlap as overlap_counts(RS1_i, ROI_i)/K and randomizes each region over every isoform it maps to. Evaluate association with ev_function_1 = overlapCountsTxIA (weighted overlap) and ev_function_2 = overlapCountsTx, e.g. ntimes = 50 permutations; the full 9181-site m6A set gives an observed weighted overlap of ~804.93 versus a mean random overlap of ~365.28.

## Related tools

- **RgnTX** (Provides the getStopCodon customPick function and permTestTxIA_customPick to extract stop codon ROI and run the isoform-ambiguity-aware permutation test.) — https://bioconductor.org/packages/RgnTX
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (UCSC hg19 transcriptome annotation from which the getStopCodon function derives stop codon regions (the isoform-unambiguous reference set).)
- **R** (Runtime environment for the RgnTX package and TxDb annotation.)

## Examples

```
permTestTxIA_customPick(RS1 = m6A_sites_data[1:500], txdb = txdb, customPick_function = getStopCodon, ntimes = 50, ev_function_1 = overlapCountsTxIA, ev_function_2 = overlapCountsTx)
```

## Evaluation signals

- The full 9181-site m6A test yields an observed weighted overlap of ~804.93 and a mean random overlap of ~365.28; large deviations from these values indicate a setup error (wrong txdb, wrong customPick, or wrong ev_function).
- The permutation p-value should be far in the tail (observed overlap far above what random permutation produces), consistent with the paper's reported statistically significant m6A–stop codon association.
- ROI returned by the customPick path is non-empty and derived per isoform: for each RS1_i the function must resolve the K aligned isoforms before picking stop codon regions.
- ev_function_1 must be overlapCountsTxIA (TxIA-aware weighted counts), not overlapCountsTx, since the input m6A sites lack isoform association.
- Both the 500-region and 9181-site variants should produce per-test statistics that can be written to permtest_txIA_m6A_results.csv without missing fields (observed, mean random, p-value, z-score).

## Limitations

- Stop codon ROI depends entirely on the UCSC hg19 knownGene annotation; alternative annotations or genomes require a different TxDb and may change the region set.
- Regions lacking any isoform association cannot be projected — the TxIA randomization handles regions without clear isoform association by averaging over aligned isoforms, but features mapping to no transcript cannot be tested.
- Statistical power depends on the number of permutations (ntimes, e.g. 50 in the example); the article grouped m6A sites into independent groups of 100 regions with 100 permutations and applied Benjamini-Hochberg correction for multiple testing.
- No RgnTX version, R version, or dependency versions are stated in the article or README.

## Evidence

- [readme] The following example shows permTestTxIA_customPick tests the association between m6A sites (which specific isoform it is located on is not known) with the stop codons.: [evidence span withheld — anchor status: external_doc]
- [readme] The ROI is not directly provided, but generated by the input 'customPick_function'.: [evidence span withheld — anchor status: external_doc]
- [results] m6A RNA methylation sites (with isoform ambiguity) from miCLIP-seq and stop codon regions (without isoform ambiguity) derived from UCSC transcriptome annotation.: "m6A RNA methylation sites (with isoform ambiguity) from miCLIP-seq [36] and stop codon regions (without isoform ambiguity) derived from UCSC transcriptome annotation"
- [results] We used the annotation database from UCSC to extract the following backgrounds.: "We used the annotation database from UCSC to extract the following backgrounds"
- [abstract] It supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations.: "Importantly, it supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations."
