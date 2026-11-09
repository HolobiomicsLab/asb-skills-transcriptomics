---
name: cosine-similarity-benchmarking-across-scoring-methods
description: Use when you need to quantitatively validate that a fast or reimplemented
  pathway scoring method reproduces the outputs of an established method on the same
  cells and pathway set, e.g. via per-cell cosine similarity of PAS vectors.
when_to_use_negative:
- You are comparing methods on different pathway sets or different max rank thresholds
  — agreement is only expected when both use the same threshold and pathways.
- You need a differential-expression or pathway-enrichment test rather than a numerical
  equivalence check between two PAS matrices.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scPAFA
  role: provides the fast_Ucell implementation used to compute one side of the PAS
    comparison
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: AUCell
  role: established pathway activity scoring method used as the reference PAS in the
    cosine similarity comparison
- name: UCell
  role: fast_ucell provides the same PAS as UCell, which is closely related to AUCell
    PAS when the same max rank threshold is set
- name: scikit-learn
  role: supplies cosine_similarity used to compute pairwise per-cell similarity between
    PAS vectors
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:4baf4bb7a39150c0eeeb7f09a5450f4a9c432e347b828b46ce93e6006acb3d08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - AUCell
  - Python
  - UCell
  - scPAFA
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# cosine-similarity-benchmarking-across-scoring-methods

## Summary

Validates agreement between two single-cell pathway activity scoring methods (scPAFA's fast_ucell and AUCell) by computing per-cell cosine similarity between their PAS vectors and averaging per dataset.

## When to use

Use when you need to quantitatively validate that a fast or reimplemented pathway scoring method reproduces the outputs of an established method on the same cells and pathway set, e.g. via per-cell cosine similarity of PAS vectors.

## When NOT to use

- You are comparing methods on different pathway sets or different max rank thresholds — agreement is only expected when both use the same threshold and pathways.
- You need a differential-expression or pathway-enrichment test rather than a numerical equivalence check between two PAS matrices.

## Inputs

- scRNA-seq expression matrix subset (e.g. 100k-cell subsets of CRC and lupus datasets)
- pathway dictionary (pathway name -> gene list), e.g. from MsigDB or NCATS BioPlanet
- PAS matrix from fast_ucell (scPAFA)
- PAS matrix from AUCell computed on the same cells and pathways with the same max rank threshold

## Outputs

- per-cell cosine similarity values between fast_ucell and AUCell PAS vectors
- per-dataset average cosine similarity (e.g. pas_cosine_similarity_comparison.csv with CRC and lupus averages)

## How to apply

Run both scoring methods on identical inputs: the same expression matrix subset (e.g. 100k-cell subsets of the CRC and lupus scRNA-seq datasets) and the same pathway dictionaries (e.g. MsigDB/NCATS BioPlanet, 1629 pathways for CRC, 1383 for lupus). Compute fast_ucell PAS and AUCell PAS using the same max rank threshold, since fast_ucell provides the same PAS as UCell which is closely related to AUCell PAS when max rank is matched. For each cell, compute the pairwise cosine similarity between the fast_ucell PAS vector and the AUCell PAS vector using scikit-learn's cosine_similarity. Average the per-cell similarities per dataset and write results (e.g. pas_cosine_similarity_comparison.csv). Judge success against the reported averages of 0.97 (CRC) and 0.94 (lupus) within a small tolerance.

## Related tools

- **scPAFA** (provides the fast_Ucell implementation used to compute one side of the PAS comparison) — https://github.com/ZhuoliHuang/scPAFA
- **AUCell** (established pathway activity scoring method used as the reference PAS in the cosine similarity comparison)
- **UCell** (fast_ucell provides the same PAS as UCell, which is closely related to AUCell PAS when the same max rank threshold is set)
- **scikit-learn** (supplies cosine_similarity used to compute pairwise per-cell similarity between PAS vectors)

## Examples

```
from sklearn.metrics.pairwise import cosine_similarity; sims = [cosine_similarity(fu_p[c].reshape(1,-1), au_p[c].reshape(1,-1))[0,0] for c in range(fu_p.shape[0])]
```

## Evaluation signals

- Average per-cell cosine similarity should reproduce the reported 0.97 for the CRC dataset within a small tolerance.
- Average per-cell cosine similarity should reproduce the reported 0.94 for the lupus dataset within a small tolerance.
- Both methods must be run on the same cell subset and same pathway set with the same max rank threshold before comparison.
- Output CSV (e.g. pas_cosine_similarity_comparison.csv) should contain one average value per dataset (CRC and lupus).

## Limitations

- Agreement holds only when fast_ucell and AUCell use the same max rank threshold; otherwise PAS values need not match.
- Like AUCell, fast_ucell PAS is based on within-cell gene expression ranking, making it insensitive to dataset composition — the benchmark reflects ranking-based behavior rather than absolute expression scale.
- The validation was performed on 100k-cell subsets, not the full million-cell datasets.

## Evidence

- [other] fast_ucell PAS closely matches AUCell PAS with reported average cosine similarities: [evidence span withheld — anchor status: not_found]
- [discussion] The 'fast_ucell' function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold.: "The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold."
- [other] For each cell, compute the pairwise cosine similarity between the fast_ucell PAS vector and the AUCell PAS vector using scikit-learn's cosine similarity.: [evidence span withheld — anchor status: not_found]
- [readme] README documents fast_Ucell as the PAS computation step: "single-cell gene expression matrix and collection of pathways are used to compute PAS by ‘fast_Ucell’"
