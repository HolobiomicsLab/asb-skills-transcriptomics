---
name: novel-cell-type-rejection-analysis
description: Use when a query scRNA-seq dataset (h5ad) may contain cell types absent
  from the curated reference, and you want annotation transfer to correctly reject
  those cells so the unassigned cluster can be characterized by marker genes.
when_to_use_negative:
- The reference dataset already contains the cell type of interest — annotations will
  simply be transferred and no novel-type rejection can occur
- The query dataset has fewer than ~1000 cells — DIRECTi is over-parameterized and
  small datasets may suffer from under-training issues
- The goal is mega-scale dataset integration with highly skewed cell-type composition,
  which the authors note risks over-correcting batches
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST
  role: Cell-querying framework whose default cutoff rejects query cells not matching
    reference annotations, flagging putative novel cell types
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model trained on the type-removed reference to produce unbiased
    cell embeddings for similarity estimation
  repo: github.com/gao-lab/Cell_BLAST
- name: Cell_BLAST.blast.BLAST
  role: Object built from the fitted DIRECTi model used to query the Plasschaert cells
    and transfer (or withhold) reference annotations
  repo: github.com/gao-lab/Cell_BLAST
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:67b2f193dcc0758f488e8fa75ed5d89c928b3f700adb95ec562514d8bd6aa20e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - Cell_BLAST.blast.BLAST
  - DIRECTi
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# novell-cell-type-rejection-analysis

## Summary

Detects putative novel cell types by querying a dataset against a reference that excludes the candidate type: Cell BLAST's DIRECTi model and cell-to-cell similarity metric leave cells of the missing type unassigned ('rejected'), and differential expression on the rejected cluster confirms its identity.

## When to use

Use when a query scRNA-seq dataset (h5ad) may contain cell types absent from the curated reference, and you want annotation transfer to correctly reject those cells so the unassigned cluster can be characterized by marker genes.

## When NOT to use

- The reference dataset already contains the cell type of interest — annotations will simply be transferred and no novel-type rejection can occur
- The query dataset has fewer than ~1000 cells — DIRECTi is over-parameterized and small datasets may suffer from under-training issues
- The goal is mega-scale dataset integration with highly skewed cell-type composition, which the authors note risks over-correcting batches

## Inputs

- Reference scRNA-seq dataset in h5ad format with the candidate cell type excluded (e.g. Montoro airway epithelium minus ionocytes)
- Query scRNA-seq dataset in h5ad format containing cells of the potentially novel type (e.g. Plasschaert airway epithelium)
- Candidate marker genes for validating the rejected cluster (e.g. Cftr, Foxi1, Ascl3 for pulmonary ionocytes)

## Outputs

- Per-query-cell annotation transfers with hit lists and similarity scores, including rejected (unassigned) cells
- Marker-based accuracy (MBA) of transferred annotations
- Count/identity of rejected query cells forming a putative novel cell type cluster
- Differential expression table for the rejected cluster confirming marker enrichment

## How to apply

Build a reference database in h5ad format with the candidate cell type deliberately removed (e.g. the Montoro airway epithelium reference without ionocytes), then train a DIRECTi deep generative model on it with Cell_BLAST.directi.fit_DIRECTi to learn unbiased low-dimensional cell embeddings that handle batch effect. Wrap the fitted model in a Cell_BLAST.blast.BLAST object and query the cells of interest (e.g. the Plasschaert airway epithelium dataset) against the reference, estimating cell-to-cell similarity as the distance between posterior distributions and transferring reference annotations only to query cells whose similarity passes the default cutoff. Cells exceeding the cutoff-free rejection zone remain unassigned; collect them as a putative novel cell type cluster. In the published ionocyte case this correctly rejected 14 of 19 Plasschaert ionocytes while achieving an MBA of 0.873 on the assignable cells. Confirm the rejected cluster's identity by differential expression using Cell_BLAST's public functions, checking for expected markers — for pulmonary ionocytes, enrichment of Cftr, Foxi1, and Ascl3. For reproduction, check out the `rep` branch of the Cell_BLAST repository.

## Related tools

- **Cell BLAST** (Cell-querying framework whose default cutoff rejects query cells not matching reference annotations, flagging putative novel cell types) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model trained on the type-removed reference to produce unbiased cell embeddings for similarity estimation) — github.com/gao-lab/Cell_BLAST
- **Cell_BLAST.blast.BLAST** (Object built from the fitted DIRECTi model used to query the Plasschaert cells and transfer (or withhold) reference annotations) — github.com/gao-lab/Cell_BLAST

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST; import Cell_BLAST as cb
```

## Evaluation signals

- Reproduced marker-based accuracy (MBA) equals 0.873 on the ionocyte-removed Montoro reference against the Plasschaert query
- Exactly 14 of 19 Plasschaert ionocytes are rejected (left unassigned) under the default cutoff
- Differential expression table for the rejected cluster contains Cftr, Foxi1, and Ascl3 as enriched marker genes
- Query cells of known reference types are still annotated with high agreement, indicating rejection is specific to the missing type
- Package imports cleanly (import Cell_BLAST as cb) before running the workflow

## Limitations

- The model is over-parameterized (like all neural network-based models) and requires a reasonable number of samples; datasets with fewer than 1000 cells may suffer from under-training, while 3000 cells are generally sufficient
- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition
- Only the pip installation path is currently supported for the Python package

## Evidence

- [intro] identify novel cell types via two case studies: "use two case studies to demonstrate that Cell BLAST can further be utilized to predict continuous cell-differentiation potential and identify novel cell types"
- [methods] DIRECTi deep generative model: "a deep generative model (currently named **DIRECTi**)"
- [methods] import Cell_BLAST as cb: "import Cell_BLAST as cb"
- [readme] package import check: [evidence span withheld — anchor status: external_doc]
- [readme] reproduce paper results: [evidence span withheld — anchor status: external_doc]
