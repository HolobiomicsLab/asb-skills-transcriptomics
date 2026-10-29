---
name: top-k-candidate-benchmark-evaluation-against-validated-labels
description: Use when you have a ranked list of predicted peptide-MHC candidates and
  an external gold-standard set of experimentally validated immunogenic labels (e.g.,
  TESLA syn21048999), and you need to quantify whether top-k selection enriches true
  immunogenic neoantigens above baseline.
when_to_use_negative:
- You lack experimentally validated immunogenic labels for the candidate pMHCs — ranking
  alone cannot be scored and the 6.1% baseline comparison is not possible
- Your candidates have not passed the DeepHLApan binding/immunogenic score >0.5 and
  TPM >= 1 filters, so top-k lists would be dominated by low-confidence pairs
- You need de novo immunogenicity prediction (e.g., TCR-pMHC binding modeling), which
  TSNAD v2.0 explicitly does not provide
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0121
tools:
- name: DeepHLApan
  role: Supplies the binding and immunogenic scores (>0.5 thresholds) used to rank
    and filter candidate peptide-MHC pairs into top-k lists
  repo: https://github.com/jiujiezz/tsnad
- name: TSNAD v2.0
  role: One-stop pipeline that produces the candidate neoantigens (via BWA, GATK Mutect2,
    OptiType, VEP) under GRCh37 or GRCh38 that are then benchmarked
  repo: https://github.com/jiujiezz/tsnad
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:97b548b3f7dde4e2e593b0f3e203c4218ff9f16fb229f145ffa752a74dbc05a7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jiujiezz/tsnad
  repo_url: https://github.com/jiujiezz/tsnad
  tools:
  - BWA
  - DeepHLApan
  - Docker
  - GATK
  - HISAT2
  - OptiType
  - SAMtools
  - StringTie
  - TSNAD v2.0
  - Trimmomatic
  - VEP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# Top-k candidate benchmark evaluation against validated labels

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a neoantigen prediction pipeline by ranking candidate peptide-MHC pairs per peptide length by DeepHLApan binding score, forming top-10/20/30/40/50 candidate lists, and scoring them against TESLA-validated immunogenic pMHC labels to compare achieved positive rates with the 6.1% TESLA baseline.

## When to use

Use when you have a ranked list of predicted peptide-MHC candidates and an external gold-standard set of experimentally validated immunogenic labels (e.g., TESLA syn21048999), and you need to quantify whether top-k selection enriches true immunogenic neoantigens above baseline.

## When NOT to use

- You lack experimentally validated immunogenic labels for the candidate pMHCs — ranking alone cannot be scored and the 6.1% baseline comparison is not possible
- Your candidates have not passed the DeepHLApan binding/immunogenic score >0.5 and TPM >= 1 filters, so top-k lists would be dominated by low-confidence pairs
- You need de novo immunogenicity prediction (e.g., TCR-pMHC binding modeling), which TSNAD v2.0 explicitly does not provide

## Inputs

- Ranked candidate peptide-MHC pairs with DeepHLApan binding and immunogenic scores (per patient, peptide lengths 8-11)
- TESLA-validated immunogenic pMHC labels from Synapse (syn21048999)
- Per-sample high-confidence neoantigen lists (immunogenic score >0.5, binding score top-20 per peptide length)
- Positive-rate baseline from TESLA: 37 immunogenic out of 608 tested neoantigens

## Outputs

- Per-k (top-10/20/30/40/50) accuracy/positive-rate metrics under GRCh37 and GRCh38
- tesla_benchmark_results.csv with per-k accuracy metrics and baseline comparison

## How to apply

After TSNAD v2.0 produces candidate pMHCs (peptides of 8-11 amino acids with DeepHLApan binding score >0.5 and immunogenic score >0.5, TPM >= 1 expression filter applied), rank the binding-scored pairs per peptide length and apply the top-20-per-length rule (yielding 80 high-confidence neoantigens per sample). Form top-10/20/30/40/50 candidate lists for each patient under each reference genome (GRCh37 and GRCh38), then score each top-k list against the TESLA-validated immunogenic pMHC labels, computing the achieved positive rate per k. Judge success by comparing against the TESLA baseline where 37 of 608 predicted neoantigens (6.1%) from 28 teams were immunogenic; the paper reports that even the poorest top-10 selection on GRCh38 achieves 11.1% accuracy, exceeding this baseline. Write the per-k accuracy metrics for both reference genomes and the baseline comparison to tesla_benchmark_results.csv.

## Related tools

- **DeepHLApan** (Supplies the binding and immunogenic scores (>0.5 thresholds) used to rank and filter candidate peptide-MHC pairs into top-k lists) — https://github.com/jiujiezz/tsnad
- **TSNAD v2.0** (One-stop pipeline that produces the candidate neoantigens (via BWA, GATK Mutect2, OptiType, VEP) under GRCh37 or GRCh38 that are then benchmarked) — https://github.com/jiujiezz/tsnad

## Examples

```
python TSNAD.py -I samples/ -R RNA-seq/ -V grch38 -O results/
```

## Evaluation signals

- Per-k positive rate exceeds the TESLA baseline of 37/608 = 6.1% immunogenic
- GRCh38 run's poorest top-10 selection achieves at least 11.1% accuracy per the paper
- Each patient's high-confidence set contains 80 neoantigens (top-20 binding-ranked per peptide length, lengths 8-11)
- All candidates in the scored lists have binding score >0.5 and immunogenic score >0.5
- tesla_benchmark_results.csv contains per-k accuracy rows for both GRCh37 and GRCh38 reference genomes

## Limitations

- Only 37 of 608 TESLA-tested neoantigens (6.1%) were immunogenic, so accuracy numbers are low in absolute terms and depend on the baseline composition
- The paper does not include a TCR-pMHC binding prediction model; T-cell receptor sequencing and TCR-pMHC interaction analysis are planned for future updates
- Ranking is by binding score only within the top-20-per-length rule; alternative ranking strategies (e.g., immunogenic score) are not benchmarked

## Evidence

- [results] Even TSNAD v2.0's poorest top-10 selection on GRCh38 achieves 11.1% accuracy, exceeding the 6.1% overall positive rate of the TESLA dataset: [evidence span withheld — anchor status: not_found]
