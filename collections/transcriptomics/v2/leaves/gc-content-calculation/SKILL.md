---
name: gc-content-calculation
description: Use when building position-specific k-mer feature vectors from fixed-length
  (41-nt) RNA sequences for 5hmC site prediction and a per-sequence GC-content descriptor
  is required as an additional feature block.
when_to_use_negative:
- Input sequences vary in length — GC content as implemented here assumes the fixed
  41-nt sequences used by iRNA5hmC-PS
- You already have a feature table (e.g., All_Features_Dataset.csv) — GC content is
  then already computed and recomputing it duplicates a block
- You want to combine GC content with frequency-based k-mer descriptors in a single
  model — the authors report position-specific features do not work well with existing
  frequency-based counterparts and defer this to future work
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: pandas
  role: Loads Dataset.csv and concatenates the GC-content column into the final All_Features_Dataset.csv
    feature matrix
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: numpy
  role: Vectorized computation of the per-sequence GC-content values alongside the
    binary indicator feature blocks
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: iRNA5hmC-PS
  role: Parent framework whose Feature Generation.ipynb notebook computes GC content
    as one feature block among the position-specific k-mer descriptors
  repo: https://github.com/zahid6454/iRNA5hmC-PS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2020.10.032
    title: Accurate prediction of RNA 5-hydroxymethylcytosine modification by utilizing
      novel position-specific gapped k-mer descriptors
  source_content_hash: sha256:ae37cc02d6d9f63e65c585d454a6ef041694dbfaa90d0783eb1b2d6a5e0aff97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zahid6454/iRNA5hmC-PS
  tools:
  - Python
  - iRNA5hmC-PS
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# gc-content-calculation

## Summary

Computes the GC-content feature for each 41-nt RNA sequence as one component of the iRNA5hmC-PS feature matrix, which combines six position-specific k-mer blocks (19,877 features total) plus this scalar descriptor for RNA 5-hydroxymethylcytosine site prediction.

## When to use

Use when building position-specific k-mer feature vectors from fixed-length (41-nt) RNA sequences for 5hmC site prediction and a per-sequence GC-content descriptor is required as an additional feature block.

## When NOT to use

- Input sequences vary in length — GC content as implemented here assumes the fixed 41-nt sequences used by iRNA5hmC-PS
- You already have a feature table (e.g., All_Features_Dataset.csv) — GC content is then already computed and recomputing it duplicates a block
- You want to combine GC content with frequency-based k-mer descriptors in a single model — the authors report position-specific features do not work well with existing frequency-based counterparts and defer this to future work

## Inputs

- Dataset.csv containing 41-nt RNA sequences (the iRNA5hmC-PS benchmark dataset)
- Nucleotide strings of length 41 in A/C/G/U(T) alphabet

## Outputs

- A per-sequence GC-content scalar column included in the 19,877-column All_Features_Dataset.csv feature matrix

## How to apply

Load the 41-nt RNA sequences from Dataset.csv with pandas, then compute the fraction of G and C residues in each sequence as a single scalar feature (step 4 of the Feature Generation workflow). Concatenate this GC-content column with the position-specific blocks — PsM, PsD, and PsT (164, 640, and 2496 indicators) and the PSG k-mer modes PsM(G)D, PsD(G)M, and PsM(G)M(G)M (7104, 7104, and 2304 indicators) — into a single 19,877-column matrix saved as All_Features_Dataset.csv. The authors judged GC-content's contribution by rerunning 5-fold cross-validation with SVM, GNB, and LR excluding it, so a correct application supports this ablation. Note the article's caveat that position-specific features do not combine well with frequency-based features, which is why GC content is appended as a standalone block rather than as a frequency feature.

## Related tools

- **pandas** (Loads Dataset.csv and concatenates the GC-content column into the final All_Features_Dataset.csv feature matrix) — https://github.com/zahid6454/iRNA5hmC-PS
- **numpy** (Vectorized computation of the per-sequence GC-content values alongside the binary indicator feature blocks) — https://github.com/zahid6454/iRNA5hmC-PS
- **iRNA5hmC-PS** (Parent framework whose Feature Generation.ipynb notebook computes GC content as one feature block among the position-specific k-mer descriptors) — https://github.com/zahid6454/iRNA5hmC-PS

## Examples

```
df['gc_content'] = (df['sequence'].str.count('G') + df['sequence'].str.count('C')) / df['sequence'].str.len()  # step 4 of Feature Generation.ipynb
```

## Evaluation signals

- Feature matrix column count matches Table 2: total 19,877 features per sequence with GC content contributing exactly 1 column
- GC-content values fall in [0, 1] for every 41-nt sequence in Dataset.csv
- All_Features_Dataset.csv is produced after running Feature Generation.ipynb with 'Run All' and contains one row per input sequence
- Ablation check: 5-fold cross-validation for LR/SVM/GNB with the GC-content feature excluded can be reproduced and compared against the full-feature results

## Limitations

- The authors report that position-specific features (including GC content as part of the position-specific set) do not work well in conjunction with existing frequency-based features; combining both is deferred to a future feature-subspace ensemble approach
- The contribution of GC content is modest enough that the authors ran dedicated 5-fold cross-validation experiments excluding it; its benefit should be verified per classifier
- Sequences must be exactly 41 nt (webserver accepts only 41-length FASTA input), so GC content computed on other lengths is out of scope for this pipeline

## Evidence

- [results] Ablation of GC content: "we have performed additional 5-fold cross-validation experiments for SVM, GNB, and LR without taking GC-content into account"
- [methods] Feature generation workflow: ""Feature Generation.ipynb" uses "Dataset.csv" file to generate all the proposed features and stores that into a new file titled as "All_Features_Dataset.csv""
- [full_text] Combining feature blocks into 19,877 features: [evidence span withheld — anchor status: not_found]
- [readme] Fixed sequence length: "The user has to copy and paste RNA sequence having length of 41. The format has to be FASTA format."
