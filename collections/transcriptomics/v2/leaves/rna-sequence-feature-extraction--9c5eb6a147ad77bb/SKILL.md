---
name: rna-sequence-feature-extraction
description: Use when raw RNA sequences (fixed length, e.g. 41 nt FASTA or CSV) must
  be converted into position-specific k-mer/gapped k-mer binary feature vectors for
  ML-based modification-site prediction.
when_to_use_negative:
- Input is already a feature table (e.g. All_Features_Dataset.csv exists) — proceed
  directly to model training/feature selection instead
- Sequences vary in length or exceed 41 nt without re-defining the position-specific
  matching window, since the descriptor counts depend on the 41-nt coordinate grid
- The goal is frequency-based k-mer features or combination with existing frequency-based
  descriptors — the authors found position-specific features do not work well in conjunction
  with frequency-based counterparts
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: iRNA5hmC-PS
  role: Framework providing the PSG k-mer and position-specific k-mer descriptor definitions
    implemented by this skill
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: Python
  role: Implementation language for the Feature Generation.ipynb notebook
- name: pandas
  role: Loading Dataset.csv and concatenating/saving the 19,877-column feature matrix
    to All_Features_Dataset.csv
- name: numpy
  role: Vectorized construction of the binary position-specific occurrence-indicator
    arrays
- name: Jupyter
  role: Execution environment for the Feature Generation.ipynb notebook (Cell > Run
    All)
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2020.10.032
    title: Accurate prediction of RNA 5-hydroxymethylcytosine modification by utilizing
      novel position-specific gapped k-mer descriptors
  source_content_hash: sha256:be5102dcea7a1a92f3cd3ec53fa6e6e40bbedc085f31b654f8659313ebef97fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zahid6454/iRNA5hmC-PS
  tools:
  - Jupyter
  - Python
  - iRNA5hmC-PS
  - matplotlib
  - numpy
  - pandas
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# rna-sequence-feature-extraction

## Summary

Encodes 41-nt RNA sequences into position-specific gapped k-mer (PSG k-mer), position-specific k-mer, and GC-content features — 19,877 numerical features per sequence in total — as used by iRNA5hmC-PS for 5-hydroxymethylcytosine (5hmC) site prediction.

## When to use

Use when raw RNA sequences (fixed length, e.g. 41 nt FASTA or CSV) must be converted into position-specific k-mer/gapped k-mer binary feature vectors for ML-based modification-site prediction.

## When NOT to use

- Input is already a feature table (e.g. All_Features_Dataset.csv exists) — proceed directly to model training/feature selection instead
- Sequences vary in length or exceed 41 nt without re-defining the position-specific matching window, since the descriptor counts depend on the 41-nt coordinate grid
- The goal is frequency-based k-mer features or combination with existing frequency-based descriptors — the authors found position-specific features do not work well in conjunction with frequency-based counterparts

## Inputs

- Dataset.csv — 41-nt RNA sequences (benchmark 5hmC dataset from https://github.com/zahid6454/iRNA5hmC-PS)
- Optional: FASTA-formatted 41-nt RNA sequences as accepted by the iRNA5hmC-PS webserver

## Outputs

- All_Features_Dataset.csv — 19,877-column feature matrix (PsM(G)D 7104 + PsD(G)M 7104 + PsM(G)M(G)M 2304 + PsM 164 + PsD 640 + PsT 2496 + 1 GC-content feature per sequence)

## How to apply

Load the 41-nt RNA sequences from Dataset.csv with pandas. Compute the three PSG k-mer modes — PsM(G)D (7104 features), PsD(G)M (7104 features), and PsM(G)M(G)M (2304 features) — as binary position-specific occurrence indicators by matching each nucleotide pattern (contiguous or gapped k-mer) against all consecutive-residue groups along the sequence, using numpy arrays for vectorized construction. Compute the position-specific (Ps k-mer) modes PsM (164), PsD (640), and PsT (2496) binary indicators, plus a GC-content feature. Concatenate all feature blocks into a single 19,877-column matrix with pandas and save it as All_Features_Dataset.csv. The rationale is that position-specific binary indicators retrieve crucial sequential information that k-mer frequencies and one-hot encoding miss, improving class separation between 5hmC and non-5hmC sites.

## Related tools

- **iRNA5hmC-PS** (Framework providing the PSG k-mer and position-specific k-mer descriptor definitions implemented by this skill) — https://github.com/zahid6454/iRNA5hmC-PS
- **Python** (Implementation language for the Feature Generation.ipynb notebook)
- **pandas** (Loading Dataset.csv and concatenating/saving the 19,877-column feature matrix to All_Features_Dataset.csv)
- **numpy** (Vectorized construction of the binary position-specific occurrence-indicator arrays)
- **Jupyter** (Execution environment for the Feature Generation.ipynb notebook (Cell > Run All))

## Examples

```
jupyter nbconvert --to notebook --execute "Feature Generation.ipynb"  # reads Dataset.csv, writes All_Features_Dataset.csv (19,877 features)
```

## Evaluation signals

- Feature counts per pattern block match Table 2: PsM(G)D = 7104, PsD(G)M = 7104, PsM(G)M(G)M = 2304, PsM = 164, PsD = 640, PsT = 2496
- Total feature matrix has exactly 19,877 feature columns per sequence (19,876 k-mer indicators + 1 GC-content feature)
- Every indicator column is binary (0/1 occurrence at a specific position group along the 41-nt sequence)
- All_Features_Dataset.csv is produced after running Feature Generation.ipynb, matching the README's stated output file
- Downstream check: a Logistic Regression on the selected features under 5-fold cross-validation should approximate the reported 78.3% accuracy / 0.56 MCC on the benchmark dataset

## Limitations

- Descriptor dimensions are tied to the 41-nt sequence convention; other lengths require rederiving the position-group enumeration and feature counts
- The proposed position-specific features do not work well in conjunction with existing frequency-based counterparts; combining them is deferred to future work using a feature subspace ensemble approach
- Efficacy on other RNA modification site identification problems has not been evaluated

## Evidence

- [readme] Feature Generation uses Dataset.csv to generate all the proposed features and stores that into a new file titled as All_Features_Dataset.csv: ""Feature Generation.ipynb" uses "Dataset.csv" file to generate all the proposed features and stores that into a new file titled as "All_Features_Dataset.csv""
- [abstract] Each nucleotide pattern is matched position-specifically along the sequence as binary indicators: "we extract a set of novel sequence-based features called Position-Specific Gapped k-mer (PSG k-mer) to obtain maximum sequential information"
- [intro] The rationale is that richer sequence-based feature extraction improves class separation: "we started with the hypothesis that exploring more sequence-based feature extraction approaches would increase the possibility of retrieving crucial information, relevant for better class-separation,"
- [methods] Concrete execution procedure for the feature generation step: "Execute the "Feature Generation.ipynb" from Jupyter clicking "Cell" dropdown button on top and click "Run All" button."
- [results] Downstream performance validates the encoding: "LR achieves 78.3%, 78.0%, 80.0%, 79.5%, 0.78, 0.56 in terms of Accuracy, Precision, Sensitivity, Specificity, F1-Score, and MCC, respectively"
