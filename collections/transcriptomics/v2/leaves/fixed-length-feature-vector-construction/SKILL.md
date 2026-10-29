---
name: fixed-length-feature-vector-construction
description: Use when raw nucleotide sequences (e.g., 41-nt m6A positive/negative
  samples from tissue-specific benchmark datasets like h_b, m_k, r_l) must be encoded
  as fixed-length numeric matrices before feeding a deep-learning model.
when_to_use_negative:
- Input is already a numeric feature table or embedding (e.g., Gene2Vec-style vectors)
  rather than raw nucleotide sequences
- Sequences are not composed solely of A/U/C/G (e.g., contain N or lowercase ambiguous
  codes) and no handling decision has been made
- The downstream model expects a different sequence representation (e.g., k-mer or
  SVM-based features as in iRNA-m6A) rather than one-hot CNN input
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: im6A-TS-CNN
  role: The encoding scheme is defined as the second step of the im6A-TS-CNN pipeline,
    producing one-hot inputs for its CNN model construction
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: Python 3.6
  role: Implementation language for the lookup-dict encoding and NumPy storage of
    the encoded matrices
- name: Keras in TensorFlow 2.0
  role: 'Downstream consumer: the encoded 164-dimensional vectors feed CNN training/prediction
    in Keras under TensorFlow 2.0'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:d8535b7b6d52d020ce42bd75809caf6a165248c13df9243c1001457c146d380a
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
  - Keras in TensorFlow 2.0
  - Python 3.6
  - im6A-TS-CNN
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# fixed-length-feature-vector-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts variable-text RNA sequences into fixed-length numeric feature vectors via the one-hot scheme used by im6A-TS-CNN: each nucleotide A/U/C/G becomes a 4-dimensional binary vector, so a 41-nt m6A benchmark segment yields a 4 x 41 = 164-dimensional input suitable for CNN training.

## When to use

Use when raw nucleotide sequences (e.g., 41-nt m6A positive/negative samples from tissue-specific benchmark datasets like h_b, m_k, r_l) must be encoded as fixed-length numeric matrices before feeding a deep-learning model.

## When NOT to use

- Input is already a numeric feature table or embedding (e.g., Gene2Vec-style vectors) rather than raw nucleotide sequences
- Sequences are not composed solely of A/U/C/G (e.g., contain N or lowercase ambiguous codes) and no handling decision has been made
- The downstream model expects a different sequence representation (e.g., k-mer or SVM-based features as in iRNA-m6A) rather than one-hot CNN input

## Inputs

- 41-nt positive RNA sequences (m6A-containing) from a tissue-specific benchmark dataset (e.g., Table 3 datasets h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l)
- 41-nt negative RNA sequences (non-m6A) matched to the same species-tissue combination
- Nucleotide-to-vector lookup scheme (A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1))

## Outputs

- Encoded positive feature matrix of shape (n_positives, 164)
- Encoded negative feature matrix of shape (n_negatives, 164)
- Label vector(s) for the encoded samples
- NumPy archive file (e.g., one_hot_encoded_sequences.npz) holding features and labels for CNN training

## How to apply

Load the curated 41-nt positive and negative RNA sequences for the target species-tissue dataset (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l). Map each nucleotide to its 4-dimensional one-hot vector with a Python lookup dict: A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1), per the im6A-TS-CNN encoding scheme. Concatenate the per-nucleotide vectors so each 41-nt segment becomes a single 164-dimensional (4 x 41) feature vector. Store the encoded positive and negative feature matrices with labels (e.g., in a NumPy archive one_hot_encoded_sequences.npz) for downstream CNN construction in Keras/TensorFlow 2.0 under Python 3.6. This encoding is the second step of the im6A-TS-CNN pipeline, between tissue-specific m6A data collection and model construction.

## Related tools

- **im6A-TS-CNN** (The encoding scheme is defined as the second step of the im6A-TS-CNN pipeline, producing one-hot inputs for its CNN model construction) — https://github.com/liukeweiaway/DeepM6A_cnn
- **Python 3.6** (Implementation language for the lookup-dict encoding and NumPy storage of the encoded matrices)
- **Keras in TensorFlow 2.0** (Downstream consumer: the encoded 164-dimensional vectors feed CNN training/prediction in Keras under TensorFlow 2.0)

## Examples

```
map = {'A':(1,0,0,0),'U':(0,1,0,0),'C':(0,0,1,0),'G':(0,0,0,1)}; X = np.array([np.concatenate([map[c] for c in seq]) for seq in seqs]); np.savez('one_hot_encoded_sequences.npz', X=X, y=labels)
```

## Evaluation signals

- Every encoded row has exactly 4 x 41 = 164 dimensions
- Encoded sample counts match the Table 3 row counts for the chosen dataset (e.g., 4,605 training positives and 4,605 training negatives for h_b; 1,762 for r_k)
- Decoding any encoded row via argmax of each 4-vector block recovers the original nucleotide sequence exactly
- Only the four allowed binary patterns (1,0,0,0), (0,1,0,0), (0,0,1,0), (0,0,0,1) appear; no all-zero or multi-hot blocks
- Positive and negative sets are encoded with the identical scheme and stored together with labels for downstream training

## Limitations

- The scheme covers only A, U, C, and G; nucleotides outside this alphabet have no defined encoding
- Fixed 41-nt segment length is assumed; sequences of other lengths produce vectors of different dimensionality (4 x length) and are not directly compatible
- The CD-HIT preprocessing (sequence similarity < 80%) must have been applied to the source dataset before encoding, otherwise redundancy inflates the encoded set

## Evidence
