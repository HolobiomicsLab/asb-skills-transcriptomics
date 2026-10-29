---
name: cnn-architecture-construction-for-rna-classification
description: Use when a tissue-specific RNA m6A classifier must be built from 41-nt
  one-hot encoded sequences (164-dimensional vectors) in Keras/TensorFlow 2.0, e.g.,
  reproducing im6A-TS-CNN model construction for one of the 11 species-tissue benchmarks.
when_to_use_negative:
- The task is only to run the already-trained im6A-TS-CNN predictor on new FASTA input
  (choosing species, tissue, and low/normal/high threshold) rather than reconstructing
  or retraining the CNN architecture itself.
- Input segments are not 41-nt RNA sequences from the 11 human/mouse/rat tissue m6A
  benchmarks; characters outside A/U/C/G break the 4-symbol one-hot encoding and fixed
  164-dimensional input shape.
- The target modification or organism falls outside the benchmarked scope (m6A in
  brain, liver, kidney, heart, testis of Homo sapiens, Mus musculus, Rattus norvegicus);
  the trained artifacts and reported metrics do not transfer automatically.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: im6A-TS-CNN
  role: The CNN classifier being constructed and trained per tissue dataset; its command-line
    version packages the trained models for prediction with -i/-o/-s/-t/-ts options.
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: Keras in TensorFlow 2.0
  role: Framework used to define, train (SGD lr 0.001, categorical cross-entropy,
    early stopping patience 50), and save the 200-filter/kernel-6/max-pool-4/dropout-0.16
    CNN with 164-to-32 fully connected compression and softmax output.
- name: Python 3.6
  role: Runtime environment for the Keras/TensorFlow implementation, pinned with pandas==1.0.3,
    tensorflow-cpu==2.1.0, numpy==1.18.3.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:aa18f785b72684102c4108bbc185dc1e3257af952038d3a8b18deccf0c912098
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cnn-architecture-construction-for-rna-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs and trains the tissue-specific convolutional neural network classifiers of im6A-TS-CNN in Keras on TensorFlow 2.0 under Python 3.6, converting 41-nt RNA segments into 164-dimensional one-hot vectors to identify N6-methyladenine sites across human, mouse, and rat tissues.

## When to use

Use when a tissue-specific RNA m6A classifier must be built from 41-nt one-hot encoded sequences (164-dimensional vectors) in Keras/TensorFlow 2.0, e.g., reproducing im6A-TS-CNN model construction for one of the 11 species-tissue benchmarks.

## When NOT to use

- The task is only to run the already-trained im6A-TS-CNN predictor on new FASTA input (choosing species, tissue, and low/normal/high threshold) rather than reconstructing or retraining the CNN architecture itself.
- Input segments are not 41-nt RNA sequences from the 11 human/mouse/rat tissue m6A benchmarks; characters outside A/U/C/G break the 4-symbol one-hot encoding and fixed 164-dimensional input shape.
- The target modification or organism falls outside the benchmarked scope (m6A in brain, liver, kidney, heart, testis of Homo sapiens, Mus musculus, Rattus norvegicus); the trained artifacts and reported metrics do not transfer automatically.

## Inputs

- FASTA/sequence file of 41-nt positive and negative m6A RNA segments for one species-tissue benchmark (e.g., h_b: 4,605 training positives and 4,605 training negatives)
- One-hot encoded 164-dimensional sample vectors per RNA segment (A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1))
- Species-tissue configuration choice: Human/Mouse/Rat combined with brain, liver, kidney, heart, or testis

## Outputs

- Trained Keras (TensorFlow 2.0) CNN model artifact saved per species-tissue dataset
- 5-fold cross-validation and independent-test metrics (Sn, Sp, Acc, MCC, AUC), e.g., h_b AUC 0.8029 (CV) and 0.8056 (independent test)
- Per-sequence m6A prediction scores for new FASTA input, written to an output.html results file via the command-line tool

## How to apply

