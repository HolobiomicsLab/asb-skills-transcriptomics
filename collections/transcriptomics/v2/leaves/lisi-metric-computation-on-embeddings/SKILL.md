---
name: lisi-metric-computation-on-embeddings
description: 'Use when quantifying batch effects in scRNA-seq data via local batch-label
  diversity: you have an embedding (BEENE or PCA), per-cell batch labels across multiple
  batches/platforms, and need per-sample iLISI values for downstream kBET/iLISI comparison
  tables.'
when_to_use_negative:
- The goal is batch effect removal/correction (e.g., integration with Harmony) rather
  than detection and estimation of the extent of batch effects
- 'The downstream metric is kBET and you want a pure-Python path: kBET has no Python
  implementation and must run in R, so rpy2 transfer is mandatory'
- You expect iLISI to be bounded by [1, 5] on a dataset with fewer (or more) than
  five batch categories — the iLISI upper bound equals the number of batches, so range
  checks must match the batch count
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BEENE
  role: Provides the _beene_model_ class and the evaluate_batch_iLisi(data, batch_var,
    bio_var, seed) member function that trains the model and returns per-sample iLISI
    values on the learned nonlinear embedding
  repo: https://github.com/ashiq24/BEENE
- name: LISI
  role: Reference R implementation of the Local Inverse Simpson's Index batch effect
    estimation metric
  repo: https://github.com/immunogenomics/LISI
- name: kBET
  role: Complementary k-nearest-neighbour batch effect test computed on the same embeddings;
    rejection rates accompany median iLISI in the comparison table
  repo: https://github.com/theislab/kBET
- name: rpy2
  role: Transfers the learned embedding from Python to the R environment for kBET
    calculation
- name: scikit-learn
  role: OneHotEncoder produces the dense one-hot batch-label matrix required by evaluate_batch_iLisi
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad479
    title: 'BEENE: deep learning-based nonlinear embedding improves batch effect estimation'
  source_content_hash: sha256:8308039e293d10072f709868650c2acf354b8b82868b0a9774729ffe1531377a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ashiq24/BEENE
  tools:
  - BEENE
  - LISI
  - R
  - kBET
  - numpy
  - rpy2
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# lisi-metric-computation-on-embeddings

## Summary

Computes the Local Inverse Simpson's Index (iLISI) batch-mixing metric on BEENE nonlinear embeddings instead of PCA space, quantifying how well cells from different batches (e.g., inDrop, Fluidigm C1, Smart-Seq2, CEL-seq, CEL-seq2) are locally mixed.

## When to use

Use when quantifying batch effects in scRNA-seq data via local batch-label diversity: you have an embedding (BEENE or PCA), per-cell batch labels across multiple batches/platforms, and need per-sample iLISI values for downstream kBET/iLISI comparison tables.

## When NOT to use

- The goal is batch effect removal/correction (e.g., integration with Harmony) rather than detection and estimation of the extent of batch effects
- The downstream metric is kBET and you want a pure-Python path: kBET has no Python implementation and must run in R, so rpy2 transfer is mandatory
- You expect iLISI to be bounded by [1, 5] on a dataset with fewer (or more) than five batch categories — the iLISI upper bound equals the number of batches, so range checks must match the batch count

## Inputs

- 2D numpy array of gene expression per cell (rows = samples, columns = genes), e.g. the 14,746-cell pancreatic islet matrix from harmony2019 data/figure5
- Batch variable as a dense one-hot numpy matrix (five platforms: inDrop, Fluidigm C1, Smart-Seq2, CEL-seq, CEL-seq2) or 1D 0/1 array for two categories
- Biological variable (donor labels) as dense one-hot matrix or 1D array
- Integer random seed for the train/validation/test split (e.g., seed=20)
- Learned BEENE embedding or PCA coordinates to score

## Outputs

