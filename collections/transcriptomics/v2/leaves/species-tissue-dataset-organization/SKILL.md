---
name: species-tissue-dataset-organization
description: Use when building species- and tissue-stratified sequence benchmark sets
  from site-level modification calls, and redundancy must be removed before balanced
  train/test partitioning (e.g., CD-HIT <80% cutoff).
when_to_use_negative:
- Sequences are already deduplicated or you are reusing the published Table 3 benchmark
  splits — re-running CD-HIT and re-splitting would break comparability with reported
  results
- Your modification sites come from antibody-based (m6A-seq/MeRIP) peak data rather
  than the antibody-independent endoribonuclease detection data, since single-nucleotide
  resolution of the site center is required for 41-nt segment labeling
- The task is single-species, single-tissue prediction with no need for tissue stratification
  — a generic sequence benchmark suffices
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3512
tools:
- name: CD-HIT
  role: Clustering tool applied to each tissue dataset to enforce a sequence similarity
    cutoff below 80%, removing redundant 41-nt segments before train/test partitioning
- name: im6A-TS-CNN
  role: Downstream CNN classifier whose 5-fold cross-validation and independent tests
    consume the curated, balanced species-tissue datasets
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:34272c9da143f22959d23f8504df195acfe81e9399a19d7bb5406b0170003ec1
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# species-tissue-dataset-organization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curate balanced, tissue-specific RNA benchmark datasets (e.g., 11 species-tissue m6A sets spanning brain, liver, kidney, heart, and testis of human, mouse, and rat) by extracting 41-nt segments centered on candidate modification sites and deduplicating with CD-HIT below 80% sequence similarity before splitting into equal positive/negative training and testing subsets.

## When to use

Use when building species- and tissue-stratified sequence benchmark sets from site-level modification calls, and redundancy must be removed before balanced train/test partitioning (e.g., CD-HIT <80% cutoff).

## When NOT to use

- Sequences are already deduplicated or you are reusing the published Table 3 benchmark splits — re-running CD-HIT and re-splitting would break comparability with reported results
- Your modification sites come from antibody-based (m6A-seq/MeRIP) peak data rather than the antibody-independent endoribonuclease detection data, since single-nucleotide resolution of the site center is required for 41-nt segment labeling
- The task is single-species, single-tissue prediction with no need for tissue stratification — a generic sequence benchmark suffices

## Inputs

- Antibody-independent m6A site calls per species-tissue (human, mouse, rat; brain, liver, kidney, heart, testis)
- Genome/transcriptome sequences for extracting 41-nt windows centered on candidate sites
- Positive/negative labels per candidate site (segment contains vs. lacks an m6A site)

## Outputs

- 11 deduplicated species-tissue FASTA datasets with sequence similarity <80% (CD-HIT filtered)
- Balanced training and testing subsets per tissue with equal positive/negative counts (Table 3 counts, e.g., m_b: 8,025/8,025/8,025/8,025)
- Species-tissue keyed dataset manifest (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l) for cross-validation and independent tests

## How to apply

Start from antibody-independent m6A site calls per species-tissue (e.g., Zhang et al.'s m6A-sensitive RNA endoribonuclease data) and extract 41-nt RNA segments centered on each candidate site, labeling segments containing an m6A site as positive and those without as negative. Run CD-HIT on each tissue dataset separately with a sequence similarity cutoff below 80% to eliminate redundant sequences — this prevents near-identical 41-mers from inflating performance across the train/test boundary. Partition each of the 11 filtered species-tissue datasets (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l) into balanced training and testing subsets with equal positive and negative counts, matching the per-dataset counts reported in Table 3 (e.g., h_b: 4,605/4,605 training positives/negatives and 4,604/4,604 testing positives/negatives). Keep the species-tissue identifiers explicit throughout so downstream cross-validation, independent testing, and cross-species/cross-tissue validation can be traced back to the same partitions.

## Related tools

- **CD-HIT** (Clustering tool applied to each tissue dataset to enforce a sequence similarity cutoff below 80%, removing redundant 41-nt segments before train/test partitioning)
- **im6A-TS-CNN** (Downstream CNN classifier whose 5-fold cross-validation and independent tests consume the curated, balanced species-tissue datasets) — https://github.com/liukeweiaway/DeepM6A_cnn

## Examples

```
cd-hit -i tissue_positives.fasta -o tissue_positives_cd80.fasta -c 0.8
```

## Evaluation signals

- Sequence similarity within each curated dataset is confirmed <80% by CD-HIT clustering
- Every species-tissue dataset yields exactly balanced positive/negative counts per Table 3 (e.g., h_b: 4,605/4,605 training and 4,604/4,604 testing; m_t: 4,707/4,707 training and 4,706/4,706 testing)
- All 11 datasets (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l) are present with counts matching Table 3 exactly
- Each extracted segment is 41 nt with the candidate m6A site at the center; positives contain the site, negatives do not
- Training and testing subsets are disjoint — no segment appears in both after the CD-HIT filter

## Limitations

- The 80% CD-HIT cutoff governs redundancy; alternative thresholds change dataset sizes and are not explored in the main text (threshold sensitivity deferred to supplementary Table S1 for the classifier, not the curation)
- Only 11 species-tissue combinations are covered; heart data exist only for human and mouse, and testis only for mouse, so cross-tissue coverage is asymmetric
- Dataset quality depends on the underlying antibody-independent m6A detection data (Zhang et al.); site-call errors propagate into positive labels

## Evidence
