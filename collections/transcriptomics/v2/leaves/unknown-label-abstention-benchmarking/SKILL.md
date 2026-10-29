---
name: unknown-label-abstention-benchmarking
description: 'Use when evaluating whether a label-transfer annotator handles truly
  unseen cell types: a cell type is deleted from the training reference and held-out
  query cells of that type should be reported as ''Unknown'' instead of receiving
  a wrong label.'
when_to_use_negative:
- All query cell types are represented in the training reference — abstention benchmarking
  is only meaningful for a deliberately deleted/unseen cell type.
- The query AnnData gene space or gene order differs from ref_adata.var_names and
  cannot be aligned — TOSICA requires identical, same-order var_names.
- The goal is absolute annotation accuracy on known labels rather than open-set recognition
  / 'Unknown' detection behavior.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TOSICA
  role: Trains the pathway-masked Transformer on the alpha-deleted hPancreas reference
    and predicts 'Unknown' vs known labels for held-out alpha cells
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: AnnData handling for subsetting the reference (excluding alpha cells) and
    aligning query genes to ref_adata.var_names
- name: torch
  role: Deep learning backend (version 1.7.1) for TOSICA model training and inference
  repo: https://github.com/JackieHanLab/TOSICA
- name: SingleR
  role: Comparator annotator that fails to abstain and incorrectly labels held-out
    alpha cells
- name: SciBet
  role: Comparator annotator that incorrectly assigns 'PP', 'delta' or 'beta' to unseen
    alpha cells
- name: ACTINN
  role: Comparator annotator that does not identify held-out alpha cells as a new
    cell type
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:e64e45733f36888b87f1b9230297dad4549e94d9c18044fbc7aff722af966337
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - ACTINN
  - SciBet
  - SingleR
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# unknown-label-abstention-benchmarking

## Summary

Benchmark whether a supervised cell-type annotator can abstain with an 'Unknown' label when a high-abundance reference cell type (e.g., alpha cells in hPancreas) is deleted from training, rather than force-mislabeling held-out cells. In the TOSICA paper, removing alpha cells from the reference leads TOSICA to flag ~76% of held-out alpha cells as 'Unknown' while SingleR, SciBet, and ACTINN incorrectly assign 'PP', 'delta', or 'beta'.

## When to use

Use when evaluating whether a label-transfer annotator handles truly unseen cell types: a cell type is deleted from the training reference and held-out query cells of that type should be reported as 'Unknown' instead of receiving a wrong label.

## When NOT to use

- All query cell types are represented in the training reference — abstention benchmarking is only meaningful for a deliberately deleted/unseen cell type.
- The query AnnData gene space or gene order differs from ref_adata.var_names and cannot be aligned — TOSICA requires identical, same-order var_names.
- The goal is absolute annotation accuracy on known labels rather than open-set recognition / 'Unknown' detection behavior.

## Inputs

- hPancreas reference AnnData (ref_adata) with cell-type labels in an obs column
- Query AnnData containing held-out cells of the deleted type (alpha cells)
- Pre-prepared gene-set mask: 'human_reactome' (reactome.gmt) or 'human_gobp' (GO_bp.gmt), or a path to a .gmt file
- Label column name in ref_adata.obs (label_key)
- Comparator annotator outputs from SingleR, SciBet, ACTINN, Cell BLAST, CHETAH, or CaSTLe (optional)

## Outputs

- new_adata.obs['Prediction'] per query cell (including 'Unknown' labels)
- new_adata.obs['Probability'] per query cell
- Attention matrix (new_adata.X) and gene2token_weights.csv
- alpha_unknown_fraction.csv with the fraction of held-out alpha cells labeled 'Unknown'
- Trained model artifacts: mask.npy, pathway.csv, label_dictionary.csv, model-n.pth

## How to apply

