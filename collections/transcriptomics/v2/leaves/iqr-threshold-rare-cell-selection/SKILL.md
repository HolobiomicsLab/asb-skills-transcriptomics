---
name: iqr-threshold-rare-cell-selection
description: Use when FiRE (or an equivalent LSH-based density estimator) has produced
  a per-cell rareness score array and a binary rare/abundant cell label is needed
  for downstream clustering of candidate rare subpopulations.
when_to_use_negative:
- Input is raw or unnormalized counts — FiRE scores must be computed on the log2-transformed
  ranger_preprocess output before IQR thresholding is meaningful
- The goal is doublet detection — the article notes doublet rate varies between 1%
  and 30%, a range that does not necessarily comply with the notion of cell rareness
- Rare and outlier cells must be distinguished — FiRE does not discriminate between
  an outlier and cells representing minor cell types, and outlier cells may get submerged
  into minor cell clusters when the selected cells are clustered
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: FiRE
  role: Produces the per-cell rareness scores (LSH-based density estimator) that the
    IQR threshold is applied to
  repo: https://github.com/princethewinner/FiRE
- name: Python
  role: Implementation environment; thresholding uses numpy (np.percentile) and scipy.stats
    (iqr)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:07dbfc7e431c7160b64edec4769fc361c67b7c8643918e5a2038e75568e20784
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princethewinner/FiRE
  tools:
  - FiRE
  - Python
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# iqr-threshold-rare-cell-selection

## Summary

Selects rare cells from a scRNA-seq dataset by thresholding per-cell FiRE rareness scores with the IQR rule (score >= Q3 + 1.5*IQR), converting continuous rareness scores into a binary rare-cell call. In the Arc-ME mouse brain application this retained 727 rare cells that were subsequently clustered into R1-R12 subpopulations.

## When to use

Use when FiRE (or an equivalent LSH-based density estimator) has produced a per-cell rareness score array and a binary rare/abundant cell label is needed for downstream clustering of candidate rare subpopulations.

## When NOT to use

- Input is raw or unnormalized counts — FiRE scores must be computed on the log2-transformed ranger_preprocess output before IQR thresholding is meaningful
- The goal is doublet detection — the article notes doublet rate varies between 1% and 30%, a range that does not necessarily comply with the notion of cell rareness
- Rare and outlier cells must be distinguished — FiRE does not discriminate between an outlier and cells representing minor cell types, and outlier cells may get submerged into minor cell clusters when the selected cells are clustered

## Inputs

- Per-cell FiRE rareness score array (np.array[nCells], higher score = rarer) from model.score(preprocessedData)
- Log2-transformed preprocessed expression matrix (np.array[nCells x nVariableGenes]) from pp.ranger_preprocess

## Outputs

- Index array of rare-cell indices passing the IQR threshold (e.g. indIqr)
- Binary prediction vector (np.zeros(nCells) with selected rare cells set to 1)
- Subset expression matrix of rare cells (np.array[nRare x nVariableGenes]) for downstream dropClust clustering, e.g. the 727 Arc-ME rare cells

## How to apply

Compute the 75th percentile (Q3) of the FiRE score vector with np.percentile(score, 75) and the interquartile range with scipy.stats.iqr(score); the detection threshold is th = Q3 + 1.5*IQR. Select all cells whose score >= th (e.g. indIqr = np.where(score >= th)[0]) and subset the log2-transformed preprocessed expression matrix (rows = selected cells, columns = the ~1000 variable genes kept by ranger_preprocess) accordingly. Build a binary prediction vector where selected rare cells are labeled 1 and all other cells 0. Higher FiRE scores correspond to rarer cells, so the threshold keeps only the sparse-density tail of the score distribution. In the Arc-ME mouse brain dataset (Campbell et al., ~20k cells) this IQR rule retained exactly 727 rare cells, which were then clustered with dropClust into 12 clusters (R1-R12) without special outlier treatment.

## Related tools

- **FiRE** (Produces the per-cell rareness scores (LSH-based density estimator) that the IQR threshold is applied to) — https://github.com/princethewinner/FiRE
- **Python** (Implementation environment; thresholding uses numpy (np.percentile) and scipy.stats (iqr))

## Examples

```
q3 = np.percentile(score, 75); iqr = stats.iqr(score); indIqr = np.where(score >= q3 + 1.5*iqr)[0]; predictions[indIqr] = 1
```

## Evaluation signals

- Threshold equals Q3 + 1.5*IQR of the score vector: verify th == np.percentile(score, 75) + 1.5 * stats.iqr(score)
- Number of retained cells should be a small sparse tail; on the ~20k Arc-ME mouse brain dataset the IQR rule retains 727 rare cells
- All retained indices satisfy score >= th, i.e. np.all(score[indIqr] >= th)
- Binary prediction vector has exactly len(indIqr) entries equal to 1 and the rest 0
- Retained cells, when clustered with dropClust, should reproduce marker-defined clusters such as R6 erythrocyte markers (Alas2, Hba-a1, Hba-a2, Hbb-bt) and the R10 Hesx1+/Cyp2f2+ pars tuberalis sub-type

## Limitations

- FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types, so IQR-selected cells may include technical outliers
- Outlier cells, if any, get submerged into the minor cell clusters when FiRE-detected rare cells are clustered with dropClust, since dropClust administers no special treatment to outliers
- The IQR rule is sensitive to the score distribution shape; the number of detected rare cells (e.g. 727 of ~20k in Arc-ME) depends on the dataset's density profile

## Evidence

- [readme] IQR threshold selects sparse-density tail of FiRE scores: [evidence span withheld — anchor status: external_doc]
- [readme] Threshold formula th = q3 + 1.5*iqr: "q3 = np.percentile(score, 75)
iqr = stats.iqr(score)
th = q3 + 1.5*iqr"
- [readme] Higher FiRE scores represent rare cells: [evidence span withheld — anchor status: external_doc]
- [abstract] FiRE assigns rareness score to every profile in seconds: "We propose Finder of Rare Entities (FiRE), an algorithm that, in a matter of seconds, assigns a rareness score to every individual expression profile under study."
- [abstract] FiRE recovers novel pars tuberalis sub-type from mouse brain scRNA-seq: "When applied to a large scRNA-seq dataset of mouse brain cells, FiRE recovered a novel sub-type of the pars tuberalis lineage."
- [discussion] dropClust used for clustering FiRE-detected rare cells: "For clustering FiRE detected rare cells, in all our analyses, we adhered to dropClust24 that does not administer any special treatment to outliers."
