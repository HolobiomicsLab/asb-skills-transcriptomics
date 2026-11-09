---
name: fasta-sequence-file-curation
description: Use when assembling tissue-specific m6A benchmark FASTA datasets — 41-nt
  RNA segments from human/mouse/rat brain, liver, kidney, heart, or testis — need
  redundancy removal (CD-HIT <80%) and balanced train/test splits before model training.
when_to_use_negative:
- The input FASTA is already a deduplicated benchmark matching the Table 3 counts
  — re-running CD-HIT could discard valid sequences without benefit.
- The goal is m6A prediction on new sequences rather than benchmark construction —
  invoke im6A-TS-CNN directly with its -s/-t/-ts options instead.
- Segments are not 41-nt windows centered on candidate sites, or come from species/tissues
  outside the human/mouse/rat brain, liver, kidney, heart, and testis scope of the
  source m6A data.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3474
tools:
- name: CD-HIT
  role: Sequence-similarity clustering used to enforce the <80% identity cutoff within
    each of the 11 species-tissue FASTA datasets, eliminating redundant 41-nt segments
- name: im6A-TS-CNN
  role: Downstream CNN classifier that consumes the curated, balanced FASTA benchmark
    datasets (after one-hot encoding) to identify m6A sites per tissue and species
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:9ab20c7b97601dc53ecd68c25d85200efc528ea5d01ebceb0565aa259782526c
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
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# fasta-sequence-file-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curation of tissue-specific m6A benchmark FASTA collections: extracting 41-nt RNA segments centered on candidate sites from human, mouse, and rat tissues, labeling site-containing segments as positive, and removing redundancy with CD-HIT at <80% sequence similarity before CNN training.

## When to use

Use when assembling tissue-specific m6A benchmark FASTA datasets — 41-nt RNA segments from human/mouse/rat brain, liver, kidney, heart, or testis — need redundancy removal (CD-HIT <80%) and balanced train/test splits before model training.

## When NOT to use

- The input FASTA is already a deduplicated benchmark matching the Table 3 counts — re-running CD-HIT could discard valid sequences without benefit.
- The goal is m6A prediction on new sequences rather than benchmark construction — invoke im6A-TS-CNN directly with its -s/-t/-ts options instead.
- Segments are not 41-nt windows centered on candidate sites, or come from species/tissues outside the human/mouse/rat brain, liver, kidney, heart, and testis scope of the source m6A data.

## Inputs

- Tissue-specific m6A site calls from antibody-independent m6A-sensitive RNA endoribonuclease detection data (Zhang et al.) for human, mouse, and rat
- FASTA files of 41-nt RNA segments centered on candidate m6A sites, separated into positive (site present) and negative (site absent) classes per species-tissue combination

## Outputs

- 11 deduplicated species-tissue benchmark FASTA datasets (e.g., h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k) with sequence similarity <80%
- Balanced training and testing subsets per tissue with equal positive/negative counts (Table 3, e.g., h_b: 4,605/4,605 training and 4,604/4,604 testing)
- One-hot encoded sample matrices (A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1)) feeding im6A-TS-CNN training and testing

## How to apply

Collect antibody-independent m6A site calls (Zhang et al.'s m6A-sensitive RNA endoribonuclease detection data) covering brain, liver, kidney, heart, and testis of Homo sapiens, Mus musculus, and Rattus norvegicus, then cut 41-nt RNA segments centered on each candidate site and store them in FASTA format. Label segments containing an m6A site as positive and segments without as negative, keeping the two classes separate per species-tissue combination (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, ...). Run the CD-HIT program on each of the 11 tissue datasets with a sequence-similarity cutoff below 80% so that no redundant near-identical 41-nt segments remain — the rationale is that unfiltered homologs inflate cross-validation performance. Partition each filtered tissue dataset into balanced training and testing subsets with equal positive and negative counts per Table 3 (e.g., h_b: 4,605/4,605 training positives/negatives and 4,604/4,604 testing positives/negatives). The curated FASTA files are then one-hot encoded (A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1)) as input to im6A-TS-CNN, implemented in Keras in TensorFlow 2.0 under Python 3.6.

## Related tools

- **CD-HIT** (Sequence-similarity clustering used to enforce the <80% identity cutoff within each of the 11 species-tissue FASTA datasets, eliminating redundant 41-nt segments)
- **im6A-TS-CNN** (Downstream CNN classifier that consumes the curated, balanced FASTA benchmark datasets (after one-hot encoding) to identify m6A sites per tissue and species) — https://github.com/liukeweiaway/DeepM6A_cnn

## Examples

```
python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- Sequence counts of all 11 curated datasets exactly match the positive/negative training and testing counts reported in Table 3 (e.g., h_b: 4,605/4,605 training and 4,604/4,604 testing).
- CD-HIT clustering of each filtered dataset confirms no sequence pair retains ≥80% similarity.
- Every retained segment is a 41-nt window centered on the candidate site, and each split is class-balanced (equal positives and negatives).
- Each curated FASTA record converts losslessly to one-hot 4-tuples (A/U/C/G) with no ambiguous residues.
- Downstream 5-fold cross-validation of im6A-TS-CNN on the curated sets reports AUC > 0.8 for most tissues (e.g., r_k AUC 0.9017), consistent with the published results.

## Limitations

- Only species-tissue combinations with available antibody-independent m6A endoribonuclease data can be curated; per-species tissue coverage is uneven (e.g., human has brain, liver, kidney; mouse also has heart and testis).
- The <80% CD-HIT cutoff trades redundancy removal against discarding near-threshold but distinct sequences, so curated dataset sizes differ per tissue (1,762 to 8,025 pairs in Table 3).
- Strict class balancing makes some training/testing splits asymmetric (e.g., h_b: 4,605 training vs 4,604 testing pairs), which must be respected when reproducing Table 3.

## Evidence
