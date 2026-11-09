---
name: nearest-neighbor-reference-profile-matching
description: Use when scRNA-seq cells (e.g., 10x Chromium barcoded cells in _counts_UB.tsv)
  need lineage or cell-type labels without marker genes or supervised training labels,
  and a suitable reference expression profile set exists.
when_to_use_negative:
- The goal is variant/genotype calling per cell (e.g., expressed somatic SNVs) — use
  the mutation barcode caller instead, not expression-profile matching.
- No reference expression profiles are available for the expected cell types of the
  sample, so nearest-neighbor matching would assign arbitrary labels.
- Marker genes or supervised cell-type labels are already available and a supervised
  classifier is preferred.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: python3
  role: Language used to compute Spearman correlations and train/run the nearest-neighbor
    classifier on DMAP reference profiles
- name: sridnona/python3:180925.v1
  role: Docker image packaging the Python environment in which the DMAP-matching workflow
    is launched (e.g., via bsub -a 'docker(sridnona/python3:180925.v1)')
- name: Docker
  role: Container runtime used to launch the packaged analysis environment on LSF
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:d002e83a284389acd522b40aecefb8ac7b2644b36ce250c15d601f5f445f8837
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - python3
  - sridnona/python3:180925.v1
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# nearest-neighbor-reference-profile-matching

## Summary

Unsupervised, marker-free cell-type assignment in 10x Chromium scRNA-seq data: each cell's expression profile is matched to hematopoietic reference profiles (e.g., DMAP lineages such as RBC, HSC, CMP, MEP, monocyte, T-cell, B-cell) using Spearman rank correlation as the similarity measure, and a nearest-neighbor classifier assigns the most-correlated lineage label.

## When to use

Use when scRNA-seq cells (e.g., 10x Chromium barcoded cells in _counts_UB.tsv) need lineage or cell-type labels without marker genes or supervised training labels, and a suitable reference expression profile set exists.

## When NOT to use

- The goal is variant/genotype calling per cell (e.g., expressed somatic SNVs) — use the mutation barcode caller instead, not expression-profile matching.
- No reference expression profiles are available for the expected cell types of the sample, so nearest-neighbor matching would assign arbitrary labels.
- Marker genes or supervised cell-type labels are already available and a supervised classifier is preferred.

## Inputs

- Per-cell expression count matrix (_counts_UB.tsv), read counts per Chromium molecular barcode (UB_tag)
- DMAP hematopoietic lineage reference expression profiles (RBC, HSC, CMP, MEP, monocyte, T-cell, B-cell, etc.)

## Outputs

- cell_lineage_labels.tsv — one row per cell with cell barcode and inferred lineage assignment

## How to apply

Launch the packaged Python environment via the Docker image sridnona/python3:180925.v1, then load the per-cell read count matrix (_counts_UB.tsv, read counts per Chromium molecular barcode/UB_tag) together with the DMAP hematopoietic lineage reference expression profiles. Compute the Spearman rank correlation between each cell's expression vector and each lineage profile; rank-based correlation is chosen because it is robust to the dropout and non-linear scaling inherent in 10x Chromium scRNA-seq data. Train a nearest-neighbor classifier on the reference profiles and assign each cell to the most-correlated lineage, requiring no marker genes or supervised labels. Write per-cell assignments to cell_lineage_labels.tsv, one row per cell with the cell barcode and lineage label.

## Related tools

- **python3** (Language used to compute Spearman correlations and train/run the nearest-neighbor classifier on DMAP reference profiles)
- **sridnona/python3:180925.v1** (Docker image packaging the Python environment in which the DMAP-matching workflow is launched (e.g., via bsub -a 'docker(sridnona/python3:180925.v1)'))
- **Docker** (Container runtime used to launch the packaged analysis environment on LSF)

## Examples

```
bsub -a "docker(sridnona/python3:180925.v1)" python3 dmap_match.py --counts _counts_UB.tsv --reference dmap_profiles.tsv --out cell_lineage_labels.tsv
```

## Evaluation signals

- cell_lineage_labels.tsv exists and is valid TSV with one row per cell, each row pairing a cell barcode with a lineage assignment
- Every cell across all five AML samples receives exactly one lineage label (no unassigned cells)
- Every assigned label is a member of the DMAP lineage set used for classification (e.g., myelomonocytic, T-cell, B-cell, erythrocyte-like labels)
- The inferred lineage composition varies widely among subjects, consistent with per-sample heterogeneity expected in AML samples
- Spearman correlation (not linear/Pearson on raw counts) is demonstrably the similarity measure used, confirming robustness to dropout and non-linear coverage scaling

## Limitations

- Coverage drops non-linearly across transcript length on end-biased platforms such as Chromium, which affects profile comparability and detection sensitivity
- Dropout (transcript and allelic) occurs with most scRNA-seq platforms, limiting the fidelity of both expression profiles and derived lineage labels
- Labels are only as good as the DMAP reference set — lineages absent from the reference cannot be assigned

## Evidence

- [methods] Read counts per Chromium molecular barcode input: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [methods] Packaged Docker analysis environment: "Docker image
----------
* sridnona/python3:180925.v1"
- [other] DMAP-matching produced per-cell inferred lineage labels that varied across subjects: [evidence span withheld — anchor status: not_found]
- [discussion] Dropout limits scRNA-seq analyses platform-wide: "Dropout (including transcript dropout and allelic dropout) occurs with most scRNA-seq platforms."
