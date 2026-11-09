---
name: adt-clr-normalization
description: Use when processing CITE-Seq, TEA-Seq, or Multiome protein (ADT) counts
  that must be scaled and clustered on their own terms before multimodal integration
  with scTriangulate.
when_to_use_negative:
- Input is RNA or ATAC-seq counts — these use log-CPTT normalization with hypervariable
  gene/peak selection, not CLR
- Data already normalized for protein modality (e.g., prior CLR or scaled ADT matrix)
  — double normalization will distort downstream clustering
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTriangulate
  role: Provides preprocessing.Normalization.CLR_normalization for ADT center log-ratio
    scaling and concat_rna_and_other to merge normalized RNA/ADT/ATAC AnnData
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: Executes the per-modality PCA and Leiden clustering (15 PCs, Leiden r=1,2,3)
    on the CLR-normalized ADT data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:cb224bbc81f235af6ac4cf8fd1e004352e86edb2779a5cb9eeebf369cf1ad2c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ClusterEnsembles
  - TotalVI
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# adt-clr-normalization

## Summary

Center log-ratio (CLR) normalization of antibody-derived tag (ADT) CITE-Seq/TEA-Seq counts so protein modality data can be clustered and concatenated with RNA AnnData in the scTriangulate framework.

## When to use

Use when processing CITE-Seq, TEA-Seq, or Multiome protein (ADT) counts that must be scaled and clustered on their own terms before multimodal integration with scTriangulate.

## When NOT to use

- Input is RNA or ATAC-seq counts — these use log-CPTT normalization with hypervariable gene/peak selection, not CLR
- Data already normalized for protein modality (e.g., prior CLR or scaled ADT matrix) — double normalization will distort downstream clustering

## Inputs

- ADT/CITE-Seq count matrix (CSV of antibody-derived tag counts, e.g., TEA-Seq ADT count CSV from GSM4949911)
- QC-filtered AnnData object (e.g., min_gene >= 300, min_counts >= 500, pct_counts_mt <= 20%, min_atac_peaks >= 1000)

## Outputs

- CLR-normalized ADT AnnData (adata.X in centered log-ratio space)
- ADT Leiden annotation columns at resolutions 1, 2, 3 for input to ScTriangulate
- Combined tri-modal AnnData concatenated via concat_rna_and_other

## How to apply

For tri-modal TEA-Seq data, load the ADT count CSV (e.g., from GEO GSM4949911), QC-filter nuclei, then apply CLR normalization with the scTriangulate.preprocessing.Normalization.CLR_normalization function rather than RNA-style log-CPTT. Follow CLR with PCA (15 PCs in the paper's tri-modal benchmark) and Leiden clustering at resolutions 1, 2, and 3 to generate the ADT annotation columns. Normalize RNA and ATAC separately (log-CPTT with top-3000 HVGs/15 PCs and top-60000 peaks/100 PCs, respectively), then merge all three normalized AnnData objects using scTriangulate.preprocessing.concat_rna_and_other. The rationale is that scTriangulate integrates at the decision level and makes no assumption about raw value distributions, so each modality's normalized .X is only used for per-modality clustering and marker detection — the Shapley-value coalitional step reconciles the resulting annotation columns.

## Related tools

- **scTriangulate** (Provides preprocessing.Normalization.CLR_normalization for ADT center log-ratio scaling and concat_rna_and_other to merge normalized RNA/ADT/ATAC AnnData) — https://github.com/frankligy/scTriangulate
- **scanpy** (Executes the per-modality PCA and Leiden clustering (15 PCs, Leiden r=1,2,3) on the CLR-normalized ADT data)

## Examples

```
from sctriangulate.preprocessing.Normalization import CLR_normalization; adata = CLR_normalization(adt_adata)
```

## Evaluation signals

- ADT values in adata.X are in centered log-ratio space (per-cell geometric-mean-centered), not log-CPTT or raw counts
- Concatenated AnnData contains RNA, ATAC, and ADT layers produced via concat_rna_and_other without loss of per-modality normalization
- ADT-derived Leiden annotation columns at resolutions 1, 2, and 3 are present in adata.obs and acceptable as ScTriangulate query columns
- Downstream lazy_run on the concatenated object yields per-cell final annotations with a winning-fraction pruning threshold (win_fraction_cutoff=0.25)

## Limitations

- CLR and other default settings may not be optimal for all study designs; the article notes programmatic defaults are not guaranteed for every dataset
- Normalization choices are per-modality and optional — scTriangulate accepts counts or already-normalized matrices, so normalization must be handled upstream before annotation integration

## Evidence

- [methods] CLR_normalization: "we conducted Center log-ratio (CLR) normalization using scTriangulate.preprocessing.Normalization.CLR_normalization function."
- [methods] concat_rna_and_other: "we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other function."
- [readme] readme_normalization: [evidence span withheld — anchor status: external_doc]
- [intro] decision_level: "scTriangulate makes no assumption about the distributions of raw underlying values."
