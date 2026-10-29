---
name: pca-embedding-batch-assessment
description: Use when you need to quantify residual batch effects after correction
  (e.g. ComBat on two-technical-replicate mESC data) in a linear PC space, or to build
  the PCA baseline needed for a sensitivity comparison against a nonlinear embedding
  such as BEENE.
when_to_use_negative:
- The goal is to remove/correct batch effects (e.g. running ComBat itself) rather
  than estimate their extent — this skill assesses mixing in an embedding downstream
  of correction
- The data have no batch variable (single batch) — kBET and iLISI require batch labels
  to compute rejection rates and local inverse Simpson indices
- The embedding under evaluation is already a nonlinear BEENE embedding — this skill
  defines the PCA baseline space for comparison, not the nonlinear embedding generation
  step
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn
  role: Computes the PCA embedding of the (ComBat-corrected) expression matrix that
    serves as the baseline batch-effect space
- name: BEENE
  role: Nonlinear challenger embedding (built with get_hybrid_model_1, trained with
    train_model) whose kBET/iLISI values are compared against the PCA baseline
  repo: https://github.com/ashiq24/BEENE
- name: kBET
  role: R package computing the average chi-square test rejection rate on the k-nearest-neighbour
    graph of the PCA embedding to quantify batch bias
  repo: https://github.com/theislab/kBET/
- name: LISI
  role: Source of the iLISI metric (evaluate_batch_iLisi) measuring batch mixing of
    cells in the PCA space
  repo: https://github.com/immunogenomics/LISI
- name: rpy2
  role: Transfers the Python-computed PCA embedding into the R environment so kBET
    can be executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad479
    title: 'BEENE: deep learning-based nonlinear embedding improves batch effect estimation'
  source_content_hash: sha256:ac880944ee82671c383ce85df616eeea0702a214dd19d0ff242cfa68f27a5dbc
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# pca-embedding-batch-assessment

## Summary

Computes a PCA embedding of RNA-seq/single-cell data (raw or ComBat-corrected, e.g. mESC technical replicates) and quantifies residual batch effects with kBET rejection rate and iLISI. This PCA space serves as the linear baseline against which the BEENE paper shows nonlinear embeddings are more sensitive to sophisticated batch effects.

## When to use

Use when you need to quantify residual batch effects after correction (e.g. ComBat on two-technical-replicate mESC data) in a linear PC space, or to build the PCA baseline needed for a sensitivity comparison against a nonlinear embedding such as BEENE.

## When NOT to use

- The goal is to remove/correct batch effects (e.g. running ComBat itself) rather than estimate their extent — this skill assesses mixing in an embedding downstream of correction
- The data have no batch variable (single batch) — kBET and iLISI require batch labels to compute rejection rates and local inverse Simpson indices
- The embedding under evaluation is already a nonlinear BEENE embedding — this skill defines the PCA baseline space for comparison, not the nonlinear embedding generation step

## Inputs

- 2D numpy expression matrix (rows = samples/cells, columns = genes), e.g. GSE65525 mESC data
- Batch labels: 1D array of zeros/ones for two batches, or dense one-hot matrix for >2 batches
- Biological variable labels (1D 0/1 array or one-hot), or None if absent
- ComBat-corrected expression matrix (when assessing residual effects post-correction)
- Integer random seed controlling the train/validation/test split

## Outputs

- PCA embedding matrix (n_samples x k coordinates, e.g. k=5)
- kBET average rejection rate for the PCA embedding (R package output)
- Per-sample iLISI values for the PCA embedding
- Comparison table of kBET and iLISI values for PCA vs BEENE embeddings

## How to apply

