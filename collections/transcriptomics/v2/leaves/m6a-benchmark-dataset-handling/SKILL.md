---
name: m6a-benchmark-dataset-handling
description: Use when reconstructing or reproducing the 11 species-tissue m6A benchmark
  datasets (e.g., h_b, h_k, m_t, r_k) from Zhang et al. antibody-independent m6A calls,
  or when preparing 41-nt RNA segments for CNN training/testing.
when_to_use_negative:
- Input is a complete FASTA sequence awaiting prediction — use the im6A-TS-cnn command-line
  tool instead of rebuilding benchmark datasets
- Sequences cannot be reduced below 80% similarity, or you need tissue-unbiased (multi-tissue)
  m6A models rather than the 11 species-tissue datasets
- Sequences are already encoded as numeric feature vectors (e.g., from Gene2Vec or
  DeepM6ASeq-style pipelines)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: CD-HIT
  role: Deduplication of each of the 11 tissue-specific datasets to enforce <80% sequence
    similarity before partitioning
- name: Python 3.6
  role: Runtime for one-hot encoding lookup dicts and NumPy .npz storage, and for
    running the Keras/TensorFlow 2.0 predictions
- name: im6A-TS-CNN
  role: Downstream CNN consumer of the encoded benchmark datasets; command-line version
    released at the project repository
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:d0fe5e715b81d22497ebc806fcf2362aa3cb935246b604ffe1cd44849bf47d03
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
  - Python 3.6
  - im6A-TS-CNN
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# m6a-benchmark-dataset-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curates tissue-specific m6A benchmark datasets (41-nt segments centered on candidate sites) and encodes them for the im6A-TS-CNN CNN pipeline: CD-HIT deduplication below 80% sequence similarity, then one-hot nucleotide encoding into 164-dimensional (4 x 41) feature vectors stored as NumPy .npz archives.

## When to use

Use when reconstructing or reproducing the 11 species-tissue m6A benchmark datasets (e.g., h_b, h_k, m_t, r_k) from Zhang et al. antibody-independent m6A calls, or when preparing 41-nt RNA segments for CNN training/testing.

## When NOT to use

- Input is a complete FASTA sequence awaiting prediction — use the im6A-TS-cnn command-line tool instead of rebuilding benchmark datasets
- Sequences cannot be reduced below 80% similarity, or you need tissue-unbiased (multi-tissue) m6A models rather than the 11 species-tissue datasets
- Sequences are already encoded as numeric feature vectors (e.g., from Gene2Vec or DeepM6ASeq-style pipelines)

## Inputs

- Antibody-independent m6A site calls from Zhang et al.'s m6A-sensitive RNA endoribonuclease detection data (brain, liver, kidney, heart, testis of human, mouse, rat)
- Curated 41-nt positive and negative RNA sequence segments per species-tissue combination (FASTA or text format)
- Python 3.6 environment with numpy, pandas, and tensorflow (e.g., tensorflow-cpu 2.1.0, numpy 1.18.3)

## Outputs

- CD-HIT-filtered, redundancy-reduced benchmark datasets for 11 species-tissue combinations with Table 3-matched counts
- Balanced training and testing subsets per tissue (equal positive/negative counts)
- one_hot_encoded_sequences.npz containing 164-dimensional one-hot feature matrices and class labels

## How to apply

Extract 41-nt RNA segments centered on candidate m6A sites from Zhang et al.'s antibody-independent m6A-sensitive RNA endoribonuclease data, labeling segments containing an m6A site as positive and those without as negative. Run CD-HIT on each of the 11 tissue datasets with a sequence similarity cutoff below 80% to remove redundancy. Partition each filtered dataset into balanced training and testing subsets matching Table 3 counts (e.g., h_b: 4,605/4,605 training positives/negatives and 4,604/4,604 testing positives/negatives). Then apply the one-hot scheme — A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1) — concatenating per-nucleotide vectors so each 41-nt segment becomes a 164-dimensional feature vector, and store positive/negative matrices with labels in one_hot_encoded_sequences.npz for downstream CNN training in Keras/TensorFlow 2.0 under Python 3.6.

## Related tools

- **CD-HIT** (Deduplication of each of the 11 tissue-specific datasets to enforce <80% sequence similarity before partitioning)
- **Python 3.6** (Runtime for one-hot encoding lookup dicts and NumPy .npz storage, and for running the Keras/TensorFlow 2.0 predictions)
- **im6A-TS-CNN** (Downstream CNN consumer of the encoded benchmark datasets; command-line version released at the project repository) — https://github.com/liukeweiaway/DeepM6A_cnn

## Examples

```
enc={'A':(1,0,0,0),'U':(0,1,0,0),'C':(0,0,1,0),'G':(0,0,0,1)}; X=[sum([list(enc[c]) for c in seq],[]) for seq in seqs]; np.savez('one_hot_encoded_sequences.npz', X=X)
```

## Evaluation signals

- Sequence counts of all 11 curated datasets exactly match the positive/negative training and testing counts in Table 3 (e.g., h_k: 2,634/2,634/2,634/2,634)
- CD-HIT run confirms sequence similarity within each dataset is less than 80%
- Every encoded row has exactly 4 x 41 = 164 dimensions
- Decoding any one-hot row recovers the original 41-nt nucleotide sequence
- Training and testing subsets are balanced (equal positive and negative counts per tissue)

## Limitations

- The curation and one-hot scheme are tied to 41-nt segments and four canonical nucleotides (A, U, C, G); modifications outside this scheme are not handled
- Benchmark coverage is limited to the 11 species-tissue combinations (e.g., mouse and rat testis datasets are not among the curated Table 3 entries listed here)
- Threshold performance with different options is deferred to supplementary Table S1 and not given in the main text

## Evidence
