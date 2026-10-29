---
name: bag-level-instance-label-relabeling
description: Use when candidate m6A site labels are uncertain and inputs are positive/negative
  bag-level datasets, so instance labels must be inferred via multiple instance learning.
when_to_use_negative:
- Input is already instance-level labeled data where supervised learning is possible
  rather than a multiple-instance bag problem
- Bag labels are 0/1-valued rather than +1/-1 — the misvm package notes you will likely
  get strange results with 0/1 labels
- You need bag-level-only classifiers (e.g. NSK) where the instancePrediction flag
  is unavailable and per-instance relabeling via instance predictions is not supported
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: misvm
  role: Python multiple-instance SVM library that supplies the miSVM/MISVM classifiers
    used by m6A_model_training.py to train bag-level models on the m6A candidate bags
  repo: https://github.com/garydoranjr/misvm
- name: HLMethy
  role: Repository providing m6A_model_training.py and m6A_model_prediction.py, which
    train the bag-level miSVM/MISVM model and generate scores to relabel candidate
    m6A sites
  repo: https://github.com/NWAFU-LiuLab/HLMethy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11103-019-00930-x
    title: 'HLMethy: a machine learning-based model to identify the hidden labels
      of m6A candidates'
  source_content_hash: sha256:4eb4ad4c91e11169ff8d2adf4f43261869e6e26dee23b4b74cb5e92858d2a62b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/garydoranjr/misvm
  tools:
  - HLMethy
  - misvm
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# bag-level-instance-label-relabeling

## Summary

Train a bag-level miSVM/MISVM model with HLMethy's m6A_model_training.py (via the misvm package) to generate scores that relabel hidden-label candidate m6A sites, addressing the partially-labeled positive-bag problem of multiple instance learning.

## When to use

Use when candidate m6A site labels are uncertain and inputs are positive/negative bag-level datasets, so instance labels must be inferred via multiple instance learning.

## When NOT to use

- Input is already instance-level labeled data where supervised learning is possible rather than a multiple-instance bag problem
- Bag labels are 0/1-valued rather than +1/-1 — the misvm package notes you will likely get strange results with 0/1 labels
- You need bag-level-only classifiers (e.g. NSK) where the instancePrediction flag is unavailable and per-instance relabeling via instance predictions is not supported

## Inputs

- positive bag-level dataset text file (e.g. traindata/train_pos_example.txt)
- negative bag-level dataset text file (e.g. traindata/train_neg_example.txt)
- instance-selection method argument: 0 (miSVM) or 1 (MISVM)

## Outputs

- pickled SVM model file (svc.pkl)
- pickled scaler/normalized file (scale.pkl)
- prediction score file (score.txt)
- reserved samples file (reserved.txt)
- feature encoding file (train_features.txt)
- bag index file (bag_index)

## How to apply

Install the misvm package from github.com/garydoranjr/misvm (it depends on numpy, scipy, and cvxopt). Run m6A_model_training.py with positive_dataset, negative_dataset, an instance-selection method argument (0 = miSVM, 1 = MISVM, both sanctioned by the script), and output filenames for the model, scaler, scores, and reserved samples. The script trains a bag-level model on the example bags, then a scoring step (m6A_model_prediction.py with the trained svc.pkl and scale.pkl) generates scores used to relabel the candidate m6A sites. The choice between method 0 (miSVM) and method 1 (MISVM) is the key decision point: the README example uses method 1, but the script is explicitly documented to train either a miSVM or MISVM bag-level model.

## Related tools

- **misvm** (Python multiple-instance SVM library that supplies the miSVM/MISVM classifiers used by m6A_model_training.py to train bag-level models on the m6A candidate bags) — https://github.com/garydoranjr/misvm
- **HLMethy** (Repository providing m6A_model_training.py and m6A_model_prediction.py, which train the bag-level miSVM/MISVM model and generate scores to relabel candidate m6A sites) — https://github.com/NWAFU-LiuLab/HLMethy

## Examples

```
python m6A_model_training.py ./traindata/train_pos_example.txt ./traindata/train_neg_example.txt 1 svc.pkl scale.pkl score.txt reserved_samples.txt
```

## Evaluation signals

- All six named artifacts are emitted and non-empty: svc.pkl, scale.pkl, score.txt, reserved.txt, train_features.txt, and bag_index
- The script runs to completion without error, producing the same artifact set as the sanctioned method-1 (MISVM) README example run
- m6A_model_prediction.py successfully loads svc.pkl and scale.pkl and writes prediction scores for the test dataset (e.g. test_example.txt to predict_res)
- Trained labels must be +1/-1 bag labels as required by misvm's fit function; 0/1 labels indicate a misuse
- The method argument passed is strictly 0 (miSVM) or 1 (MISVM) as documented in the script usage

## Limitations

- misvm classifiers must receive bag labels of +1 or -1; 0/1-valued labels give strange results
- The instancePrediction flag is not available for bag-level classifiers such as NSK; interpreting per-instance labels from bag-level classifiers requires caution (e.g. via singleton bags)
- The misvm code depends on numpy, scipy, and cvxopt being installed first or the build will likely fail
- Only methods 0 (miSVM) and 1 (MISVM) are documented selection options for m6A_model_training.py

## Evidence

- [readme] This script is used to train a bag level miSVM/MISVM model.: "This script is used to train a bag level miSVM/MISVM model."
- [readme] method of selection: 0 -- miSVM, 1 -- MISVM: "method : method of selection

        0 -- miSVM
        
        1 -- MISVM"
- [readme] This script will generate scores to relabel the candidate m6A sites.: "This script will generate scores to relabel the candidate m6A sites."
- [intro] The misvm package should be installed at first, which can be downloaded from https://github.com/garydoranjr/misvm.: "The misvm package should be installed at first, which can be downloaded from https://github.com/garydoranjr/misvm."
- [abstract] Relabeling candidate m6As with multiple instance learning.: "**Relabeling candidate m6As with multiple instance learning.**"
