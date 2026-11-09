---
name: image-like-feature-matrix-construction
description: Use when fixed-length protein or RNA sequences must be turned into 2D
  numeric matrices that preserve positional dependencies among residues/nucleotides
  before feeding a CNN classifier.
when_to_use_negative:
- Sequences are still variable-length and have not been converted to fixed-length
  by the copy-padding trick (the encoding presumes fixed-length inputs)
- Input is already a numeric feature matrix or a precomputed feature table — re-encoding
  from raw sequences is unnecessary
- The downstream model is not a CNN expecting image-like 2D inputs (e.g., sequence-kernel
  or alignment-based methods)
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: LPI_feature.py
  role: Script used to generate the feature coding matrices via high-order one-hot
    encoding
  repo: github.com/NWPU-903PR/LPI-CNNCP
- name: Python 3
  role: Runtime environment in which the encoding is executed (Linux, per README system
    requirements)
- name: numpy
  role: Numerical library (version 1.17.4) used to build and save the coding matrices
    as .npz arrays
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ab.2020.113767
    title: 'LPI-CNNCP: Prediction of lncRNA-protein interactions by using convolutional
      neural network with the copy-padding trick'
  source_content_hash: sha256:04e0f4600ba9cf2c0a82a345e4adc3579443b486e66250226827477436f0d486
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LPI_feature.py
  - Python 3
  - numpy
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# image-like-feature-matrix-construction

## Summary

Converts fixed-length protein/RNA sequences into image-like, non-independent numeric inputs via high-order one-hot encoding, so a CNN (e.g., LPI-CNNCP) can model dependencies among amino acids or nucleotides for lncRNA-protein interaction prediction.

## When to use

Use when fixed-length protein or RNA sequences must be turned into 2D numeric matrices that preserve positional dependencies among residues/nucleotides before feeding a CNN classifier.

## When NOT to use

- Sequences are still variable-length and have not been converted to fixed-length by the copy-padding trick (the encoding presumes fixed-length inputs)
- Input is already a numeric feature matrix or a precomputed feature table — re-encoding from raw sequences is unnecessary
- The downstream model is not a CNN expecting image-like 2D inputs (e.g., sequence-kernel or alignment-based methods)

## Inputs

- data.zip: training, testing, and independent testing dataset with sequences, pair names, and labels
- Sequence pairs in the naming format 'label$protein_tag$lncRNA_tag$protein_sequence#lncRNA_sequence' (e.g., 1$3UZK-3$3UZK-A$MAHKKGLGSTRNG#GGUCAAGAUGGUA)
- Subsequence-variant pairs 'label$protein_tag$lncRNA_tag_(subsequence index)$protein_subsequence#lncRNA_subsequence' for sequences exceeding the fixed length range

## Outputs

- feature_coding_matrices.npz: numpy archive of high-order one-hot encoded, image-like coding matrices per sequence pair, keyed by pair name
- Per-sequence coding matrices with dimensions consistent with the fixed-length CNN input requirements

## How to apply

First ensure each sequence pair is already at a fixed length (LPI-CNNCP applies its copy-padding trick beforehand to turn variable-length sequences into fixed-length ones). Parse the pair naming format 'label$protein_tag$lncRNA_tag$protein_sequence#lncRNA_sequence' (or the '_(subsequence index)' variant for sequences exceeding the fixed-length range) from data.zip, splitting each record into its protein and lncRNA components. Run ./LPI_feature.py (reproducible in Python 3 with numpy 1.17.4) to apply the high-order one-hot encoding, which transforms each sequence into an image-like non-independent input capturing dependencies among amino acids (or nucleotides) rather than treating positions as independent. Stack the resulting per-sequence coding matrices into numpy arrays keyed by pair name and save them as feature_coding_matrices.npz; validate that every pair in data.zip yields an encoded matrix entry with dimensions consistent with the fixed-length CNN input requirements.

## Related tools

- **LPI_feature.py** (Script used to generate the feature coding matrices via high-order one-hot encoding) — github.com/NWPU-903PR/LPI-CNNCP
- **Python 3** (Runtime environment in which the encoding is executed (Linux, per README system requirements))
- **numpy** (Numerical library (version 1.17.4) used to build and save the coding matrices as .npz arrays)

## Examples

```
python LPI_feature.py
```

## Evaluation signals

- LPI_feature.py executes without error (script_runs signal from the source card)
- Every pair in data.zip yields an encoded matrix entry (file_exists, field_present signals)
- Each coding matrix has dimensions consistent with the fixed-length CNN input requirements
- Output archive feature_coding_matrices.npz is keyed by pair name matching the 'label$protein_tag$lncRNA_tag$...' naming format
- Encoded inputs are non-independent (image-like) rather than independent per-position one-hot vectors, per the paper's stated encoding design

## Limitations

- Requires sequences to be at a fixed length first; variable-length sequences must be pre-processed with the copy-padding trick or split into '(subsequence index)' subsequences
- Environment-specific: documented against Linux, Python 3, Keras 2.2.4/TensorFlow backend, scikit-learn 0.21.3, numpy 1.17.4; other versions may not reproduce results
- Repository documentation provides no changelog, limiting version traceability

## Evidence

- [intro] High-order one-hot encoding captures dependencies: "the high-order one-hot encoding way is used to transform the protein/RNA sequences into image-like inputs of non-independent for capturing the dependencies among amino acids (or nucleotides)."
- [readme] LPI_feature.py generates coding matrices: "./LPI_feature.py: the python code, it can be used to generate feature coding matrices."
- [readme] data.zip dataset description: "./data.zip: the training, testing and independent testing dataset with sequence, pair name and label."
- [readme] Pair naming format: "label+'$'+protein_tag+'$'+lncRNA_tag+'$'+protein_sequence+'#'+lncRNA_sequence."
- [intro] Copy-padding precedes encoding for fixed length: "LPI-CNNCP designs a copy-padding trick to turn the protein/RNA sequences with variable-length into the fixed-length sequences, letting them meet the input requirements of CNN model."
