---
name: binary-indicator-vector-construction
description: Use when converting fixed-length nucleotide sequences (e.g., 41-nt RNA
  5hmC windows) into numerical feature vectors via position-specific binary occurrence
  indicators of k-mer and gapped k-mer patterns before classifier training.
when_to_use_negative:
- Input sequences have variable length or are not 41-nt windows — the position-specific
  indicators assume fixed consecutive-residue-group positions
- Input is already a feature table (e.g., All_Features_Dataset.csv); proceed directly
  to Model.ipynb training/evaluation instead
- You need frequency-based gapped k-mer features rather than position-specific binary
  indicators — the paper shows position-specific features do not combine well with
  frequency-based counterparts in single models
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iRNA5hmC-PS
  role: Prediction framework whose feature extraction stage is implemented by this
    skill; PSG k-mer and Ps k-mer descriptors are its novel components
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: pandas
  role: Loads Dataset.csv and concatenates the pattern blocks into All_Features_Dataset.csv
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: numpy
  role: Vectorized construction of the binary position-specific indicator arrays for
    each pattern mode
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: Jupyter
  role: Execution environment for the Feature Generation.ipynb notebook that runs
    this encoding
  repo: https://github.com/zahid6454/iRNA5hmC-PS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2020.10.032
    title: Accurate prediction of RNA 5-hydroxymethylcytosine modification by utilizing
      novel position-specific gapped k-mer descriptors
  source_content_hash: sha256:493b09b940f538ab22d39becad4208bb4600849b46799941c619a57a564fed16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zahid6454/iRNA5hmC-PS
  tools:
  - Jupyter
  - Python
  - iRNA5hmC-PS
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# binary-indicator-vector-construction

## Summary

Encodes fixed-length (41-nt) RNA sequences into position-specific binary indicator vectors by testing each nucleotide pattern (contiguous or gapped k-mer) against all consecutive-residue-groups along the sequence, then concatenates the pattern blocks with a GC-content feature into a single 19,877-column feature matrix.

## When to use

Use when converting fixed-length nucleotide sequences (e.g., 41-nt RNA 5hmC windows) into numerical feature vectors via position-specific binary occurrence indicators of k-mer and gapped k-mer patterns before classifier training.

## When NOT to use

- Input sequences have variable length or are not 41-nt windows — the position-specific indicators assume fixed consecutive-residue-group positions
- Input is already a feature table (e.g., All_Features_Dataset.csv); proceed directly to Model.ipynb training/evaluation instead
- You need frequency-based gapped k-mer features rather than position-specific binary indicators — the paper shows position-specific features do not combine well with frequency-based counterparts in single models

## Inputs

- Dataset.csv containing 41-nt RNA sequences in FASTA-like tabular form
- Pattern definitions for the six modes: PsM(G)D, PsD(G)M, PsM(G)M(G)M (gapped) and PsM, PsD, PsT (contiguous)

## Outputs

- All_Features_Dataset.csv — a 19,877-column per-sequence feature matrix (7104 + 7104 + 2304 + 164 + 640 + 2496 binary indicators plus GC-content)
- Feature count validation report per pattern type against Table 2

## How to apply

Load the 41-nt RNA sequences from Dataset.csv with pandas, then build each pattern mode as a binary position-specific indicator array using vectorized numpy operations: for each pattern, slide it across all consecutive-residue-groups of the sequence and set the indicator to 1 wherever the pattern matches at that position. Compute the three PSG k-mer modes PsM(G)D, PsD(G)M, and PsM(G)M(G)M (7104, 7104, and 2304 binary features respectively) and the three position-specific k-mer modes PsM, PsD, and PsT (164, 640, and 2496 features respectively). Add a per-sequence GC-content feature, then concatenate all blocks into a 19,877-column matrix and save it as All_Features_Dataset.csv. Validate per-block feature counts against Table 2 before finalizing, since the downstream LR/SVM/GNB models in Model.ipynb depend on this exact feature layout.

## Related tools

- **iRNA5hmC-PS** (Prediction framework whose feature extraction stage is implemented by this skill; PSG k-mer and Ps k-mer descriptors are its novel components) — https://github.com/zahid6454/iRNA5hmC-PS
- **pandas** (Loads Dataset.csv and concatenates the pattern blocks into All_Features_Dataset.csv) — https://github.com/zahid6454/iRNA5hmC-PS
- **numpy** (Vectorized construction of the binary position-specific indicator arrays for each pattern mode) — https://github.com/zahid6454/iRNA5hmC-PS
- **Jupyter** (Execution environment for the Feature Generation.ipynb notebook that runs this encoding) — https://github.com/zahid6454/iRNA5hmC-PS

## Examples

```
jupyter nbconvert --to notebook --execute "Feature Generation.ipynb"  # reads Dataset.csv, writes All_Features_Dataset.csv (19,877 features per 41-nt sequence)
```

## Evaluation signals

- Per-block feature counts must match Table 2: PsM(G)D = 7104, PsD(G)M = 7104, PsM(G)M(G)M = 2304, PsM = 164, PsD = 640, PsT = 2496
- Total concatenated feature count must equal 19,877 columns per sequence (plus GC-content handling as specified)
- All indicator values must be binary (0/1) after encoding; GC-content lies in [0, 1]
- All_Features_Dataset.csv is produced in the working folder after running Feature Generation.ipynb with Run All, matching the README's stated working procedure
- Row count of All_Features_Dataset.csv must equal the sequence count in Dataset.csv (one feature vector per 41-nt sequence)

## Limitations

- Features are designed for 41-nt RNA sequences; other window lengths require re-deriving position layouts
- Position-specific features do not work well in conjunction with existing frequency-based counterparts in single models; the authors defer frequency+position subspace-ensemble combination to future work
- The encoding was motivated by classification of 5hmC sites; its suitability for deep learning architectures (convolutional/recurrent layers) is claimed but not demonstrated in this work

## Evidence

- [results] position-specific binary occurrence indicators yield 7104, 7104, 2304, 164, 640, 2496 features; total 19,877: [evidence span withheld — anchor status: not_found]
- [methods] Feature Generation notebook pipeline: ""Feature Generation.ipynb" uses "Dataset.csv" file to generate all the proposed features and stores that into a new file titled as "All_Features_Dataset.csv""
- [discussion] Feature combination caveat: "our proposed position-specific features do not work well in conjunction with existing frequency-based counterparts"
- [discussion] Deep learning suitability caveat: "Therefore, our features are suitable for deep learning architectures consisting of convolutional and recurrent layers that exploit relative ordering of the residues"
- [readme] Notebook execution procedure: "Execute the "Feature Generation.ipynb" from Jupyter clicking "Cell" dropdown button on top and click "Run All" button."
