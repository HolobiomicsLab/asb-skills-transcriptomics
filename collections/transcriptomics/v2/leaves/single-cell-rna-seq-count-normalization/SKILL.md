---
name: single-cell-rna-seq-count-normalization
description: Use when a sparsely-connected autoencoder (SCA) or similar cluster-level
  analysis must be run on a raw UMI counts matrix and you need to test whether input
  normalization changes cluster quality metrics or near-threshold cluster calls.
when_to_use_negative:
- When the input is already a log-space or normalized expression table rather than
  a raw counts table (re-normalizing again is not what the QCM/QCC sensitivity study
  evaluates)
- When the goal is bulk RNA-seq normalization — this skill targets single-cell UMI
  counts where TMM/RLE global-scaling variants are preferred
- 'When clusters are far from the 0.5 QCM/QCC threshold: the study showed normalization
  mainly alters near-threshold clusters (2 and 3), so comparisons on stable low-quality
  clusters are uninformative'
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: rCASC
  role: Hosts counts2log (log10 conversion), checkCountDepth (SCnorm wrapper) and
    the SCA autoencoder/autoencoderAnalysis module used to score each normalized matrix
  repo: https://github.com/kendomaniac/rCASC
- name: Docker
  role: Execution environment required for the rCASC containers in which normalization
    and SCA runs are performed
  repo: https://github.com/kendomaniac/rCASC
- name: SCnorm
  role: Count-depth assessment of the UMI data before choosing a normalization strategy,
    accessed via the rCASC checkCountDepth wrapper
  repo: https://github.com/rhondabacher/SCnorm
- name: R
  role: Language in which rCASC normalization functions and the SCA workflow are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:582d16508dc38514e1f1ea6d224409ac98e0ac2ab225e3b3b7236c2cb3eedcd3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/rCASC
  tools:
  - Docker
  - R
  - SCnorm
  - devtools
  - rCASC
  - scCrossTalk
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-rna-seq-count-normalization

## Summary

Generate alternative normalized versions (log10 transformation plus CLR, RLE, FQ, SUM and TMM global-scaling) of a single-cell UMI counts table before running SCA, and assess how normalization shifts the QCM/QCC scores of clusters sitting near the 0.5 significance threshold.

## When to use

Use when a sparsely-connected autoencoder (SCA) or similar cluster-level analysis must be run on a raw UMI counts matrix and you need to test whether input normalization changes cluster quality metrics or near-threshold cluster calls.

## When NOT to use

- When the input is already a log-space or normalized expression table rather than a raw counts table (re-normalizing again is not what the QCM/QCC sensitivity study evaluates)
- When the goal is bulk RNA-seq normalization — this skill targets single-cell UMI counts where TMM/RLE global-scaling variants are preferred
- When clusters are far from the 0.5 QCM/QCC threshold: the study showed normalization mainly alters near-threshold clusters (2 and 3), so comparisons on stable low-quality clusters are uninformative

## Inputs

- Raw single-cell UMI counts matrix (e.g., setA PBMC-derived input counts table)
- Cluster assignments for each cell (clusters 1–5, e.g., cluster 2 = Monocytes, cluster 3 = HSC)
- Experimentally validated miRNA-target gene sets from miRTarBase used as SCA hidden-layer structure

## Outputs

- Six normalized counts matrices (log10, CLR, RLE, FQ, SUM, TMM)
- Per-cluster QCM and QCC scores for each normalization run
- Cluster-specific ranked markers (e.g., miRNAs) from the SCA encoded space per normalization

## How to apply

Load the single-cell counts matrix together with its cluster assignments (e.g., the setA PBMC-derived data with clusters 1–5, where cluster 2 is Monocytes and cluster 3 is HSC). Produce six normalized versions of the input counts: a log10 transformation (rCASC counts2log) followed by CLR, RLE, FQ, SUM and TMM global-scaling normalizations, noting that TMM and RLE are the variants suited to UMI data. For each normalized matrix, run SCA via the rCASC autoencoder/autoencoderAnalysis functions inside Docker containers, then compute QCM (Quality Control of Model) and QCC (Quality Control of Cluster) per cluster per run. Accept the result when clusters near the 0.5 QCM/QCC threshold (clusters 2 and 3) show the expected cross-normalization variation while low-quality clusters (1, 4, 5) remain minimally affected, and top-ranked miRNAs (miR-191/miR-132-3p under TMM; miR-187-3p under FQ/SUM) stay consistent with the reference figures/tables. Optionally check library-size/count-depth behavior first with the rCASC checkCountDepth wrapper for the SCnorm package.

## Related tools

- **rCASC** (Hosts counts2log (log10 conversion), checkCountDepth (SCnorm wrapper) and the SCA autoencoder/autoencoderAnalysis module used to score each normalized matrix) — https://github.com/kendomaniac/rCASC
- **Docker** (Execution environment required for the rCASC containers in which normalization and SCA runs are performed) — https://github.com/kendomaniac/rCASC
- **SCnorm** (Count-depth assessment of the UMI data before choosing a normalization strategy, accessed via the rCASC checkCountDepth wrapper) — https://github.com/rhondabacher/SCnorm
- **R** (Language in which rCASC normalization functions and the SCA workflow are executed)

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC")
```

## Evaluation signals

- Six normalized matrices are produced, corresponding to log10 plus the five normalizations CLR, RLE, FQ, SUM, TMM
- QCM and QCC are computed per cluster per normalization, and clusters 2/3 change across runs while staying near the 0.5 threshold
- Clusters 1, 4 and 5 show minimal QCM/QCC variation across the six normalizations
- Top-ranked miRNAs reproduce the reference marker set: miR-191 and miR-132-3p under TMM; miR-187-3p under FQ and SUM
- Every SCA run is executed inside the rCASC Docker containers, matching the reproducibility setup of the SCAtutorial

## Limitations

- Normalization only reliably alters near-threshold clusters; clusters with firmly low QCM/QCC (1, 4, 5) are minimally affected, so normalization is not a rescue for poor-quality clusters
- TMM and RLE are the normalization variants suited to UMI data; other global-scaling variants may not be appropriate for sparse counts
- SCnorm is not intended for very sparse data: it is not recommended for datasets with more than ~80% zero counts and K may fail to converge, though ditherCounts = TRUE may help with tied counts
- The rCASC SCA workflow requires a 64-bit Linux workstation/server with the Docker daemon installed

## Evidence

- [abstract] QCM and QCC quantify SCA reconstruction quality: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [methods] log10 conversion of counts table: "Converting a count table in log10: counts2log"
- [methods] checkCountDepth SCnorm wrapper in rCASC: "checkCountDepth provides a wrapper, in rCASC, for the checkCountDepth of the [*SCnorm package*]"
- [readme] SCnorm sparse-data limitation: [evidence span withheld — anchor status: external_doc]
- [readme] Docker requirement for rCASC workflows: [evidence span withheld — anchor status: not_found]