Load one of the 11 species-tissue m6A benchmark sets (e.g., h_b: 4,605 training positives and 4,605 negatives, 41-nt sequences) and encode each RNA segment as a 164-dimensional one-hot vector (A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1)). In Keras on TensorFlow 2.0 (Python 3.6), build the CNN as a convolutional layer with 200 filters of kernel size 6 with ReLU activation, a max-pooling layer of size 4, dropout rate 0.16, then fully connected layers compressing 164 to 32 units, ending in a softmax output. Compile with the stochastic gradient descent optimizer (learning rate 0.001) and categorical cross-entropy loss, and train up to 2,000 epochs with early stopping (patience 50, min_delta 0.001), using 5-fold cross-validation on the training split while holding the independent test set out. Save the trained Keras model artifact per tissue dataset, repeat for all combinations (Human brain/liver/kidney/heart/testis, Mouse brain/liver/kidney, Rat brain/liver/kidney), and accept a model when it reproduces the reported tissue-level performance on the independent test set (e.g., h_b: Sn 75.35%, Sp 69.71%, Acc 72.53%, MCC 0.4523, AUC 0.8029) with most tissue AUCs exceeding 0.8.

## Related tools

- **im6A-TS-CNN** (The CNN classifier being constructed and trained per tissue dataset; its command-line version packages the trained models for prediction with -i/-o/-s/-t/-ts options.) — https://github.com/liukeweiaway/DeepM6A_cnn
- **Keras in TensorFlow 2.0** (Framework used to define, train (SGD lr 0.001, categorical cross-entropy, early stopping patience 50), and save the 200-filter/kernel-6/max-pool-4/dropout-0.16 CNN with 164-to-32 fully connected compression and softmax output.)
- **Python 3.6** (Runtime environment for the Keras/TensorFlow implementation, pinned with pandas==1.0.3, tensorflow-cpu==2.1.0, numpy==1.18.3.)

## Examples

```
python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- Every encoded sample is exactly 164-dimensional (41 nt × 4 one-hot channels); any other dimensionality indicates the encoding step is wrong.
- 5-fold cross-validation and independent-test metrics for h_b reproduce the reported Sn 75.35%/75.17%, Sp 69.71%/70.20%, Acc 72.53%/72.69%, MCC 0.4523/0.4543, AUC 0.8029/0.8056.
- Most of the 11 tissue AUCs exceed 0.8 in both CV and independent testing, with r_k highest (AUC 0.9017 CV, 0.9077 independent); large deviations flag a faulty reimplementation.
- Training terminates via early stopping (patience 50, min_delta 0.001) within the 2,000-epoch cap, using the SGD optimizer at learning rate 0.001 with categorical cross-entropy loss.
- A saved Keras model artifact exists for each of the 11 species-tissue combinations (Human brain/liver/kidney/heart/testis, Mouse brain/liver/kidney, Rat brain/liver/kidney).

## Limitations

- im6A-TS-CNN is not uniformly superior: iRNA-m6A had slightly higher AUC for mouse brain in cross-validation (0.8731 vs 0.8705) and rat brain in the independent test (0.8968 vs 0.8516).
- The environment is version-pinned (Python 3.6, pandas==1.0.3, tensorflow-cpu==2.1.0, numpy==1.18.3) and the README documents only V0.1.2 (updated 2020-05-03).
- Performance with the three threshold options (low/normal/high) is deferred to supplementary Table S1 and not detailed in the main text.

## Evidence

- [results] Keras/TensorFlow implementation: "the Keras in TensorFlow 2.0 under Python 3.6 was used to perform the predictions"
- [methods] One-hot encoding scheme: "in an RNA segment, A is represented as (1,0,0,0), U as (0,1,0,0), C as (0,0,1,0), and G as (0,0,0,1)"
- [results] Expected performance range: [evidence span withheld — anchor status: not_found]
- [methods] h_b benchmark counts: "h_b	4,605	4,605	4,604	4,604"
- [readme] Pinned environment: "pip install pandas==1.0.3 tensorflow-cpu==2.1.0 numpy==1.18.3"
- [readme] Tool scope: "im6A-TS-cnn is able to identify multi-tissues in Human, Mouse and Rat."
