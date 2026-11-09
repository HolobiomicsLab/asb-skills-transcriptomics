---
name: nucleotide-sequence-preprocessing
description: Use when raw 41-nt RNA sequences (FASTA or plain text) must be converted
  into fixed-length numeric feature matrices before feeding a CNN-based m6A site predictor
  such as im6A-TS-CNN.
when_to_use_negative:
- Input is already a numeric feature table or embedding (e.g., Gene2Vec-style features)
  rather than raw nucleotide sequences
- Sequences are not 41-nt RNA segments centered on candidate m6A sites — dimensionality
  checks (4 x 41 = 164) will fail or require re-trimming
- The downstream model expects a different encoding scheme (e.g., k-mer or embedding-based)
  specified elsewhere
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: im6A-TS-CNN
  role: Pipeline whose second step is one-hot encoding of 41-nt m6A sequences prior
    to CNN model construction
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: Python 3.6
  role: Implementation environment for the one-hot lookup and array concatenation
    (with numpy/pandas)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:7e3b506163c108a1e7d550f5a315269b158de23dc2040e16dbbc5581429f2fa0
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
n_claims: 10
---

# nucleotide-sequence-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes RNA nucleotide sequences into numeric feature vectors suitable for deep-learning models such as the im6A-TS-CNN, using the paper's 4-dimensional one-hot scheme (A/U/C/G) applied to 41-nt m6A candidate segments from human, mouse, and rat tissues.

## When to use

Use when raw 41-nt RNA sequences (FASTA or plain text) must be converted into fixed-length numeric feature matrices before feeding a CNN-based m6A site predictor such as im6A-TS-CNN.

## When NOT to use

- Input is already a numeric feature table or embedding (e.g., Gene2Vec-style features) rather than raw nucleotide sequences
- Sequences are not 41-nt RNA segments centered on candidate m6A sites — dimensionality checks (4 x 41 = 164) will fail or require re-trimming
- The downstream model expects a different encoding scheme (e.g., k-mer or embedding-based) specified elsewhere

## Inputs

- Curated 41-nt RNA sequences (positive m6A-containing and negative samples) from species-tissue benchmark datasets such as h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l
- Fasta-format input sequence file for prediction (e.g., input.txt per the im6A-TS-cnn README)

## Outputs

- One-hot encoded feature matrix of shape n_samples x 164 (4 x 41 per sequence)
- NumPy archive (e.g., one_hot_encoded_sequences.npz) containing encoded positive and negative feature matrices with labels

## How to apply

Load the curated 41-nt positive and negative RNA sequences from the benchmark dataset matching the species-tissue combination (e.g., h_b: 4,605 training positives and 4,605 negatives; also h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l). Map each nucleotide to its 4-dimensional one-hot vector using the paper's scheme: A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1), implemented as a simple Python lookup dict. Concatenate the per-nucleotide vectors so each 41-nt segment becomes a single 164-dimensional (4 x 41) feature vector; store encoded matrices and labels in a NumPy archive (e.g., one_hot_encoded_sequences.npz) for CNN construction in Keras/TensorFlow 2.0 under Python 3.6. This encoding is the second step of the im6A-TS-CNN pipeline, between tissue-specific data collection and model construction.

## Related tools

- **im6A-TS-CNN** (Pipeline whose second step is one-hot encoding of 41-nt m6A sequences prior to CNN model construction) — https://github.com/liukeweiaway/DeepM6A_cnn
- **Python 3.6** (Implementation environment for the one-hot lookup and array concatenation (with numpy/pandas))

## Examples

```
python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- Each encoded row has exactly 4 x 41 = 164 dimensions
- Encoded sample counts match the Table 3 row counts for the chosen dataset (e.g., 4,605 positives and 4,605 negatives for h_b)
- Decoding any encoded row with the inverse mapping recovers the original 41-nt nucleotide sequence exactly
- Only the four canonical RNA letters A, U, C, G appear in the input (any ambiguous base must be handled or rejected before encoding)
- Encoded positive/negative matrices are label-balanced as in the benchmark (positives = negatives per dataset)

## Limitations

- The scheme covers only A, U, C, G; ambiguous or modified bases in the FASTA input are not addressed by the paper's mapping
- Fixed 41-nt window: sequences of other lengths cannot be encoded without re-trimming or padding, which the paper does not describe
- CD-HIT filtering (<80% sequence similarity) was applied at dataset construction; encoding new sequences does not replicate that redundancy control

## Evidence
