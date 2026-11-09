---
name: naive-psi-estimation-from-inclusion-skipping-counts
description: Use when a merged allele-specific splicing count table with IJC/SJC inclusion
  and skipping junction counts plus effective isoform lengths (incLen, skpLen) is
  available and naive per-allele PSI (IncLevel) values are needed for quality filtering
  prior to ASAS testing.
when_to_use_negative:
- Input lacks per-allele IJC/SJC junction counts or effective isoform lengths (incLen/skpLen),
  so the length-adjusted inclusion ratio cannot be computed
- The analysis goal is differential splicing between experimental conditions rather
  than allele-specific alternative splicing within individuals sharing a heterozygous
  SNP
- The event's heterozygous SNP lies within or too distant from the alternative exon,
  so allele-specific read assignment on RNA-seq reads is not possible
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: PAIRADISE
  role: Produces the merged allele-specific IJC/SJC count table (Step 4) whose IncLevel
    and AvgTotalCount schema this PSI computation uses, and consumes the filtered
    events in its Step 5 statistical model
  repo: github.com/Xinglab/PAIRADISE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:cfb5a0e151d02ce2dfc5d5ca0987b5c7e92e9d569780b7c8e0a70ab31a0c941a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Xinglab/PAIRADISE
  tools:
  - PAIRADISE
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Naive PSI estimation from inclusion/skipping counts

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-allele naive exon inclusion levels (PSI) as effective-length-adjusted inclusion ratios from PAIRADISE IJC/SJC count data, along with average total read counts, to enable exclusion of unreliable exon skipping events before PAIRADISE's Step 5 ASAS statistical test.

## When to use

Use when a merged allele-specific splicing count table with IJC/SJC inclusion and skipping junction counts plus effective isoform lengths (incLen, skpLen) is available and naive per-allele PSI (IncLevel) values are needed for quality filtering prior to ASAS testing.

## When NOT to use

- Input lacks per-allele IJC/SJC junction counts or effective isoform lengths (incLen/skpLen), so the length-adjusted inclusion ratio cannot be computed
- The analysis goal is differential splicing between experimental conditions rather than allele-specific alternative splicing within individuals sharing a heterozygous SNP
- The event's heterozygous SNP lies within or too distant from the alternative exon, so allele-specific read assignment on RNA-seq reads is not possible

## Inputs

- Merged allele-specific splicing count table from PAIRADISE Step 4 count-merging stage (IJC/SJC counts for reference and alternative alleles)
- Effective isoform lengths per event (incLen, skpLen)
- Sample/replicate structure across individuals sharing a heterozygous SNP

## Outputs

- filtered_asas_counts.tsv containing retained events with original count and PSI columns
- Per-allele naive PSI (IncLevel) values and AvgTotalCount values per event

## How to apply

Load the merged count table produced by PAIRADISE Step 4 (merging the counts), which contains IJC_REF, SJC_REF, IJC_ALT, SJC_ALT inclusion/skipping junction counts plus incLen and skpLen effective isoform lengths. Compute the naive PSI per allele as the effective-length-adjusted inclusion ratio, e.g. IJC/incLen divided by (IJC/incLen + SJC/skpLen), and the average total read count per allele across individuals (the IncLevel and AvgTotalCount quantities of the PAIRADISE output schema). Apply the unreliable-event exclusion criteria: retain events only if the average exon inclusion level across all individuals is between 5% and 95% for at least one allele, removing near-constitutive and near-skipped events, and remove events whose average total read count is below 10 for either the reference or alternative allele. Write the retained events with their original count and PSI columns to filtered_asas_counts.tsv, ready for the PAIRADISE Step 5 statistical modeling and visualization.

## Related tools

- **PAIRADISE** (Produces the merged allele-specific IJC/SJC count table (Step 4) whose IncLevel and AvgTotalCount schema this PSI computation uses, and consumes the filtered events in its Step 5 statistical model) — github.com/Xinglab/PAIRADISE

## Examples

```
psi = IJC/incLen / (IJC/incLen + SJC/skpLen)
```

## Evaluation signals

- Every retained event has an average exon inclusion level within the 5-95% range for at least one allele
- Every retained event has an average total read count of at least 10 for both the reference and alternative allele
- filtered_asas_counts.tsv exists and contains the expected count-table columns (IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen plus PSI columns)
- Naive PSI values fall in [0, 1] and match recomputation from IJC/incLen and SJC/skpLen ratios
- No near-constitutive or near-skipped events (average inclusion outside 5-95% for all alleles) remain in the output

## Limitations

- Events that are near-constitutive (average inclusion <5% for all alleles) or near-skipped (>95% for all alleles) are excluded, so very high or low inclusion events cannot be tested
- Requires a heterozygous SNP outside the alternative exon but close enough for allele-specific assignment on the same RNA-seq read; SNPs within or too distant from alternative exons cannot be analyzed with short reads
- Only heterozygous SNP-tagged events with sufficient per-allele coverage (average total reads >= 10 for both alleles) can be estimated reliably

## Evidence

- [other] 5-95% inclusion filter for at least one allele: [evidence span withheld — anchor status: not_found]
- [other] Average total read count threshold of 10 per allele: [evidence span withheld — anchor status: not_found]
- [other] Naive PSI computed as effective-length-adjusted inclusion ratio: [evidence span withheld — anchor status: not_found]
- [readme] PAIRADISE output count columns defining the schema: [evidence span withheld — anchor status: not_found]
