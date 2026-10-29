---
name: isoform-ambiguity-fractional-overlap-scoring
description: Use when a region set (e.g. m6A sites from POSTAR3/miCLIP-seq) has genomic
  positions but no known transcript isoform association, and you need an overlap score
  or permutation test against transcriptome regions of interest such as stop codons.
when_to_use_negative:
- The region set already has clear isoform association (each element maps to a known
  transcript) — use permTestTx/permTestTx_customPick with overlapCountsTx instead.
- The analysis target is genome-based colocalization of genomic region sets where
  transcriptome heterogeneity is irrelevant (e.g. ChIP-seq peaks vs genome intervals
  via LOLA/GREAT/BEDTools).
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RgnTX
  role: Provides overlapCountsTxIA and permTestTxIA_customPick, which implement the
    fractional per-isoform overlap scoring for features with isoform ambiguity
  repo: https://github.com/yue-wang-biomath/RgnTX
- name: R
  role: Runtime language; RgnTX is an R/Bioconductor package invoked from R scripts
  repo: https://bioconductor.org/packages/RgnTX
- name: TxDb.Hsapiens.UCSC.hg19.knownGene
  role: Supplies UCSC hg19 transcriptome annotation (transcripts, exons, CDS, stop
    codons) used to resolve isoform candidates and pick ROIs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.08.021
    title: 'RgnTX: Colocalization analysis of transcriptome elements in the presence
      of isoform heterogeneity and ambiguity'
  source_content_hash: sha256:307aa95d27c2ac1bfb375bdea8e3b0dfa926ae47343f3199496c93b2b652e665
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yue-wang-biomath/RgnTX
  tools:
  - R
  - RgnTX
  - TxDb.Hsapiens.UCSC.hg19.knownGene
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# isoform-ambiguity-fractional-overlap-scoring

## Summary

Compute colocalization overlap scores for transcriptome features that carry isoform ambiguity by summing per-feature fractional contributions c(i)/n(i), where c(i) is the count of overlaps with the transcriptome ROIs and n(i) is the number of isoform transcripts the feature can be aligned to, as implemented in RgnTX's overlapCountsTxIA.

## When to use

Use when a region set (e.g. m6A sites from POSTAR3/miCLIP-seq) has genomic positions but no known transcript isoform association, and you need an overlap score or permutation test against transcriptome regions of interest such as stop codons.

## When NOT to use

- The region set already has clear isoform association (each element maps to a known transcript) — use permTestTx/permTestTx_customPick with overlapCountsTx instead.
- The analysis target is genome-based colocalization of genomic region sets where transcriptome heterogeneity is irrelevant (e.g. ChIP-seq peaks vs genome intervals via LOLA/GREAT/BEDTools).

## Inputs

- GRanges region set RS1 with genomic positions only (no isoform association), e.g. m6A sites from m6A_sites_data.rds
- TxDb annotation object, e.g. TxDb.Hsapiens.UCSC.hg19.knownGene
- customPick_function that returns transcriptome regions of interest (ROI) per transcript, e.g. getStopCodon
- number of permutations (ntimes) and evaluation functions (ev_function_1 = overlapCountsTxIA, ev_function_2 = overlapCountsTx)

## Outputs

- Per-feature IA overlap contributions c(i)/n(i), each in [0, 1]
- observed_overlap (IA overlap score) between RS1 and ROI
- random_overlap_list from ntimes permutations (null distribution)
- p-value and z-score from permTestTxIA_customPick; comparison table (feature, custom IA score, overlapCountsTxIA score, c(i)/n(i) ratio) e.g. ia_overlap_verification.csv

## How to apply

For each ambiguous feature i in RS1, first find the K isoform transcripts that RS1_i can be aligned to using the txdb annotation (e.g. TxDb.Hsapiens.UCSC.hg19.knownGene). Then, for each isoform k, let the customPick_function (e.g. getStopCodon) pick the ROI over that isoform and count overlaps of RS1_i with the ROI collection, dividing the per-feature count by K to obtain a contribution clamped to [0, 1]. Accumulate these per-feature ratios c(i)/n(i) into the IA overlap score; in the permutation-test wrapper permTestTxIA_customPick the same scheme is applied to random regions generated per isoform over ntimes permutations to build the null distribution. Rationale: because technical limitations such as in POSTAR3 mean no specific isoform is known, fractional attribution over all candidate isoforms avoids discarding features or arbitrarily assigning one isoform. Verify a from-scratch implementation against overlapCountsTxIA outputs on a synthetic feature set, asserting equality within numerical tolerance and that every per-feature ratio lies in [0, 1].

## Related tools

- **RgnTX** (Provides overlapCountsTxIA and permTestTxIA_customPick, which implement the fractional per-isoform overlap scoring for features with isoform ambiguity) — https://github.com/yue-wang-biomath/RgnTX
- **R** (Runtime language; RgnTX is an R/Bioconductor package invoked from R scripts) — https://bioconductor.org/packages/RgnTX
- **TxDb.Hsapiens.UCSC.hg19.knownGene** (Supplies UCSC hg19 transcriptome annotation (transcripts, exons, CDS, stop codons) used to resolve isoform candidates and pick ROIs)

## Examples

```
permTestTx_results <- permTestTxIA_customPick(RS1 = m6A_sites_data[1:500], txdb = txdb, customPick_function = getStopCodon, ntimes = 50, ev_function_1 = overlapCountsTxIA, ev_function_2 = overlapCountsTx)
```

## Evaluation signals

- Every per-feature contribution c(i)/n(i) lies within [0, 1]
- Custom IA scores match overlapCountsTxIA outputs on a synthetic feature set within numerical tolerance for every feature
- Comparison table written to ia_overlap_verification.csv contains feature, custom IA score, overlapCountsTxIA score, and per-feature c(i)/n(i) ratio columns
- In permTestTxIA_customPick, the observed_overlap and each random_overlap_i are divided by the number of candidate isoforms K per feature
- Returned permTestTx results include observed_overlap, random_region_list, random_overlap_list, pvalue, and zscore

## Limitations

- Score quality depends on the txdb annotation completeness; isoform candidates are only those K isoforms present in the UCSC hg19 knownGene annotation.
- Features outside all annotated isoforms cannot contribute overlaps and may be dropped or contribute 0.
- No version number or dependency versions are stated for RgnTX, so reproducibility across package versions is not guaranteed.
- Permutation-based p-values depend on ntimes (e.g. 50–100 permutations in the article's tests), giving limited resolution at low permutation counts.

## Evidence

- [readme] The function permTestTxIA_customPick performs permutation test to evaluate association between genomic features (RS) that one only know its genomic positions and do not know which specific transcript is should be aligned to: [evidence span withheld — anchor status: not_found]
- [readme] observed_overlap_i: overlapping counts between RS1_i and ROI_i / K: [evidence span withheld — anchor status: external_doc]
- [abstract] Importantly, it supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations.: "Importantly, it supports the testing of transcriptome elements without clear isoform association, which is often the real scenario due to technical limitations."
- [results] m6A RNA methylation sites (with isoform ambiguity) from miCLIP-seq [36] and stop codon regions (without isoform ambiguity) derived from UCSC transcriptome annotation: "m6A RNA methylation sites (with isoform ambiguity) from miCLIP-seq [36] and stop codon regions (without isoform ambiguity) derived from UCSC transcriptome annotation"
- [intro] To date, there are no computational methods available to address the issue of isoform ambiguity during transcriptome element colocalization analysis.: "To date, there are no computational methods available to address the issue of isoform ambiguity during transcriptome element colocalization analysis."