- Per-sample iLISI values for each sample in the held-out test set
- embedding.txt: space-separated BEENE embedding stored with np.savetxt
- batch_metrics_table.csv with per-batch kBET rejection rates and median iLISI for PCA and BEENE spaces

## How to apply

One-hot encode the multi-category batch labels with sklearn's OneHotEncoder as a dense matrix (two-category labels may be a 1D array of 0s/1s), and supply the biological variable (e.g., donor) likewise. Call the BEENE model member function evaluate_batch_iLisi(data, batch_var, bio_var, seed): it internally splits the data into training/validation/test sets, trains the model on the training set, selects the best model on the validation set, and returns iLISI values for each sample in the randomly chosen test set computed on the BEENE embedding. Run it in parallel on PCA-space coordinates and on BEENE embeddings (saved via np.savetxt as embedding.txt) so the two can be compared, since BEENE is claimed to be more robust and sensitive than PCA embedding for detecting and quantifying batch effects. For the kBET rejection-rate counterpart, transfer the embeddings to R through rpy2 (>= 3.4.0) because kBET has no Python implementation, then aggregate per-batch kBET rates and median iLISI values into a metrics table.

## Related tools

- **BEENE** (Provides the _beene_model_ class and the evaluate_batch_iLisi(data, batch_var, bio_var, seed) member function that trains the model and returns per-sample iLISI values on the learned nonlinear embedding) — https://github.com/ashiq24/BEENE
- **LISI** (Reference R implementation of the Local Inverse Simpson's Index batch effect estimation metric) — https://github.com/immunogenomics/LISI
- **kBET** (Complementary k-nearest-neighbour batch effect test computed on the same embeddings; rejection rates accompany median iLISI in the comparison table) — https://github.com/theislab/kBET
- **rpy2** (Transfers the learned embedding from Python to the R environment for kBET calculation)
- **scikit-learn** (OneHotEncoder produces the dense one-hot batch-label matrix required by evaluate_batch_iLisi)

## Examples

```
lisi_values = my_model.evaluate_batch_iLisi(Xt, bt, yt, 20)
```

## Evaluation signals

- evaluate_batch_iLisi returns one iLISI value per sample of the randomly chosen test set — the output length must equal the test-set size
- On the five-batch pancreatic islet data, every iLISI value must fall within [1, 5]; on k batches, within [1, k]
- Batch labels with more than two categories must be passed as a dense one-hot matrix, not a sparse or label-encoded array
- Per-batch kBET rejection rates must lie in [0, 1] for both PCA and BEENE embeddings, matching the Supplementary Section 2.6 reported structure
- BEENE median iLISI should be reported alongside PCA median iLISI in batch_metrics_table.csv for the same batches so the robustness/sensitivity comparison is checkable

## Limitations

- The kBET metric has no Python implementation and is available only in R (rpy2 >= 3.4.0, Linux recommended)
- evaluate_batch_iLisi performs its own train/validation/test split, so scores are computed only on the held-out test set, not all cells
- For more than two categories, batch and biological variables must be one-hot dense matrices; sparse encodings violate the API contract
- No changelog/discussion content was available in the article text, so edge-case behaviour beyond documented parameters is unverified

## Evidence

- [readme] returns: iLISI values of each of the samples in the randomly chosen test set using BEENE embedding: "returns: iLISI values of each of the samples in the randomly chosen test set using BEENE embedding"
- [readme] batch_var dense one-hot requirement: "batch_var: numpy array. For more than two categories, it must be one hot representation of"
- [methods] iLISI call signature: "lisi_values = my_model.evaluate_batch_iLisi(Xt,bt,yt,20)"
- [methods] evaluate_batch_iLisi definition: "evaluate_batch_iLisi(data, batch_var, bio_var, seed):"
- [methods] LISI metric description: "LISI is a batch effect estimation metric [Link](https://github.com/immunogenomics/LISI)"
- [abstract] BEENE outperforms PCA embedding: "an embedding that is more robust and sensitive than PCA embedding in terms of detecting and quantifying"
