---
name: iqr-threshold-dichotomization
description: Use when you have a continuous rareness score per cell (FiRE, LOF) and
  need binary rare/non-rare calls for a scRNA-seq dataset, or must compare against
  binary predictors like RaceID and GiniClust.
when_to_use_negative:
- When the rare population is expected to be far below the ~2.3% implied by a Q3 +
  1.5×IQR cutoff — the article separately inspects the top 0.25% of scores to resolve
  a 0.3% megakaryocyte cluster
- When you need to separate outlier cells from cells of genuine minor cell types —
  FiRE (and hence its thresholded calls) does not discriminate between the two
- 'When doublets are the target: doublet rates vary 1–30%, a range that does not necessarily
  comply with the notion of cell rareness captured by this cutoff'
tools:
- name: FiRE
  role: Produces the per-cell rareness score distribution that is dichotomized by
    the IQR cutoff
  repo: github.com/princethewinner/FiRE
- name: LOF
  role: Alternative continuous rareness score dichotomized with the same IQR rule
    so it can be compared with FiRE, RaceID and GiniClust via F1 scores
- name: RaceID
  role: Binary rare-cell predictor used as a comparison baseline after IQR-dichotomized
    FiRE/LOF calls are made
- name: GiniClust
  role: Binary rare-cell predictor used as a comparison baseline after IQR-dichotomized
    FiRE/LOF calls are made
- name: dropClust
  role: Clusters the IQR-flagged rare cells to assign cell-type labels (e.g., the
    megakaryocyte cluster) without special outlier treatment
- name: Python
  role: Implementation environment; the cutoff is computed with np.percentile(score,
    75) and scipy.stats.iqr
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:22ca7a952089ece78ed6ade7d6f092c30ad40c057a34d1991e91cf20dcd3203a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - GiniClust
  - LOF
  - Python
  - R
  - RaceID
  - dropClust
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# iqr-threshold-dichotomization

## Summary

Dichotomize continuous rareness scores (e.g., FiRE scores per cell) into rare vs non-rare calls by thresholding at the third quartile plus 1.5×IQR (th = Q3 + 1.5×IQR), so that score-based and binary methods can be compared and rare cells isolated.

## When to use

Use when you have a continuous rareness score per cell (FiRE, LOF) and need binary rare/non-rare calls for a scRNA-seq dataset, or must compare against binary predictors like RaceID and GiniClust.

## When NOT to use

- When the rare population is expected to be far below the ~2.3% implied by a Q3 + 1.5×IQR cutoff — the article separately inspects the top 0.25% of scores to resolve a 0.3% megakaryocyte cluster
- When you need to separate outlier cells from cells of genuine minor cell types — FiRE (and hence its thresholded calls) does not discriminate between the two
- When doublets are the target: doublet rates vary 1–30%, a range that does not necessarily comply with the notion of cell rareness captured by this cutoff

## Inputs

- Per-cell continuous rareness scores (e.g., FiRE score array of shape [nCells] from model.score())
- LOF continuous scores for the same cells (for cross-method comparability)
- Optionally, dropClust cluster labels to annotate the flagged rare cells

## Outputs

- Binary rare/non-rare prediction vector (predictions[indIqr] = 1) over all cells
- Set of rare-cell indices (indIqr) and the selected rare-cell subset of the preprocessed expression matrix
- Rare-cell counts for validation (e.g., 4238 cells on ~68k PBMC data)

## How to apply

After computing a FiRE score for every expression profile (model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489) applied to ranger-preprocessed, log2-transformed data), calculate Q3 (75th percentile) of the score distribution and its interquartile range, then set the cutoff th = Q3 + 1.5×IQR. Cells with score >= th are labeled rare (prediction 1) and the rest non-rare (prediction 0). The same rule applied to LOF scores makes FiRE and LOF comparable with the binary predictions of RaceID and GiniClust in F1-score comparisons. On the ~68k PBMC dataset this rule yielded 4238 rare cells, and the top 0.25% highest FiRE scores mapped exclusively to the megakaryocyte cluster (~0.3% of cells, per dropClust labels).

## Related tools

- **FiRE** (Produces the per-cell rareness score distribution that is dichotomized by the IQR cutoff) — github.com/princethewinner/FiRE
- **LOF** (Alternative continuous rareness score dichotomized with the same IQR rule so it can be compared with FiRE, RaceID and GiniClust via F1 scores)
- **RaceID** (Binary rare-cell predictor used as a comparison baseline after IQR-dichotomized FiRE/LOF calls are made)
- **GiniClust** (Binary rare-cell predictor used as a comparison baseline after IQR-dichotomized FiRE/LOF calls are made)
- **dropClust** (Clusters the IQR-flagged rare cells to assign cell-type labels (e.g., the megakaryocyte cluster) without special outlier treatment)
- **Python** (Implementation environment; the cutoff is computed with np.percentile(score, 75) and scipy.stats.iqr)

## Examples

```
q3 = np.percentile(score, 75); th = q3 + 1.5*stats.iqr(score); indIqr = np.where(score >= th)[0]; predictions = np.zeros(data.shape[0]); predictions[indIqr] = 1
```

## Evaluation signals

- The cutoff equals Q3 + 1.5×IQR: th = np.percentile(score, 75) + 1.5*stats.iqr(score)
- On the ~68k PBMC dataset the rule selects exactly 4238 rare cells
- The top 0.25% highest FiRE scores map exclusively to the dropClust megakaryocyte cluster (~0.3% of cells), matching Supplementary Fig. 1 and Fig. 2a
- Cross-method F1 comparison is valid only after FiRE and LOF scores are dichotomized with the same IQR rule applied to RaceID/GiniClust binary calls (Fig. 3a ranking reproduced)
- Higher FiRE scores must correspond to rarer cells — verify flagged cells sit in low-density regions of a t-SNE embedding

## Limitations

- The fixed Q3 + 1.5×IQR rule flags roughly a fixed tail fraction, so very small clusters (<~1%) may require examining the top 0.25% of scores rather than the full flagged set
- FiRE does not distinguish technical outliers from cells of minor cell types, so thresholded rare calls can be contaminated by outlier cells that dropClust submerges into minor clusters
- Doublet rates vary between 1% and 30%, which does not necessarily comply with the notion of cell rareness, limiting the thresholded calls' use for doublet detection

## Evidence

- [methods] Dichotomize cells into rare vs non-rare by thresholding FiRE scores at the interquartile range (IQR) cutoff: [evidence span withheld — anchor status: not_found]
- [methods] the rare-cell count equals 4238: [evidence span withheld — anchor status: not_found]
- [discussion] While RaceID15 and GiniClust16 offer binary predictions, FiRE gives a rareness score to every individual expression profile.: "While RaceID15 and GiniClust16 offer binary predictions, FiRE gives a rareness score to every individual expression profile."
- [discussion] FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types: "FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types."
- [readme] q3 = np.percentile(score, 75); iqr = stats.iqr(score); th = q3 + 1.5*iqr: "q3 = np.percentile(score, 75)
iqr = stats.iqr(score)
th = q3 + 1.5*iqr

indIqr = np.where(score >= th)[0]"
