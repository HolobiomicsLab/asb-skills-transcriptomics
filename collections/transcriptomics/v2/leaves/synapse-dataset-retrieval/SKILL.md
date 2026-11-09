---
name: synapse-dataset-retrieval
description: Use when reproducing the scTriangulate benchmarking analyses that rely
  on alternative clustering results (ICGS2, Monocle3, Seurat3, curated annotations)
  from a prior study deposited only in Synapse, not regenerable from GEO raw counts.
when_to_use_negative:
- When the target dataset is raw count matrix data available directly from GEO (e.g.,
  GSE117825, GSM4949911) or the 10x Genomics website — download those from their native
  sources instead of Synapse.
- When the goal is fresh unsupervised clustering rather than reusing prior study annotations
  — no Synapse retrieval is needed; scTriangulate can generate Leiden annotations
  itself.
- When the accession refers to a different study's artifact — syn26320732 is the benchmarking
  annotations set, whereas syn26320337 is the reproducibility folder root; pulling
  the wrong one will misalign annotation columns.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Synapse
  role: Repository hosting the pre-computed alternative clustering annotations (syn26320732)
    and the paper's reproducibility inputs/intermediate outputs (syn26320337)
  repo: https://www.synapse.org/#!Synapse:syn26320337/files/
- name: scTriangulate.preprocessing.add_annotation / add_umap
  role: Consume the downloaded Synapse annotation tables and UMAP coordinates, merging
    them into the existing AnnData object
  repo: https://github.com/frankligy/scTriangulate
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:7d27e2826e3c8daa77a1a3f17b03981acb5e52704cf4546c778bfea661ea0f42
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Synapse
  - scTriangulate
  - scTriangulate.preprocessing.add_annotation / add_umap
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# synapse-dataset-retrieval

## Summary

Retrieves pre-computed clustering annotations, UMAP coordinates, and intermediate outputs deposited in the Synapse database (accessions syn26320732 and the syn26320337 reproducibility folder) so that scTriangulate integration benchmarks can be reproduced without re-running upstream clustering tools.

## When to use

Use when reproducing the scTriangulate benchmarking analyses that rely on alternative clustering results (ICGS2, Monocle3, Seurat3, curated annotations) from a prior study deposited only in Synapse, not regenerable from GEO raw counts.

## When NOT to use

- When the target dataset is raw count matrix data available directly from GEO (e.g., GSE117825, GSM4949911) or the 10x Genomics website — download those from their native sources instead of Synapse.
- When the goal is fresh unsupervised clustering rather than reusing prior study annotations — no Synapse retrieval is needed; scTriangulate can generate Leiden annotations itself.
- When the accession refers to a different study's artifact — syn26320732 is the benchmarking annotations set, whereas syn26320337 is the reproducibility folder root; pulling the wrong one will misalign annotation columns.

## Inputs

- Synapse accession ID (syn26320732 for benchmarking clustering results; syn26320337 for the full reproducibility file folder)
- Alternative clustering annotation tables (ICGS2, Monocle3, Seurat3, curated labels) for the human bone marrow atlas
- UMAP coordinate files associated with each annotation set
- Existing AnnData (.h5ad) object with normalized expression (log CPTT) to merge annotations into

## Outputs

- Locally downloaded annotation and UMAP files from Synapse
- AnnData object with added annotation columns in adata.obs and UMAP in adata.obsm['X_umap'], ready as query inputs to ScTriangulate

## How to apply

Identify the Synapse accession cited by the paper's methods for the specific artifact: syn26320732 holds the alternative scRNA-Seq clustering results (ICGS2, Monocle3, Seurat3) and curated annotations from the prior bone-marrow benchmarking study, while the README points to syn26320337/files/ for all reproducibility inputs and intermediate outputs. Download the annotation tables and UMAP coordinate files for the relevant atlas (e.g., the Hay et al. human bone marrow atlas), then attach them to an existing AnnData object using scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap so the annotations become query columns for the ScTriangulate constructor. Verify each downloaded set matches the intended annotation source before mixing, since the whole point is to integrate decision-level label sets from independent algorithms rather than raw expression data, which is instead downloaded from GEO (e.g., GSE117825, GSM4949911).

## Related tools

- **Synapse** (Repository hosting the pre-computed alternative clustering annotations (syn26320732) and the paper's reproducibility inputs/intermediate outputs (syn26320337)) — https://www.synapse.org/#!Synapse:syn26320337/files/
- **scTriangulate.preprocessing.add_annotation / add_umap** (Consume the downloaded Synapse annotation tables and UMAP coordinates, merging them into the existing AnnData object) — https://github.com/frankligy/scTriangulate

## Evaluation signals

- All expected annotation sets (ICGS2, Monocle3, Seurat3, curated) download successfully from the cited accession with non-empty label columns.
- After add_annotation/add_umap, the AnnData contains the retrieved labels in adata.obs and adata.obsm['X_umap'] populated for every cell.
- Running ScTriangulate with the retrieved annotations (e.g., win_fraction_cutoff=0.3, add_metrics='tfidf5') completes on all input annotation sets without missing-label errors.
- The resulting agreement table against Hay et al. reference clusters reproduces the pattern reported in Supplementary Fig. 4d.
- Transitional states HSC-cycle, MEP, and LMPP are resolved in the final mix-and-match assignment, consistent with the reported bone marrow finding.

## Limitations

- Synapse access requires network connectivity and, in practice, Synapse authentication; the README notes that on Linux servers you must 'make sure you have internet connections'.
- The retrieved annotations are decision-level outputs from a prior benchmarking study — they cannot be regenerated or validated from the raw counts alone without re-running the original upstream tools.
- Annotation provenance must be tracked manually; mixing a mis-mapped annotation column into AnnData will silently corrupt the Shapley-value integration.

## Evidence

- [methods] Synapse syn26320732 accession for benchmarking annotations: "using results deposited in the Synapse database (syn26320732)."
- [readme] README reproducibility folder on Synapse: "along 
with all the necessary input files and intermediate outputs which are avaiable in [Synapse storage](https://www.synapse.org/#!Synapse:syn26320337/files/)."
- [methods] Merging retrieved annotations into AnnData: "The obtained annotations and umap coordinates were added to the existing AnnData by scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap functions"
- [readme] Retrieved sets serve as conflicting inputs to integration: [evidence span withheld — anchor status: external_doc]
