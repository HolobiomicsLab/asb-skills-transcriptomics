---
name: rna-sequence-one-hot-encoding
description: Use when raw RNA sequence segments (e.g., 41-nt tissue-specific m6A positive/negative
  samples) must be converted into fixed-length numeric feature matrices before feeding
  a CNN or other fixed-input neural network model.
when_to_use_negative:
- Input is already a numeric feature matrix or embedding (e.g., Gene2Vec-style vectors)
  rather than raw nucleotide sequences
- Sequences are not the fixed 41-nt m6A segments used by im6A-TS-CNN, since the 4
  x 41 = 164-dimension assumption no longer holds without re-padding or re-trimming
- A sequence-aware model (e.g., RNN/transformer) that can consume raw nucleotide tokens
  directly is chosen, making one-hot flattening unnecessary
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0099
tools:
- name: im6A-TS-CNN
  role: Downstream CNN model whose input layer (164 units) fixes the required one-hot
    encoded dimensions; encoding is the second step of its pipeline
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: Python 3.6
  role: Implementation language for the nucleotide-to-vector lookup dict and NumPy
    storage of encoded matrices
- name: Keras in TensorFlow 2.0
  role: Framework that consumes the encoded 164-d vectors for 5-fold cross-validation
    and independent testing of the m6A classifier
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:3048df0a5f269e0d9da3c22ecc1c3858e72289cba392a1032f76350394c37fac
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
  - iRNA-m6A
  - im6A-TS-CNN
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# rna-sequence-one-hot-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes RNA nucleotide sequences as 4-dimensional one-hot vectors (A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1)) so each 41-nt m6A segment becomes a 164-dimensional feature vector suitable for CNN-based m6A site identification as used in im6A-TS-CNN.

## When to use

Use when raw RNA sequence segments (e.g., 41-nt tissue-specific m6A positive/negative samples) must be converted into fixed-length numeric feature matrices before feeding a CNN or other fixed-input neural network model.

## When NOT to use

- Input is already a numeric feature matrix or embedding (e.g., Gene2Vec-style vectors) rather than raw nucleotide sequences
- Sequences are not the fixed 41-nt m6A segments used by im6A-TS-CNN, since the 4 x 41 = 164-dimension assumption no longer holds without re-padding or re-trimming
- A sequence-aware model (e.g., RNN/transformer) that can consume raw nucleotide tokens directly is chosen, making one-hot flattening unnecessary

## Inputs

- 41-nt positive and negative RNA sequence segments (FASTA or plain text) from tissue-specific m6A benchmark datasets (e.g., h_b: 4,605 training positives / 4,605 training negatives)
- Species-tissue dataset selector (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l)

## Outputs

- 164-dimensional (4 x 41) one-hot feature vectors per sequence, stored as NumPy arrays (e.g., one_hot_encoded_sequences.npz) with class labels

## How to apply

Load the curated 41-nt positive and negative RNA sequences for the species-tissue dataset of interest (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l) built by Dao et al. and CD-HIT filtered to <80% similarity. Map each nucleotide through a Python lookup dict with A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1). Concatenate per-nucleotide vectors so each 41-nt segment becomes a 4 x 41 = 164-dimensional vector, matching the first fully connected layer of 164 units in the im6A-TS-CNN architecture (200 filters, kernel size 6, max-pooling 4, dropout 0.16). Store the encoded positive and negative matrices with labels (e.g., one_hot_encoded_sequences.npz) for 5-fold cross-validation and independent testing in Keras/TensorFlow 2.0 under Python 3.6.

## Related tools

- **im6A-TS-CNN** (Downstream CNN model whose input layer (164 units) fixes the required one-hot encoded dimensions; encoding is the second step of its pipeline) — https://github.com/liukeweiaway/DeepM6A_cnn
- **Python 3.6** (Implementation language for the nucleotide-to-vector lookup dict and NumPy storage of encoded matrices)
- **Keras in TensorFlow 2.0** (Framework that consumes the encoded 164-d vectors for 5-fold cross-validation and independent testing of the m6A classifier)

## Examples

```
python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- Every encoded row has exactly 4 x 41 = 164 dimensions, matching the CNN's first fully connected layer of 164 units
- Encoded sample counts match the Table 3 row counts for the chosen dataset (e.g., 4,605 positives and 4,605 negatives for h_b)
- Decoding any encoded row with the same lookup dict recovers the original nucleotide sequence exactly
- Alphabet check: only A, U, C, G nucleotides are present; any other character fails the mapping
- Downstream sanity: trained model AUCs on most of the 11 tissue datasets exceed 0.8 in both 5-fold cross-validation and independent test (e.g., r_k AUC 0.9017 / 0.9077)

## Limitations

- The scheme assumes exactly four nucleotides (A, U, C, G); ambiguous bases or N characters are not handled by the specified lookup dict
- Encoding dimension is tied to the 41-nt segment length; other sequence lengths change the 164-d vector size and break compatibility with the published model
- One-hot encoding discards positional independence information beyond sequence position; the paper reports iRNA-m6A (SVM) still outperforms on mouse and rat brain in AUC

## Evidence