Load the hPancreas reference AnnData and subset it to exclude the target cell type (alpha cells), yielding ref_adata for training. Train TOSICA via TOSICA.train(ref_adata, gmt_path, project, label_name) using a pre-prepared human gene-set mask such as 'human_reactome' (reactome.gmt) or 'human_gobp' (GO_bp.gmt); this writes mask.npy, pathway.csv, label_dictionary.csv, and model-n.pth to the project folder. Align the held-out alpha-cell query AnnData to the reference gene order with query_adata = query_adata[:,ref_adata.var_names] (gene names and order must be identical between ref and query). Predict with TOSICA.pre(query_adata, project) to obtain new_adata.obs['Prediction'], new_adata.obs['Probability'], and the attention matrix. Extract the alpha-cell predictions, compute the fraction labeled 'Unknown', and compare against the reported ~76% abstention rate and against comparator tools (SingleR, SciBet, ACTINN, Cell BLAST, CHETAH, CaSTLe) that fail to abstain.

## Related tools

- **TOSICA** (Trains the pathway-masked Transformer on the alpha-deleted hPancreas reference and predicts 'Unknown' vs known labels for held-out alpha cells) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (AnnData handling for subsetting the reference (excluding alpha cells) and aligning query genes to ref_adata.var_names)
- **torch** (Deep learning backend (version 1.7.1) for TOSICA model training and inference) — https://github.com/JackieHanLab/TOSICA
- **SingleR** (Comparator annotator that fails to abstain and incorrectly labels held-out alpha cells)
- **SciBet** (Comparator annotator that incorrectly assigns 'PP', 'delta' or 'beta' to unseen alpha cells)
- **ACTINN** (Comparator annotator that does not identify held-out alpha cells as a new cell type)

## Examples

```
ref_adata = ref_adata[ref_adata.obs['labels']!='alpha']; TOSICA.train(ref_adata, 'human_reactome', project='alpha_deleted', label_name='labels'); query_adata = query_adata[:,ref_adata.var_names]; new_adata = TOSICA.pre(query_adata, project='alpha_deleted')
```

## Evaluation signals

- The computed fraction of held-out alpha cells labeled 'Unknown' should match the reported ~76%.
- Comparator tools (SingleR, SciBet, ACTINN, Cell BLAST, CHETAH, CaSTLe) should fail to abstain, instead assigning 'PP', 'delta', or 'beta' to the unseen cells.
- new_adata.obs must contain both 'Prediction' and 'Probability' columns after TOSICA.pre, and new_adata.X must be the attention matrix.
- The training step must emit mask.npy, pathway.csv, label_dictionary.csv, and model-n.pth in the project folder.
- query_adata.var_names must equal ref_adata.var_names in the same order before prediction (verified via query_adata = query_adata[:,ref_adata.var_names]).

## Limitations

- Abstention behavior is tool-specific: most supervised annotators (SingleR, SciBet, ACTINN, Cell BLAST, CHETAH, CaSTLe) force a wrong label on unseen cell types rather than outputting 'Unknown'.
- TOSICA requires the var_names (genes) of ref_adata and query_adata to be consistent and in the same order; mismatched gene spaces will break prediction.
- Results depend on the chosen gene-set mask (human_reactome, human_gobp, human_immune, human_tf, or mouse equivalents) passed as gmt_path.
- torch==1.7.1 is a pinned requirement; other versions are untested per the README environment specification.

## Evidence

- [other] With alpha cells removed from training, TOSICA labels 76% of held-out alpha cells as 'Unknown', while SingleR, SciBet and ACTINN do not identify them as a new cell type and instead incorrectly assign 'PP', 'delta' or 'beta'.: [evidence span withheld — anchor status: not_found]
- [readme] Warning that ref/query genes must match in identity and order.: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [readme] README training signature.: "TOSICA.train(ref_adata, gmt_path,project=<my_project>,label_name=<label_key>)"
- [readme] README prediction outputs.: "+ `new_adata.obs['Prediction']` : Predicted labels
+ `new_adata.obs['Probability']` : Probability of the prediction"
- [readme] Pre-prepared masks from the README.: "+ `human_gobp` : GO_bp.gmt
+ `human_immune` : immune.gmt
+ `human_reactome` : reactome.gmt
+ `human_tf` : TF.gmt"