Load the expression matrix and batch labels into numpy arrays (per the BEENE README, for two categories the batch variable is a 1D array of zeros and ones; for more than two, a dense one-hot matrix), and use the corrected data (e.g. ComBat output) as input. Compute a PCA embedding with scikit-learn at the same embedding dimension as the challenger embedding (e.g. 5-D, as in the article's get_hybrid_model_1(100,[50,20],5,3,2,1,2,1) example) so the kBET/iLISI comparison is fair. Compute iLISI on the PCA coordinates with evaluate_batch_iLisi(data, batch_var, bio_var, seed), using a fixed random seed for the train/validation/test split; for two-category variables plain 0/1 arrays are accepted. Because kBET has no Python implementation, transfer the PCA embedding to the R environment via rpy2 (rpy2 >= 3.4.0, R >= 4.1.0, Linux recommended) and run the kBET R package, which chi-square-tests the batch-label distribution in 10%-sampled k-nearest-neighbour neighbourhoods and returns the average rejection rate. Interpret with the README's rule: the lower the kBET rejection rate, the less batch bias — so on ComBat-corrected mESC data, PCA rejection rates of 0.33 (full) / 0.20 (test) versus BEENE's 0.58, with iLISI ~1.76 (PCA) vs 1.57 (BEENE), indicate PCA under-detects the residual batch structure BEENE surfaces. Assemble the kBET and iLISI values for both embeddings into a comparison table to conclude the assessment.

## Related tools

- **scikit-learn** (Computes the PCA embedding of the (ComBat-corrected) expression matrix that serves as the baseline batch-effect space)
- **BEENE** (Nonlinear challenger embedding (built with get_hybrid_model_1, trained with train_model) whose kBET/iLISI values are compared against the PCA baseline) — https://github.com/ashiq24/BEENE
- **kBET** (R package computing the average chi-square test rejection rate on the k-nearest-neighbour graph of the PCA embedding to quantify batch bias) — https://github.com/theislab/kBET/
- **LISI** (Source of the iLISI metric (evaluate_batch_iLisi) measuring batch mixing of cells in the PCA space) — https://github.com/immunogenomics/LISI
- **rpy2** (Transfers the Python-computed PCA embedding into the R environment so kBET can be executed)

## Examples

```
from sklearn.decomposition import PCA; pca_emb = PCA(n_components=5).fit_transform(X_combat_corrected)
```

## Evaluation signals

- On ComBat-corrected mESC (GSE65525) data, reproduce kBET rejection rates of 0.33 (full dataset) and 0.20 (test set) in PCA space vs 0.58 in BEENE space
- iLISI in PCA space should be ~1.76 vs 1.57 in the BEENE embedding for the same split seed
- The PCA embedding dimension must equal the BEENE embedding dimension (e.g. 5) for a like-for-like kBET/iLISI comparison
- Re-running evaluate_batch_iLisi with the same seed must yield identical per-sample iLISI values (deterministic split)
- Directionality check: a lower kBET rejection rate means better batch mixing per the kBET README, so residuals detected more often (higher rejection) signal a more sensitive embedding

## Limitations

- PCA-space batch estimators 'may fail to capture sophisticated non-linear batch effects', so a low kBET rejection rate in PC space does not guarantee the absence of nonlinear residual effects
- kBET has no Python implementation and must be run in R via rpy2, which the BEENE authors recommend only on Linux (rpy2 >= 3.4.0, R >= 4.1.0, Python >= 3.7.0)
- If kBET returns an average rejection rate of 1 on corrected data, the kBET README advises additionally computing silhouette width and PCA-based batch-effect measures to gauge the degree of the effect

## Evidence

- [intro] PCA under-detects nonlinear batch effects relative to BEENE: "resulting in an embedding that is more robust and sensitive than PCA embedding in terms of detecting and quantifying batch effects"
- [readme] PCA baselines may miss nonlinear batch effects: "the batch effect estimation techniques mostly depend on the local distribution of cells in the Principal Component (PC) space, which may fail to capture sophisticated non-linear batch effects"
- [methods] kBET requires embedding transfer to R: "To calculate kBET using non-linear embedding, the learned embedding must be transferred to R environment for calculation."
- [methods] iLISI computation call on the embedding: "lisi_values = my_model.evaluate_batch_iLisi(Xt,bt,yt,20)"
- [readme] Interpretation of kBET rejection rate: [evidence span withheld — anchor status: external_doc]
