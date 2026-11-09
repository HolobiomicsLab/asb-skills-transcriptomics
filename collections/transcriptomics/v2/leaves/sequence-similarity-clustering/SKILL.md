---
name: sequence-similarity-clustering
description: Use when curating RNA/DNA sequence benchmark sets for ML training and
  testing where redundant sequences could cause overfitting, and a similarity cutoff
  (e.g. <80%) must be enforced per dataset.
when_to_use_negative:
- Input sequences are already known to be non-redundant or were previously CD-HIT-processed
  at the target cutoff — re-clustering wastes time and may over-prune.
- The downstream task tolerates high sequence redundancy (e.g. per-read variant calling
  on native transcripts) rather than benchmark curation for a supervised classifier.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3308
tools:
- name: CD-HIT
  role: Clustering/deduplication program run on each tissue dataset to enforce <80%
    sequence similarity
- name: im6A-TS-CNN
  role: Downstream CNN classifier whose benchmark datasets (human, mouse, rat; brain/liver/kidney/heart/testis)
    are deduplicated by this filtering step
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:74500f366df2dae006a1cfdd4268396c752fe38381e8278a1b511d6c8f83d64a
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# sequence-similarity-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Deduplicate nucleotide sequence benchmark datasets by clustering sequences with CD-HIT so that pairwise sequence similarity stays below a specified cutoff (here <80%), preventing redundant near-identical sequences from inflating classifier performance in tissue-specific m6A prediction.

## When to use

Use when curating RNA/DNA sequence benchmark sets for ML training and testing where redundant sequences could cause overfitting, and a similarity cutoff (e.g. <80%) must be enforced per dataset.

## When NOT to use

- Input sequences are already known to be non-redundant or were previously CD-HIT-processed at the target cutoff — re-clustering wastes time and may over-prune.
- The downstream task tolerates high sequence redundancy (e.g. per-read variant calling on native transcripts) rather than benchmark curation for a supervised classifier.

## Inputs

- FASTA-format sequence segment files (41-nt RNA segments centered on candidate m6A sites) for each species-tissue dataset
- Per-dataset positive (m6A-containing) and negative (m6A-free) sequence labels
- Sequence identity cutoff parameter (0.8 / <80%)

## Outputs

- Redundancy-filtered FASTA datasets per species-tissue combination with no pairwise sequence similarity ≥80%
- Balanced training/testing partitions with counts matching Table 3 (e.g. h_b: 4,605 train pos/neg, 4,604 test pos/neg; r_k: 1,762/1,762/1,762/1,762)

## How to apply

After extracting fixed-length sequence segments (in im6A-TS-CNN, 41-nt RNA segments centered on candidate m6A sites, labeled positive if the site contains m6A and negative otherwise), run CD-HIT independently on each dataset (each of the 11 species-tissue datasets: h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, plus one more) with a sequence similarity threshold of 0.8 so no pair of sequences exceeds 80% similarity. Rationale: near-identical sequences split across training and testing partitions would let the CNN memorize rather than learn tissue-specific methylation patterns. After filtering, partition each filtered dataset into balanced training and testing subsets with equal positive and negative counts (e.g. h_b: 4,605/4,605 training positives/negatives and 4,604/4,604 testing positives/negatives per Table 3). Apply the clustering per dataset, not globally, to preserve tissue-specific composition.

## Related tools

- **CD-HIT** (Clustering/deduplication program run on each tissue dataset to enforce <80% sequence similarity)
- **im6A-TS-CNN** (Downstream CNN classifier whose benchmark datasets (human, mouse, rat; brain/liver/kidney/heart/testis) are deduplicated by this filtering step) — https://github.com/liukeweiaway/DeepM6A_cnn

## Examples

```
cd-hit -i h_b_segments.fasta -o h_b_cdhit80.fasta -c 0.8
```

## Evaluation signals

- Sequence counts of all 11 curated species-tissue datasets exactly match the positive/negative training and testing counts reported in Table 3 of the paper (e.g. h_b: 4,605/4,605/4,604/4,604).
- No pair of retained sequences within a dataset exceeds 80% identity under CD-HIT's clustering definition.
- Each filtered dataset remains balanced: equal positive and negative counts in training and in testing subsets.
- Training and testing counts differ by at most 1 per class (e.g. 4,605 train vs 4,604 test for h_b), consistent with an 80/20 or similar balanced split after deduplication.

## Limitations

- The 80% similarity cutoff is dataset- and purpose-specific; other tasks may require stricter (e.g. <50% for protein remote homology) or looser cutoffs.
- CD-HIT clustering is greedy and approximate — clusters depend on sequence order and the representative-selection heuristic, so results may vary slightly with input ordering.
- The paper curates 11 species-tissue datasets only (brain, liver, kidney, heart, testis in human/mouse/rat as available); applying the cutoff to other tissues/species requires re-verifying balance, and m_l counts (2,201 train pos/neg, 2,200 test pos/neg) show some positives are lost to deduplication.

## Evidence
