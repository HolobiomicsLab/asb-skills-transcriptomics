---
name: dmap-lineage-label-transfer
description: Use when you need per-cell lineage or cell-type labels from 10x Chromium
  scRNA-seq data without marker genes or supervised training, and a hematopoietic
  reference profile set (e.g. DMAP) is available for correlation-based matching.
when_to_use_negative:
- You already have reliable marker-gene annotations or a supervised cell-type classifier
  for your data — a marker-free correlation transfer adds no information there.
- The sample is not hematopoietic or the target lineages are absent from the DMAP
  reference set — the classifier can only assign labels drawn from the reference lineage
  set.
- Variant/genotype calling (e.g. cb_sniffer or VarTrix) is the actual goal — this
  skill only labels lineage, it does not call mutations.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: python3
  role: Language in which the Spearman-correlation / nearest-neighbor DMAP classifier
    is implemented
- name: sridnona/python3:180925.v1
  role: Docker image providing the packaged Python environment for the lineage-assignment
    workflow
- name: Docker
  role: Container runtime via which the packaged environment is launched (bsub -a
    "docker(...)")
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:e95be223c5abea659d99abfc0e5555bdcf10b345517af7b3b478d43f5ae37cd8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - python3
  - sridnona/python3:180925.v1
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# dmap-lineage-label-transfer

## Summary

Assigns an unsupervised, marker-free hematopoietic lineage label to every single cell by Spearman-correlating each cell's scRNA-seq expression profile against DMAP reference lineage profiles and taking the nearest (most-correlated) lineage. Used in this paper to place AML sample cells into lineage contexts (myelomonocytic, T-cell, B-cell, erythrocyte, HSC, CMP, MEP) before connecting genotype to phenotype.

## When to use

Use when you need per-cell lineage or cell-type labels from 10x Chromium scRNA-seq data without marker genes or supervised training, and a hematopoietic reference profile set (e.g. DMAP) is available for correlation-based matching.

## When NOT to use

- You already have reliable marker-gene annotations or a supervised cell-type classifier for your data — a marker-free correlation transfer adds no information there.
- The sample is not hematopoietic or the target lineages are absent from the DMAP reference set — the classifier can only assign labels drawn from the reference lineage set.
- Variant/genotype calling (e.g. cb_sniffer or VarTrix) is the actual goal — this skill only labels lineage, it does not call mutations.

## Inputs

- _counts_UB.tsv — per-cell read counts for Chromium molecular barcode (UB_tag) from the 10x Genomics Chromium Single Cell 5' (v1) workflow
- DMAP hematopoietic lineage reference expression profiles (RBC, HSC, CMP, MEP, monocyte, T-cell, B-cell, etc.)

## Outputs

- cell_lineage_labels.tsv — one row per cell with cell barcode and inferred lineage assignment
- per-cell inferred lineage composition for each of the five AML samples

## How to apply

Launch the packaged Python environment via the Docker image sridnona/python3:180925.v1 (e.g. bsub -a "docker(sridnona/python3:180925.v1)"). Load the per-cell read-count matrix (_counts_UB.tsv, read counts per Chromium molecular barcode / UB_tag) and the DMAP hematopoietic lineage reference expression profiles covering lineages such as RBC, HSC, CMP, MEP, monocyte, T-cell, and B-cell. Compute the Spearman rank correlation between each cell's expression profile and each DMAP lineage profile — rank correlation is chosen because it is robust to the dropout and non-linear scaling characteristic of 10x Chromium scRNA-seq data. Train a nearest-neighbor classifier on the DMAP reference profiles and assign each cell to the most-correlated lineage label, requiring no marker genes or supervised labels. Write per-cell assignments to cell_lineage_labels.tsv (one row per cell: barcode + lineage). Analyze genotyped cells in the context of the entire sample so expression information from non-genotyped cells also informs the lineage picture.

## Related tools

- **python3** (Language in which the Spearman-correlation / nearest-neighbor DMAP classifier is implemented)
- **sridnona/python3:180925.v1** (Docker image providing the packaged Python environment for the lineage-assignment workflow)
- **Docker** (Container runtime via which the packaged environment is launched (bsub -a "docker(...)"))

## Examples

```
bsub -a "docker(sridnona/python3:180925.v1)" python3 assign_dmap_lineages.py --counts _counts_UB.tsv --ref dmap_lineage_profiles.tsv --out cell_lineage_labels.tsv
```

## Evaluation signals

- cell_lineage_labels.tsv exists and is valid TSV with a lineage label for every cell across the five AML samples
- Every assigned label is a member of the DMAP lineage set used for classification (RBC, HSC, CMP, MEP, monocyte, T-cell, B-cell, ...)
- One output row per input cell barcode in _counts_UB.tsv — no barcodes dropped or duplicated
- Inferred lineage composition varies across subjects and recovers recognizable populations resembling myelomonocytic cells, T-cells, B-cells, and erythrocytes rather than collapsing to a single lineage

## Limitations

- Labels are only as good as the DMAP reference set — cells can only receive lineages present in the reference, so AML blasts may be forced onto the nearest normal lineage.
- Spearman correlation was chosen specifically because dropout and non-linear scaling in 10x Chromium data degrade other similarity measures; low-count cells may still receive unreliable assignments.
- The approach is unsupervised/marker-free and does not itself distinguish AML from normal cells; genotype information (mutation detection) must be integrated separately.

## Evidence

- [results] DMAP-matching workflow and per-sample output: [evidence span withheld — anchor status: not_found]
- [methods] Count matrix input format: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
- [methods] Packaged execution environment: "Docker image
----------
* sridnona/python3:180925.v1"
- [discussion] Rationale for whole-sample analysis: "we analyzed them in the context of the entire sample, thereby leveraging the expression information provided by the additional, non-genotyped cells"
