---
name: auroc-benchmark-evaluation-against-curated-markers
description: Use when validating or reproducing marker gene identification quality
  for Cell Ontology terms — e.g. checking whether top-20 OnClass-ranked markers recover
  curated marker sets via AUROC for seen vs unseen cell types in Muris FACS/droplet
  data.
when_to_use_negative:
- The task is cell type annotation/classification accuracy rather than marker gene
  validation — use OnClass cross-validation (run_one_dataset_cross_validation.py)
  or cross-dataset prediction (run_cross_dataset_prediction.py) instead.
- You have no OnClass prediction matrices (pred_Y_all) — the marker ranking depends
  on Spearman correlation against each cell type's predicted probability vector, so
  de novo marker discovery without OnClass outputs is out of scope.
- You need to distinguish topologically identical unseen cell types (siblings sharing
  the same parents in the Cell Ontology graph) — OnClass cannot separate them, so
  per-term AUROCs are uninformative for such pairs.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: OnClass
  role: Generates the pred_Y_seen/pred_Y_all probability matrices and provides run_marker_genes_identification.py
    and run_marker_gene_based_prediction.py used to rank markers and build marker-gene-based
    predictions
  repo: https://github.com/wangshenguiuc/OnClass
- name: Python 3.6
  role: Required runtime for executing the OnClass marker gene identification and
    evaluation scripts
- name: tensorflow 2.0
  role: Backend for the OnClass model that produces the prediction probability matrices
    consumed by the marker gene scripts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:e16afb84a9a55b2a0b3f4cbd3e5db11db903343456c3939b88b5b97bbe01b5a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - Python 3.6
  - TensorFlow
  - tensorflow 2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# auroc-benchmark-evaluation-against-curated-markers

## Summary

Reproduces OnClass's marker gene identification benchmark: Spearman-rank all genes against each cell type's predicted probability vector in pred_Y_all, take the top 20 markers per Cell Ontology term, and compute AUROC against curated marker gene sets, stratified for seen and unseen terms in Tabula Muris FACS and droplet (0.87–0.97 seen; 0.87 unseen FACS; 0.82 unseen droplet).

## When to use

Use when validating or reproducing marker gene identification quality for Cell Ontology terms — e.g. checking whether top-20 OnClass-ranked markers recover curated marker sets via AUROC for seen vs unseen cell types in Muris FACS/droplet data.

## When NOT to use

- The task is cell type annotation/classification accuracy rather than marker gene validation — use OnClass cross-validation (run_one_dataset_cross_validation.py) or cross-dataset prediction (run_cross_dataset_prediction.py) instead.
- You have no OnClass prediction matrices (pred_Y_all) — the marker ranking depends on Spearman correlation against each cell type's predicted probability vector, so de novo marker discovery without OnClass outputs is out of scope.
- You need to distinguish topologically identical unseen cell types (siblings sharing the same parents in the Cell Ontology graph) — OnClass cannot separate them, so per-term AUROCs are uninformative for such pairs.

## Inputs

- pred_Y_seen.released.npy (OnClass prediction matrix for seen cell types)
- pred_Y_all.released.npy (OnClass prediction matrix for all Cell Ontology terms, including unseen)
- Annotated single-cell gene expression matrix (all FACS cells from Tabula Muris Senis)
- Cell Ontology files from the OnClass figshare deposits
- Curated marker gene sets per cell type (ground truth for AUROC evaluation)

## Outputs

- Ranked list of top 20 marker genes per Cell Ontology term (seen and unseen)
- Marker-gene-based cell type predictions (Figure 5d,e,f, Extended Data Figure 7)
- Per-cell-type AUROC values stratified by seen vs unseen terms for Muris FACS and Muris droplet

## How to apply

Load the intermediate OnClass prediction matrices pred_Y_seen.released.npy and pred_Y_all.released.npy produced by run_generate_pretrained_model.py, together with the annotated single-cell expression data (all FACS cells from Tabula Muris Senis) and the Cell Ontology files from the figshare deposits. Run run_marker_genes_identification.py, which computes the Spearman correlation between each cell type's predicted probability vector in pred_Y_all and each gene's expression profile to rank all genes per cell type, then selects exactly the top 20 marker genes per cell type, mirroring OnClass's precomputation strategy. Run run_marker_gene_based_prediction.py to build marker-gene-based cell type predictions (Figure 5d,e,f and Extended Data Figure 7). Finally, compute AUROC per cell type by evaluating the identified markers against curated marker gene sets, stratifying results for seen Cell Ontology terms and unseen terms in the Tabula Muris FACS and droplet datasets. Acceptance: seen-term AUROCs should fall in the reported 0.87–0.97 range, unseen terms should reach ~0.87 in Muris FACS and ~0.82 in Muris droplet, and every cell type must receive exactly 20 ranked marker genes.

## Related tools

- **OnClass** (Generates the pred_Y_seen/pred_Y_all probability matrices and provides run_marker_genes_identification.py and run_marker_gene_based_prediction.py used to rank markers and build marker-gene-based predictions) — https://github.com/wangshenguiuc/OnClass
- **Python 3.6** (Required runtime for executing the OnClass marker gene identification and evaluation scripts)
- **tensorflow 2.0** (Backend for the OnClass model that produces the prediction probability matrices consumed by the marker gene scripts)

## Examples

```
python script/run_generate_pretrained_model.py && python script/run_marker_genes_identification.py && python script/run_marker_gene_based_prediction.py
```

## Evaluation signals

- Seen Cell Ontology term AUROCs fall within the reported 0.87–0.97 range
- Unseen term AUROCs are ~0.87 for Muris FACS and ~0.82 for Muris droplet
- Every cell type receives exactly 20 ranked marker genes
- AUROC is computed per cell type against curated marker gene sets and correctly stratified into seen vs unseen terms for both Muris FACS and droplet
- Resulting plots reproduce Figure 5d,e,f and Extended Data Figure 7

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph, so AUROCs for such sibling terms are not meaningful.
- Most Cell Ontology terms are not associated with curated marker genes, limiting the ground truth available for evaluation.
- Marker genes are precomputed based on all FACS cells from Tabula Muris Senis, so results are dataset- and platform-specific (FACS vs droplet).
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states, bounding the term space that can be evaluated.

## Evidence

- [abstract] OnClass can be used to identify marker genes for all the cell ontology categories, regardless of whether the cell types are present or absent in the training data: "OnClass can be used to identify marker genes for all the cell ontology categories, regardless of whether the cell types are present or absent in the training data"
- [methods] Marker genes are precomputed based on all FACS cells from Tabula Muris Senis.: "Marker genes are precomputed based on all FACS cells from Tabula Muris Senis."
- [methods] Then run `run_marker_genes_identification.py <https://github.com/wangshenguiuc/OnClass/blob/master/script/run_marker_genes_identification.py>`__: "Then run `run_marker_genes_identification.py <https://github.com/wangshenguiuc/OnClass/blob/master/script/run_marker_genes_identification.py>`__"
- [results] OnClass achieved an AUROC of 0.87 for unseen Cell Ontology terms in Muris FACS and an AUROC of 0.82 for unseen Cell Ontology terms in Muris droplet: [evidence span withheld — anchor status: not_found]
- [discussion] Another limitation of OnClass is that it cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph.: "Another limitation of OnClass is that it cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph."
