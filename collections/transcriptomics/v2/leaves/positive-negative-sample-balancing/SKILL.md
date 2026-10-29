---
name: positive-negative-sample-balancing
description: Use when building benchmark datasets for a tissue- or species-specific
  site-prediction model and you need equal positive and negative sequence counts in
  both training and testing partitions to avoid class-imbalance bias.
when_to_use_negative:
- Your task allows class imbalance and you plan to use weighted loss or resampling
  during model training instead — this skill hard-balances at dataset construction
  time.
- The input sequences have not yet been deduplicated; balancing should follow CD-HIT
  filtering at <80% similarity, not precede it.
- You are working outside the m6A tissue-prediction setting (e.g., whole-transcript
  classification) where segment-centered 41-nt positive/negative labeling does not
  apply.
edam_operation: http://edamontology.org/operation_3472
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3512
tools:
- name: im6A-TS-CNN
  role: Consumes the balanced per-tissue training and testing datasets for CNN-based
    m6A site prediction using one-hot encoding
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: CD-HIT
  role: Applied to each tissue dataset before balancing to ensure sequence similarity
    is less than 80%, removing redundant sequences so balanced subsets are non-redundant
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:27bea7e161021874bd5834f190be465c02374211ca74f0d1a90d8f6db61d39f6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/liukeweiaway/DeepM6A_cnn
  repo_url: https://github.com/liukeweiaway/DeepM6A_cnn
  tools:
  - CD-HIT
  - im6A-TS-CNN
  edam_operation: http://edamontology.org/operation_3472
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# positive-negative-sample-balancing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct class-balanced RNA benchmark datasets by pairing each positive (m6A-containing) 41-nt segment with an equal count of negative (m6A-free) segments, then splitting into training and testing sets with matched positive/negative counts per tissue.

## When to use

Use when building benchmark datasets for a tissue- or species-specific site-prediction model and you need equal positive and negative sequence counts in both training and testing partitions to avoid class-imbalance bias.

## When NOT to use

- Your task allows class imbalance and you plan to use weighted loss or resampling during model training instead — this skill hard-balances at dataset construction time.
- The input sequences have not yet been deduplicated; balancing should follow CD-HIT filtering at <80% similarity, not precede it.
- You are working outside the m6A tissue-prediction setting (e.g., whole-transcript classification) where segment-centered 41-nt positive/negative labeling does not apply.

## Inputs

- 41-nt RNA segments centered on candidate m6A sites, labeled positive (containing m6A) or negative (no m6A), for each of the 11 species-tissue datasets
- CD-HIT-filtered (sequence similarity < 80%) positive and negative sequence FASTA sets per tissue

## Outputs

- Balanced training and testing subsets per tissue dataset with equal positive/negative counts (e.g., h_b: 4,605/4,605 train, 4,604/4,604 test)
- 11 curated species-tissue benchmark datasets matching the counts reported in Table 3

## How to apply

After extracting 41-nt RNA segments centered on candidate m6A sites (positives) and non-site positions (negatives), run CD-HIT with a sequence similarity cutoff below 80% to remove redundancy within each tissue dataset. Then partition each filtered tissue dataset into training and testing subsets such that positives and negatives are exactly equal in each partition, following the per-dataset counts in Table 3 (e.g., h_b: 4,605/4,605 training positives/negatives and 4,604/4,604 testing positives/negatives; m_b: 8,025/8,025/8,025/8,025). Balance counts across all 11 species-tissue datasets (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, and the remaining tissue) so downstream CNN training with one-hot encoding and evaluation by Sn, Sp, Acc, MCC, and AUC is not skewed by class imbalance.

## Related tools

- **im6A-TS-CNN** (Consumes the balanced per-tissue training and testing datasets for CNN-based m6A site prediction using one-hot encoding) — https://github.com/liukeweiaway/DeepM6A_cnn
- **CD-HIT** (Applied to each tissue dataset before balancing to ensure sequence similarity is less than 80%, removing redundant sequences so balanced subsets are non-redundant)

## Examples

```
python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- Verify positive and negative counts are exactly equal within each training and each testing subset (e.g., h_b: 4,605/4,605 training and 4,604/4,604 testing; m_b: 8,025/8,025/8,025/8,025).
- Confirm the 11 curated datasets' counts exactly match the training/testing positive/negative counts reported in Table 3.
- Check that CD-HIT filtering at <80% similarity was applied to each tissue dataset before the balanced partitioning step.
- Confirm all segments are 41 nt long and centered on the candidate site, with positives containing an m6A site and negatives not.
- Downstream sanity check: 5-fold cross-validation accuracy/AUC (e.g., h_b Acc 72.53%, AUC 0.8029) should be in a plausible range, indicating no leakage from redundancy.

## Limitations

- Balancing is per tissue dataset; the absolute dataset sizes differ substantially across tissues (e.g., r_k has 1,762 pairs while m_b has 8,025), so cross-tissue sample sizes are not comparable.
- The balance depends on prior CD-HIT deduplication at <80% similarity; if filtering is skipped, near-identical sequences may leak across train/test splits.
- Threshold performance with different options was deferred to supplementary Table S1 and is not detailed in the main text.

## Evidence

- [other] Validation step confirms dataset counts were verified against reported counts: [evidence span withheld — anchor status: not_found]
