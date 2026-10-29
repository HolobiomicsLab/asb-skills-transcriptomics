---
name: exon-inclusion-count-length-normalization
description: Use when you have per-event inclusion (IJC) and skipping (SJC) junction
  counts for reference and alternative alleles plus effective isoform lengths, and
  need length-normalized exon inclusion levels for ASAS testing.
when_to_use_negative:
- Input lacks effective isoform lengths (incLen/skpLen) — raw IJC/SJC counts alone
  cannot be length-normalized correctly
- The goal is within-condition quantification of PSI where the two isoforms have comparable
  effective lengths and the analysis is not differential (length correction is still
  advisable but this pipeline's IncLevel1/2 schema is allele-pair specific)
- Reads are not haplotype-assigned counts per allele (e.g., bulk gene-level counts
  from a standard quantifier), since IJC/SJC_REF vs IJC/SJC_ALT partitioning is required
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: PAIRADISE
  role: Performs the statistical modeling on length-normalized inclusion/skipping
    counts per allele and reports incLen/skpLen and IncLevel1/2 in its output table
  repo: https://github.com/Xinglab/PAIRADISE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:4633b2dfdcb9883b3afc7edee0ed1a6fcd75182b92764a8fde2e9fdbce4b3b53
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Xinglab/PAIRADISE
  repo_url: https://github.com/Xinglab/PAIRADISE
  tools:
  - PAIRADISE
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# exon-inclusion-count-length-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalizes splicing junction counts by effective isoform lengths to estimate percent-spliced-in (PSI) levels per allele, the quantity PAIRADISE models statistically. Effective lengths (incLen, skpLen) correct inclusion and skipping counts so length-dependent count biases do not masquerade as allelic splicing differences.

## When to use

Use when you have per-event inclusion (IJC) and skipping (SJC) junction counts for reference and alternative alleles plus effective isoform lengths, and need length-normalized exon inclusion levels for ASAS testing.

## When NOT to use

- Input lacks effective isoform lengths (incLen/skpLen) — raw IJC/SJC counts alone cannot be length-normalized correctly
- The goal is within-condition quantification of PSI where the two isoforms have comparable effective lengths and the analysis is not differential (length correction is still advisable but this pipeline's IncLevel1/2 schema is allele-pair specific)
- Reads are not haplotype-assigned counts per allele (e.g., bulk gene-level counts from a standard quantifier), since IJC/SJC_REF vs IJC/SJC_ALT partitioning is required

## Inputs

- Merged per-event count table from PAIRADISE Step 4 with columns IJC_REF, SJC_REF, IJC_ALT, SJC_ALT
- Effective isoform lengths per event (incLen, skpLen)
- Sample names from sample_name.txt

## Outputs

- Length-normalized exon inclusion levels per allele (IncLevel1, IncLevel2, reported in pairadise_significant_asas_events.tsv)
- Aggregate per-event summaries (AvgIncLevel1/2, IncLevelDifference, AvgTotalCount1/2)

## How to apply

For each splicing event and each allele, take the raw exon inclusion counts (IJC_REF, IJC_ALT) and skipping counts (SJC_REF, SJC_ALT) from the merged count table produced in Step 4 of the PAIRADISE pipeline, along with the effective isoform lengths incLen and skpLen. Compute the length-normalized inclusion level as PSI = (IJC/incLen)/((IJC/incLen)+(SJC/skpLen)) separately for the reference and alternative alleles, yielding the IncLevel1/IncLevel2 columns. These normalized inclusion levels are the inputs to the PAIRADISE paired model, which uses binomial likelihoods on the length-normalized inclusion/skipping counts per allele before running the LRT on δi. Rationale: without length correction, the inclusion isoform (typically longer) accumulates more reads, biasing PSI and inflating apparent allelic differences.

## Related tools

- **PAIRADISE** (Performs the statistical modeling on length-normalized inclusion/skipping counts per allele and reports incLen/skpLen and IncLevel1/2 in its output table) — https://github.com/Xinglab/PAIRADISE

## Examples

```
psi_ref <- (IJC_REF/incLen)/((IJC_REF/incLen)+(SJC_REF/skpLen)); psi_alt <- (IJC_ALT/incLen)/((IJC_ALT/incLen)+(SJC_ALT/skpLen))
```

## Evaluation signals

- Computed PSI values lie in (0, 1) for every event and allele, and IncLevel1 − IncLevel2 equals the reported IncLevelDifference column
- Every row of the output carries non-null incLen and skpLen effective-length values alongside the raw IJC/SJC counts
- AvgIncLevel1 and AvgIncLevel2 equal the mean of the per-sample normalized inclusion levels for their allele groups
- Re-deriving PSI from (IJC/incLen) and (SJC/skpLen) reproduces the IncLevel1/IncLevel2 values in pairadise_significant_asas_events.tsv

## Limitations

- Effective lengths must reflect the actual isoform structures of each event; miscalculated incLen/skpLen propagate directly into biased PSI estimates and downstream p-values
- Normalization is applied per allele; low IJC+SJC counts for either allele make normalized inclusion levels noisy, which the paired hierarchical model must absorb via the σi1/σi2 dispersion terms

## Evidence

- [methods] binomial likelihoods on length-normalized inclusion/skipping counts per allele, with logit(psi) normal distributions for reference and alternative alleles sharing the subject effect alpha_ik: [evidence span withheld — anchor status: not_found]
