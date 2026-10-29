---
name: marker-based-accuracy-evaluation
description: Use when Cell BLAST annotation transfer must be validated quantitatively
  (target MBA such as 0.873) or when unassigned/rejected query cells need confirmation
  as a novel cell type via marker gene enrichment.
when_to_use_negative:
- Query datasets with fewer than ~1000 cells, where the DIRECTi model may be under-trained
  and annotation accuracy metrics are unreliable
- Cell types without established marker genes, since MBA and marker-based DE confirmation
  both depend on known markers
- Mega-scale dataset integration with highly skewed cell-type composition, where batch
  over-correction can distort querying and annotation transfer
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST
  role: Performs cell querying and annotation transfer whose accuracy is evaluated
    by MBA; rejected query cells define the novel-type cluster for DE validation
  repo: https://github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model trained on the reference to learn low-dimensional cell
    embeddings used for similarity-based annotation transfer
  repo: https://github.com/gao-lab/Cell_BLAST
- name: Cell_BLAST.blast.BLAST
  role: Queries cells against the reference using posterior-distribution-based cell-to-cell
    similarity and transfers annotations from reference hits to query cells
  repo: https://github.com/gao-lab/Cell_BLAST
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:c63e90a2a1f70ead05cabff1cd38d6916b0453b57252f25efc8265d90fd959f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gao-lab/Cell_BLAST
  tools:
  - Cell BLAST
  - Cell_BLAST.blast.BLAST
  - DIRECTi
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# marker-based-accuracy-evaluation

## Summary

Evaluate the accuracy of cell-type annotations transferred by Cell BLAST using marker-based accuracy (MBA), and use rejected/unassigned query cells plus differential expression of known markers (e.g. Cftr, Foxi1, Ascl3) to confirm putative novel cell types.

## When to use

Use when Cell BLAST annotation transfer must be validated quantitatively (target MBA such as 0.873) or when unassigned/rejected query cells need confirmation as a novel cell type via marker gene enrichment.

## When NOT to use

- Query datasets with fewer than ~1000 cells, where the DIRECTi model may be under-trained and annotation accuracy metrics are unreliable
- Cell types without established marker genes, since MBA and marker-based DE confirmation both depend on known markers
- Mega-scale dataset integration with highly skewed cell-type composition, where batch over-correction can distort querying and annotation transfer

## Inputs

- Cell BLAST query results with transferred annotations for query cells (e.g. Plasschaert airway epithelium h5ad)
- Reference-derived DIRECTi model and BLAST object (e.g. trained on ionocyte-removed Montoro reference)
- Known marker genes per cell type (e.g. Cftr, Foxi1, Ascl3 for pulmonary ionocytes)

## Outputs

- Marker-based accuracy (MBA) score for transferred annotations (target 0.873)
- Count of rejected/unassigned query cells interpreted as a novel cell type (target 14 of 19 ionocytes)
- Differential expression table for the rejected novel cluster confirming Cftr, Foxi1, and Ascl3 enrichment

## How to apply

After training a DIRECTi model on the reference (e.g. the ionocyte-removed Montoro airway epithelium panel) and querying a dataset (e.g. Plasschaert airway epithelium h5ad) with Cell_BLAST.blast.BLAST, score transferred annotations with marker-based accuracy (MBA), which checks that predicted cell types express their established marker genes. Under the default rejection cutoff, count query cells left unassigned; in the ionocyte rediscovery case Cell BLAST reached an MBA of 0.873 and correctly rejected 14 of 19 Plasschaert ionocytes, flagging them as a putative novel cell type. Then run differential expression on the rejected cluster using documented Cell_BLAST public functions and confirm enrichment of the canonical ionocyte markers Cftr, Foxi1, and Ascl3 to validate that the novel cluster is biologically real. Validation requires the reproduced MBA to equal 0.873, exactly 14/19 ionocytes rejected, and Cftr, Foxi1, and Ascl3 present as markers in the DE table.

## Related tools

- **Cell BLAST** (Performs cell querying and annotation transfer whose accuracy is evaluated by MBA; rejected query cells define the novel-type cluster for DE validation) — https://github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model trained on the reference to learn low-dimensional cell embeddings used for similarity-based annotation transfer) — https://github.com/gao-lab/Cell_BLAST
- **Cell_BLAST.blast.BLAST** (Queries cells against the reference using posterior-distribution-based cell-to-cell similarity and transfers annotations from reference hits to query cells) — https://github.com/gao-lab/Cell_BLAST

## Examples

```
import Cell_BLAST as cb; ref = cb.data.read_h5ad('montoro_no_ionocytes.h5ad'); model = cb.directi.fit_DIRECTi(ref); blast = cb.blast.BLAST(model, ref); result = blast.query(query_data); mba = marker_based_accuracy(result, markers={'ionocyte': ['Cftr','Foxi1','Ascl3']})
```

## Evaluation signals

- Reproduced MBA equals 0.873 for the ionocyte-removed Montoro reference against the Plasschaert query
- Exactly 14 of 19 Plasschaert ionocytes are rejected (unassigned) under the default cutoff
- Differential expression table for the rejected cluster contains Cftr, Foxi1, and Ascl3 as marker genes
- Annotations are transferred only to query cells with valid hits; rejected cells form a coherent putative novel cluster
- Query datasets are in h5ad format (legacy h5 converted via cb.data.h5_to_h5ad) before querying

## Limitations

- The Cell BLAST model is over-parameterized and requires a reasonable number of samples to train properly
- Datasets with fewer than 1000 cells may suffer from under-training issues; a cell number of 3000 is generally sufficient
- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition

## Evidence

- [results] MBA 0.873 with 14/19 ionocytes rejected: [evidence span withheld — anchor status: not_found]
- [abstract] Cell BLAST can identify novel cell types: "we demonstrate the effectiveness of Cell BLAST in annotating discrete cell types and continuous cell differentiation potential, as well as identifying novel cell types"
- [readme] Install and import package: "import Cell_BLAST as cb
print(cb.__version__)"
- [methods] Convert legacy h5 data to h5ad: "cb.data.h5_to_h5ad(inputfilename, outputfilename)"
- [discussion] Under-training limitation for small datasets: "a cell number of 3000 is generally sufficient for good performance, while small-sized datasets with fewer than 1000 cells may suffer from under-training issues."
