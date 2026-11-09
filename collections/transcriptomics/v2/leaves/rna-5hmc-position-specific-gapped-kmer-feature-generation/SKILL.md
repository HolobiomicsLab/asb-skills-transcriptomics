---
name: rna-5hmc-position-specific-gapped-kmer-feature-generation
description: Use when raw RNA sequences of length 41 (5hmC benchmark Dataset.csv)
  must be transformed into position-specific gapped/ungapped k-mer descriptors, or
  when the stability of position-specific discriminative k-mer patterns must be verified
  across cross-validation partitions.
when_to_use_negative:
- Input is already a precomputed feature table (e.g., All_Features_Dataset.csv exists)
  and only model training/evaluation via Model.ipynb is needed
- Sequences are not 41-nt RNA fragments, since position-specific descriptors assume
  a fixed coordinate frame relative to the 41-nt sequence
- The goal is combining frequency-based k-mer features with position-specific features
  in a single model — the paper reports these do not work well together and defers
  such combination to a future feature-subspace ensemble approach
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: iRNA5hmC-PS
  role: Source framework providing the PSG k-mer feature generation pipeline, benchmark
    dataset, notebooks, and webserver
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: pandas
  role: Loading Dataset.csv / All_Features_Dataset.csv and performing set operations
    on selected feature sets
- name: numpy
  role: Array and intersection operations for the 25 feature-selection sets
- name: scikit-learn
  role: RandomForestClassifier used as mode-wise feature-importance selector (top
    400 per mode per partition)
- name: Jupyter
  role: Execution environment for Feature Generation.ipynb (Run All) inside Anaconda
- name: Anaconda
  role: Required environment in which Jupyter and the Python packages are installed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2020.10.032
    title: Accurate prediction of RNA 5-hydroxymethylcytosine modification by utilizing
      novel position-specific gapped k-mer descriptors
  source_content_hash: sha256:acdf3c85400fffbcb95fdc7d4cba12e46c322397af17847bcdbfa44d6efd4637
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zahid6454/iRNA5hmC-PS
  tools:
  - Anaconda
  - Jupyter
  - Python
  - iRNA5hmC-PS
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# rna-5hmc-position-specific-gapped-kmer-feature-generation

## Summary

Generates Position-Specific Gapped k-mer (PSG k-mer) and Position-Specific k-mer features from 41-nt RNA sequences via the Feature Generation pipeline of iRNA5hmC-PS, then validates the resulting features by intersecting RF-selected feature sets across 5-times 5-fold cross-validation partitions to recover the consistently discriminative position-specific patterns.

## When to use

Use when raw RNA sequences of length 41 (5hmC benchmark Dataset.csv) must be transformed into position-specific gapped/ungapped k-mer descriptors, or when the stability of position-specific discriminative k-mer patterns must be verified across cross-validation partitions.

## When NOT to use

- Input is already a precomputed feature table (e.g., All_Features_Dataset.csv exists) and only model training/evaluation via Model.ipynb is needed
- Sequences are not 41-nt RNA fragments, since position-specific descriptors assume a fixed coordinate frame relative to the 41-nt sequence
- The goal is combining frequency-based k-mer features with position-specific features in a single model — the paper reports these do not work well together and defers such combination to a future feature-subspace ensemble approach

## Inputs

- Dataset.csv — benchmark 5hmC RNA dataset with 41-nt sequences and labels
- Feature Generation.ipynb — Jupyter notebook implementing the PSG k-mer feature generation pipeline
- All_Features_Dataset.csv — generated feature matrix used by the mode-wise RF selectors
- 25 training partitions from 5-times 5-fold cross-validation

## Outputs

- All_Features_Dataset.csv — all proposed position-specific gapped k-mer features per sequence
- 25 sets of 400 mode-wise RF-selected features (one set per cross-validation partition)
- selected_features.csv — the 34 consensus features annotated with k-mer pattern and sequence position, plus descriptor-mode tally

## How to apply

Load the benchmark dataset (Dataset.csv) with pandas and run the Feature Generation.ipynb notebook in Jupyter/Anaconda (Run All) to compute all proposed position-specific gapped k-mer features (modes B/C/D) and position-specific k-mer features into All_Features_Dataset.csv. For each of the 25 training partitions from 5-times 5-fold cross-validation, train 4 mode-wise scikit-learn RandomForestClassifier selectors and retain the top 400 features per selector, producing 25 sets of 400 features. Intersect these 25 sets with numpy/pandas set operations and verify that exactly 34 features survive in every partition. Annotate the 34 surviving features with their k-mer pattern and sequence position relative to the 41-nt sequence (e.g., A-A-A at positions 22-26, G-G-G at positions 36-40), tally descriptor-mode membership to confirm 27 of 34 come from the gapped modes B/C/D, and save the annotated table as selected_features.csv. The rationale is that group-wise feature importance calculation isolates a small feature subset carrying maximum discriminative information for 5hmC site identification.

## Related tools

- **iRNA5hmC-PS** (Source framework providing the PSG k-mer feature generation pipeline, benchmark dataset, notebooks, and webserver) — https://github.com/zahid6454/iRNA5hmC-PS
- **pandas** (Loading Dataset.csv / All_Features_Dataset.csv and performing set operations on selected feature sets)
- **numpy** (Array and intersection operations for the 25 feature-selection sets)
- **scikit-learn** (RandomForestClassifier used as mode-wise feature-importance selector (top 400 per mode per partition))
- **Jupyter** (Execution environment for Feature Generation.ipynb (Run All) inside Anaconda)
- **Anaconda** (Required environment in which Jupyter and the Python packages are installed)

## Examples

```
jupyter nbconvert --to notebook --execute "Feature Generation.ipynb" --inplace  # regenerates All_Features_Dataset.csv from Dataset.csv
```

## Evaluation signals

- Feature Generation.ipynb must produce All_Features_Dataset.csv after a full Run All execution
- Each of the 25 partition-wise selectors must retain exactly 400 features per mode-wise RF selector
- The intersection across all 25 selection sets must contain exactly 34 features
- The descriptor-mode tally must show 27 of the 34 consensus features originate from the gapped modes B/C/D
- Surviving features must be annotatable with concrete k-mer patterns and positions (e.g., A-A-A at positions 22-26, G-G-G at positions 36-40) on the 41-nt frame

## Limitations

- Position-specific features do not work well in conjunction with existing frequency-based counterparts in single models; the authors defer combination to a future feature subspace ensemble approach
- The webserver at http://103.109.52.8:81/iRNA5hmC-PS requires FASTA input with sequences of length 41
- The pattern/position validation depends on the fixed 41-nt sequence frame; the intersected 34-feature result is specific to this benchmark dataset and 5-times 5-fold partition scheme

## Evidence

- [methods] Feature Generation notebook execution: ""Feature Generation.ipynb" uses "Dataset.csv" file to generate all the proposed features and stores that into a new file titled as "All_Features_Dataset.csv""
- [abstract] Group-wise feature importance calculation for feature selection: "We also use a group-wise feature importance calculation strategy to select a small subset of features containing maximum discriminative information"
- [abstract] PSG k-mer information content: "Our feature analysis shows that our proposed PSG k-mer features contain vital information for the identification of 5hmC sites"
- [readme] README execution procedure: "Execute the "Feature Generation.ipynb" from Jupyter clicking "Cell" dropdown button on top and click "Run All" button."
- [discussion] Frequency/position-specific combination caveat: "our proposed position-specific features do not work well in conjunction with existing frequency-based counterparts"
