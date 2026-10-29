---
name: as-event-count-threshold-filtering
description: Use when a merged allele-specific splicing count table (IJC/SJC counts
  for reference and alternative alleles plus effective isoform lengths) has been produced
  by PAIRADISE Step 4 and needs to be cleaned before running the paired ASAS statistical
  test.
when_to_use_negative:
- 'Before Step 4 of the PAIRADISE pipeline: per-sample counts are not yet merged,
  so per-event average inclusion levels and average total counts across individuals
  cannot be computed'
- When the goal is read-level QC (e.g., discarding reads not uniquely aligned to both
  haplotype-modified genomes, or multi-SNP reads with a majority-voting draw) — those
  are alignment/assignment filters upstream of event-level filtering
- When analyzing SNPs located within or too distant from alternative exons, which
  PAIRADISE cannot analyze with short-read RNA-seq data regardless of count thresholds
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: PAIRADISE
  role: Produces the merged allele-specific count table (Step 4) that this filter
    consumes, and its Step 5 statistical model consumes the filtered output; IncLevel
    and AvgTotalCount are the quantities the filter thresholds.
  repo: https://github.com/Xinglab/PAIRADISE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ajhg.2020.07.005
    title: Detecting Allele-Specific Alternative Splicing from Population-Scale RNA-Seq
      Data
  source_content_hash: sha256:8a7f680664a7566f10659c8c7d3b9216d3d0861fedfbb103da80f4a5c992cbdf
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# AS event count threshold filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A quality-control pre-filtering step in the PAIRADISE ASAS pipeline that excludes unreliable exon skipping events before the Step 5 statistical model, retaining only events with average exon inclusion level between 5% and 95% for at least one allele and average total read counts of at least 10 for both alleles.

## When to use

Use when a merged allele-specific splicing count table (IJC/SJC counts for reference and alternative alleles plus effective isoform lengths) has been produced by PAIRADISE Step 4 and needs to be cleaned before running the paired ASAS statistical test.

## When NOT to use

- Before Step 4 of the PAIRADISE pipeline: per-sample counts are not yet merged, so per-event average inclusion levels and average total counts across individuals cannot be computed
- When the goal is read-level QC (e.g., discarding reads not uniquely aligned to both haplotype-modified genomes, or multi-SNP reads with a majority-voting draw) — those are alignment/assignment filters upstream of event-level filtering
- When analyzing SNPs located within or too distant from alternative exons, which PAIRADISE cannot analyze with short-read RNA-seq data regardless of count thresholds

## Inputs

- Merged allele-specific splicing count table from PAIRADISE Step 4, with IJC/SJC counts for reference (IJC_REF, SJC_REF) and alternative (IJC_ALT, SJC_ALT) alleles
- Effective lengths of the exon inclusion and skipping isoforms (incLen, skpLen) per event

## Outputs

- filtered_asas_counts.tsv: count table of retained ASAS events satisfying the 5%-95% average inclusion-level criterion for at least one allele and average total read counts >= 10 for both alleles, with original count and psi columns preserved

## How to apply

Load the merged allele-specific splicing count table from PAIRADISE Step 4 (count merging), which contains IJC/SJC inclusion and skipping junction counts for the reference and alternative alleles plus incLen/skpLen effective isoform lengths. Compute per-allele naive psi values as the effective-length-adjusted inclusion ratio (the IncLevel quantities in the PAIRADISE output schema) and the average total read count per allele across samples (AvgTotalCount). Remove events whose average exon inclusion level falls outside 5%-95% for all alleles, eliminating near-constitutive and near-skipped events, then remove events whose average total read count is below 10 for either the reference or the alternative allele. Write the retained events with their original count and psi columns to filtered_asas_counts.tsv, ready for the PAIRADISE Step 5 statistical modeling and visualization.

## Related tools

- **PAIRADISE** (Produces the merged allele-specific count table (Step 4) that this filter consumes, and its Step 5 statistical model consumes the filtered output; IncLevel and AvgTotalCount are the quantities the filter thresholds.) — https://github.com/Xinglab/PAIRADISE

## Examples

```
qsub -cwd -l h_vmem=10G qsub_files/pairadise_step4_merge.qsub
```

## Evaluation signals

- Every retained event satisfies both thresholds: average exon inclusion level within 5%-95% for at least one allele, and average total read counts of at least 10 for both the reference and alternative alleles
- No retained event is near-constitutive or near-skipped (average inclusion level outside 5%-95% for all alleles) nor has AvgTotalCount < 10 for either allele
- filtered_asas_counts.tsv exists and carries the expected PAIRADISE count-table columns (IJC_REF, SJC_REF, IJC_ALT, SJC_ALT, incLen, skpLen)
- The filtered table feeds Step 5 (pairadise_step5_stat.qsub) without schema errors, indicating the psi/count columns were preserved

## Limitations

- Events with average inclusion level outside 5%-95% for all alleles are dropped, so near-constitutive and near-skipped events are never tested for ASAS
- Events with average total read counts below 10 for either allele are excluded, reducing power for lowly expressed exons
- Downstream ASAS detection additionally requires a heterozygous SNP outside the alternative exon but close enough to be detected on the same RNA-seq read; SNPs within or too distant from alternative exons cannot be analyzed with short-read data

## Evidence

- [full_text] Events are retained only if the average exon inclusion level across all individuals is between 5% and 95% for at least one allele, and the average total read counts of all individuals are no less than 10 for both alleles.: [evidence span withheld — anchor status: not_found]
- [readme] README output schema for the PAIRADISE count table.: "**IJC_REF**: Exon inclusion counts for the reference allele."
