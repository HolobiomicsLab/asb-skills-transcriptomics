---
name: one-hot-nucleotide-sequence-encoding
description: Use when raw nucleotide sequences (e.g., 41-nt RNA segments centered
  on a candidate m6A site) must be converted to fixed-dimensional numeric feature
  vectors before feeding a CNN classifier such as im6A-TS-CNN.
when_to_use_negative:
- Input is already a numeric feature matrix or embedding (e.g., Gene2Vec-style features)
  rather than raw nucleotide sequences
- The downstream model expects sequence-based encodings other than plain one-hot (e.g.,
  learned embeddings or k-mer features), since im6A-TS-CNN was designed and benchmarked
  specifically on one-hot encoding
- Sequences are not the canonical 41-nt m6A segments and contain ambiguity codes not
  covered by the A/U/C/G one-hot scheme
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0099
tools:
- name: im6A-TS-CNN
  role: Consumes the one-hot encoded 41-nt RNA segments as model input for tissue-specific
    m6A site identification
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: Keras in TensorFlow 2.0
  role: Framework in which the one-hot encoded inputs are fed to the CNN classifiers
- name: Python 3.6
  role: Runtime (with pandas 1.0.3, tensorflow-cpu 2.1.0, numpy 1.18.3) used to perform
    the encoding and prediction pipeline
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:18c38d4e2d43559fec5dd2daced36cf392b8019c1730b6685946108c2b204688
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
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# one-hot-nucleotide-sequence-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encode 41-nt RNA segments as fixed-length one-hot vectors so they can serve as numerical input to the im6A-TS-CNN tissue-specific m6A classifiers, with A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1).

## When to use

Use when raw nucleotide sequences (e.g., 41-nt RNA segments centered on a candidate m6A site) must be converted to fixed-dimensional numeric feature vectors before feeding a CNN classifier such as im6A-TS-CNN.

## When NOT to use

- Input is already a numeric feature matrix or embedding (e.g., Gene2Vec-style features) rather than raw nucleotide sequences
- The downstream model expects sequence-based encodings other than plain one-hot (e.g., learned embeddings or k-mer features), since im6A-TS-CNN was designed and benchmarked specifically on one-hot encoding
- Sequences are not the canonical 41-nt m6A segments and contain ambiguity codes not covered by the A/U/C/G one-hot scheme

## Inputs

- 41-nt positive and negative RNA sequence segments in FASTA/text form from tissue-specific m6A benchmark datasets (e.g., h_b: 4,605 training positives/negatives)
- Nucleotide alphabet A/U/C/G per position of each RNA segment

## Outputs

- One-hot encoded numeric matrices (164-dimensional vectors per 41-nt sample) suitable as Keras CNN input
- Encoded training and independent-test splits aligned with the 11 species-tissue datasets

## How to apply

Take each 41-nt RNA segment (positive or negative m6A example from the species-tissue benchmark datasets such as h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l) and convert each nucleotide to its one-hot vector: A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1). The encoded samples form the input matrix for the Keras/TensorFlow 2.0 CNN, whose first layers compress the 164-dimensional representation down through fully connected layers of 32 units to a softmax output. This encoding step is the second step of the three-step im6A-TS-CNN pipeline (data collection, encoding, model construction) and must be applied identically across all 11 tissue datasets so that trained models remain comparable and cross-species/cross-tissue transfer is possible. Before encoding, ensure datasets have passed CD-HIT similarity filtering (<80% sequence similarity) to avoid redundancy biasing the learned representations.

## Related tools

- **im6A-TS-CNN** (Consumes the one-hot encoded 41-nt RNA segments as model input for tissue-specific m6A site identification) — https://github.com/liukeweiaway/DeepM6A_cnn
- **Keras in TensorFlow 2.0** (Framework in which the one-hot encoded inputs are fed to the CNN classifiers)
- **Python 3.6** (Runtime (with pandas 1.0.3, tensorflow-cpu 2.1.0, numpy 1.18.3) used to perform the encoding and prediction pipeline)

## Examples

```
pip install pandas==1.0.3 tensorflow-cpu==2.1.0 numpy==1.18.3 && python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- Each encoded sample is exactly 164-dimensional for a 41-nt segment (41 positions x 4 channels), with exactly one hot channel per position
- Encoded matrices contain only the four defined vectors A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1); no other patterns appear
- Downstream CNN trained on encoded data reproduces reported performance (e.g., h_b AUC 0.8029 cross-validation / 0.8056 independent test), indicating encoding is consistent with the published pipeline
- Encoding is applied identically across all 11 species-tissue datasets so cross-validation and cross-species/cross-tissue results remain comparable

## Limitations

- The scheme covers only A/U/C/G; ambiguous nucleotides have no defined representation in the published encoding
- Datasets should be CD-HIT filtered to <80% sequence similarity before encoding; encoding redundant sequences can inflate performance
- The encoding is tied to the fixed 41-nt window length used by im6A-TS-CNN; other window sizes change the input dimensionality and invalidate the trained models

## Evidence
