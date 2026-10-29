---
name: single-cell-rna-seq-reference-subsetting
description: 'Use when evaluating whether a reference-based cell type annotator can
  flag unseen cell types as ''Unknown'': remove one cell type from the training reference
  and check predictions on held-out cells of that type.'
when_to_use_negative:
- The goal is standard annotation of query cells whose types are all represented in
  the reference — subsetting would only remove training signal and degrade accuracy.
- The analysis is unsupervised clustering or de novo discovery without any reference
  dataset, since this skill presupposes a labeled training reference.
- The query and reference gene spaces cannot be reconciled — TOSICA requires query_adata
  = query_adata[:,ref_adata.var_names], so mismatched var_names must be fixed before
  this workflow is meaningful.
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: TOSICA
  role: Trains on the subset reference (TOSICA.train) and predicts on held-out cells
    (TOSICA.pre), exposing obs['Prediction'] and obs['Probability'] used to compute
    the Unknown fraction
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Used to load, subset and manipulate the reference/query AnnData objects (e.g.
    excluding alpha cells and reordering var_names)
- name: torch
  role: Deep-learning backend (torch 1.7.1) on which the TOSICA Transformer model
    is trained on the subset reference
  repo: https://github.com/JackieHanLab/TOSICA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:4202f0852a3f66f5493bcf7cadf83a359997b15070f16be52cd9ea6e020b45ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-rna-seq-reference-subsetting

## Summary

Deliberately delete a high-abundance cell type (e.g. alpha cells from the hPancreas reference AnnData) before training an annotator so that held-out cells of the deleted type can be used to test whether the tool abstains ('Unknown') instead of force-labeling.

## When to use

Use when evaluating whether a reference-based cell type annotator can flag unseen cell types as 'Unknown': remove one cell type from the training reference and check predictions on held-out cells of that type.

## When NOT to use

- The goal is standard annotation of query cells whose types are all represented in the reference — subsetting would only remove training signal and degrade accuracy.
- The analysis is unsupervised clustering or de novo discovery without any reference dataset, since this skill presupposes a labeled training reference.
- The query and reference gene spaces cannot be reconciled — TOSICA requires query_adata = query_adata[:,ref_adata.var_names], so mismatched var_names must be fixed before this workflow is meaningful.

## Inputs

- Reference AnnData object (e.g. hPancreas) with cell type labels in ref_adata.obs (label_name column)
- Held-out query AnnData containing cells of the deleted type (alpha cells)
- Gene-set mask for training (e.g. human_reactome reactome.gmt or human_gobp GO_bp.gmt, or a path to a .gmt file)

## Outputs

- Subset training reference (ref_adata) excluding the deleted cell type
- Trained model artifacts: project folder with mask.npy, pathway.csv, label_dictionary.csv, model-n.pth
- new_adata.obs['Prediction'], new_adata.obs['Probability'] and attention matrix for the query cells
- Fraction of held-out cells labeled 'Unknown' (e.g. alpha_unknown_fraction.csv)

## How to apply

Load the reference AnnData (e.g. hPancreas) and subset out one high-abundance cell type — here alpha cells — yielding ref_adata for training; this simulates a query set containing a genuinely novel cell type. Train the annotator on the reduced reference (TOSICA.train with a gene-set mask such as human_reactome or human_gobp, which writes mask.npy, pathway.csv, label_dictionary.csv and model-n.pth to the project folder). Align the held-out query AnnData to the reference gene order with query_adata = query_adata[:,ref_adata.var_names], since TOSICA requires identical var_names and order. Run TOSICA.pre to obtain new_adata.obs['Prediction'], new_adata.obs['Probability'] and the attention matrix, then compute the fraction of the deleted cell type labeled 'Unknown' (reported ~76% for TOSICA) and export it (e.g. alpha_unknown_fraction.csv). High abstention on the deleted type plus no abstention on known types indicates the skill was applied correctly; comparators such as SingleR, SciBet and ACTINN instead mislabel these cells as 'PP', 'delta' or 'beta'.

## Related tools

- **TOSICA** (Trains on the subset reference (TOSICA.train) and predicts on held-out cells (TOSICA.pre), exposing obs['Prediction'] and obs['Probability'] used to compute the Unknown fraction) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Used to load, subset and manipulate the reference/query AnnData objects (e.g. excluding alpha cells and reordering var_names))
- **torch** (Deep-learning backend (torch 1.7.1) on which the TOSICA Transformer model is trained on the subset reference) — https://github.com/JackieHanLab/TOSICA

## Examples

```
ref_adata = adata[adata.obs['celltype']!='alpha'].copy(); TOSICA.train(ref_adata, gmt_path='human_reactome', project='pancreas_no_alpha', label_name='celltype'); new_adata = TOSICA.pre(query_adata[:,ref_adata.var_names], project='pancreas_no_alpha')
```

## Evaluation signals

- The computed Unknown fraction for held-out alpha cells matches the reported ~76% labeled 'Unknown' by TOSICA.
- Trained project folder contains the expected artifacts: mask.npy, pathway.csv, label_dictionary.csv and model-n.pth.
- Query AnnData var_names exactly match ref_adata.var_names in the same order after the subsetting alignment step.
- Comparator annotators (SingleR, SciBet, ACTINN, Cell BLAST, CHETAH, CaSTLe) fail to abstain and instead assign 'PP', 'delta' or 'beta' to the deleted type.
- Results table (alpha_unknown_fraction.csv) is written with per-cell predictions and the aggregate abstention rate.

## Limitations

- The test requires a high-abundance cell type to delete; removing a rare type may not yield a statistically meaningful abstention estimate.
- The article does not report random seeds or batch size for training, so exact reproduction of the 76% Unknown fraction may vary run to run.
- Gene spaces of reference and query must be identical and identically ordered; any mismatch silently breaks prediction.
- Comparator tools benchmarked (SingleR, SciBet, ACTINN, Cell BLAST, CHETAH, CaSTLe) force a label and cannot natively output 'Unknown', so the subsetting test is only informative for abstention-capable annotators like TOSICA.

## Evidence

- [results] With alpha cells removed from training, TOSICA labels 76% of held-out alpha cells as 'Unknown', while SingleR, SciBet and ACTINN do not identify them as a new cell type: [evidence span withheld — anchor status: not_found]
- [readme] the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order.: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order. query_adata = query_adata[:,ref_adata.var_names]"
- [readme] TOSICA.train(ref_adata, gmt_path, project=<my_project>, label_name=<label_key>): "TOSICA.train(ref_adata, gmt_path,project=<my_project>,label_name=<label_key>)"
- [readme] + `./my_project/mask.npy` : Mask matrix: "+ `./my_project/mask.npy` : Mask matrix
+ `./my_project/pathway.csv` : Gene set list
+ `./my_project/label_dictionary.csv` : Label list
+ `./my_project/model-n.pth` : Weights"
