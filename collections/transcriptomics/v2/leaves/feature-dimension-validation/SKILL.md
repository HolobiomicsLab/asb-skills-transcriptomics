---
name: feature-dimension-validation
description: Use when you have generated a position-specific (gapped) k-mer feature
  matrix from fixed-length 41-nt RNA sequences and must verify per-pattern-type feature
  counts and the total matrix width before model training.
when_to_use_negative:
- Input is already a completed feature table such as All_Features_Dataset.csv rather
  than raw 41-nt sequences in Dataset.csv
- Sequences are not a fixed 41 nt in length — the position-specific binary indicators
  assume a fixed window and count budget
- You need frequency-based k-mer descriptors (e.g., the 84 k-mer frequency features
  or prior gapped k-mer frequency features) instead of position-specific occurrence
  indicators
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: pandas
  role: Loads Dataset.csv, concatenates feature blocks, and writes All_Features_Dataset.csv
    for dimension checks
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: numpy
  role: Vectorized construction of binary position-specific occurrence indicator arrays
    for the PSG k-mer and Ps k-mer blocks
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: Jupyter
  role: Runs Feature Generation.ipynb end-to-end via Run All to produce the feature
    matrix that is dimension-validated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2020.10.032
    title: Accurate prediction of RNA 5-hydroxymethylcytosine modification by utilizing
      novel position-specific gapped k-mer descriptors
  source_content_hash: sha256:aa6d368ce582328769ae17daea9c2e0c33250e759d1a30c84423c2da795bf1fc
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# feature-dimension-validation

## Summary

Validates that position-specific gapped k-mer (PSG k-mer) feature extraction for iRNA5hmC-PS produces the correct number of features per 41-nt RNA sequence — 7104 (PsM(G)D) + 7104 (PsD(G)M) + 2304 (PsM(G)M(G)M) + 164 (PsM) + 640 (PsD) + 2496 (PsT) + GC-content = 19,877 columns in All_Features_Dataset.csv.

## When to use

Use when you have generated a position-specific (gapped) k-mer feature matrix from fixed-length 41-nt RNA sequences and must verify per-pattern-type feature counts and the total matrix width before model training.

## When NOT to use

- Input is already a completed feature table such as All_Features_Dataset.csv rather than raw 41-nt sequences in Dataset.csv
- Sequences are not a fixed 41 nt in length — the position-specific binary indicators assume a fixed window and count budget
- You need frequency-based k-mer descriptors (e.g., the 84 k-mer frequency features or prior gapped k-mer frequency features) instead of position-specific occurrence indicators

## Inputs

- Dataset.csv containing 41-nt RNA sequences (benchmark dataset from the iRNA5hmC-PS GitHub repo)
- Per-pattern feature block column counts expected from Table 2: 7104, 7104, 2304 (PSG k-mer) and 164, 640, 2496 (Ps k-mer) plus GC-content

## Outputs

- All_Features_Dataset.csv with 19,877 feature columns per sequence, validated against Table 2
- Per-pattern-type dimension report (e.g., PsM(G)D=7104, PsD(G)M=7104, PsM(G)M(G)M=2304, PsM=164, PsD=640, PsT=2496, GC=1)

## How to apply

Load Dataset.csv (41-nt RNA sequences) with pandas, run the Feature Generation notebook to build the six pattern blocks — PSG k-mer modes PsM(G)D, PsD(G)M, and PsM(G)M(G)M as binary position-specific occurrence indicators (7104, 7104, and 2304 features), position-specific modes PsM, PsD, and PsT (164, 640, and 2496 binary indicators), plus one GC-content feature per sequence — and concatenate them with pandas into All_Features_Dataset.csv. Validate each block's column count and the total of 19,877 features per sequence against Table 2 before finalizing the file, then proceed to Model.ipynb for LR/SVM/GNB training with 5-fold cross-validation.

## Related tools

- **pandas** (Loads Dataset.csv, concatenates feature blocks, and writes All_Features_Dataset.csv for dimension checks) — https://github.com/zahid6454/iRNA5hmC-PS
- **numpy** (Vectorized construction of binary position-specific occurrence indicator arrays for the PSG k-mer and Ps k-mer blocks) — https://github.com/zahid6454/iRNA5hmC-PS
- **Jupyter** (Runs Feature Generation.ipynb end-to-end via Run All to produce the feature matrix that is dimension-validated)

## Examples

```
import pandas as pd; df = pd.read_csv('All_Features_Dataset.csv'); assert df.shape[1] == 19877, df.shape[1]
```

## Evaluation signals

- Column counts per pattern block match Table 2 exactly: PsM(G)D=7104, PsD(G)M=7104, PsM(G)M(G)M=2304, PsM=164, PsD=640, PsT=2496, plus one GC-content column
- Total feature width of All_Features_Dataset.csv equals 19,877 columns per sequence (19,876 binary indicators + 1 GC-content feature)
- All non-GC feature values are binary (0/1) position-specific occurrence indicators
- Number of rows in All_Features_Dataset.csv equals the number of 41-nt sequences in Dataset.csv
- Downstream Model.ipynb executes without shape errors, and LR cross-validation reproduces metrics near the reported 78.3% accuracy and 0.56 MCC

## Limitations

- Position-specific features do not work well in conjunction with existing frequency-based counterparts in single models; the authors defer combined frequency/position-specific ensembles to future work
- The feature budget is tied to the fixed 41-nt sequence length used in this benchmark; other sequence lengths change the counts
- The 19,877-dimension matrix is sparse and high-dimensional, making the validation meaningful only before group-wise feature selection is applied

## Evidence

- [methods] Feature counts per pattern type are fixed by the 41-nt window: ""Feature Generation.ipynb" uses "Dataset.csv" file to generate all the proposed features and stores that into a new file titled as "All_Features_Dataset.csv""
- [results] Total dimensionality of the validated matrix: [evidence span withheld — anchor status: not_found]
- [readme] Execution path that produces the validated output file: "Execute the "Feature Generation.ipynb" from Jupyter clicking "Cell" dropdown button on top and click "Run All" button."
- [readme] Validation checkpoint before model training: "After the execution of the "Feature Generation.ipynb", a new file titled "All_Features_Dataset.csv" will be avaiable in the folder."
