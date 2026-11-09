---
name: gene-set-mask-construction
description: Use when training TOSICA on a reference AnnData and you must pick or
  construct the gmt-path mask (human_gobp, human_immune, human_reactome, human_tf,
  mouse_gobp, mouse_reactome, mouse_tf, or a custom .gmt path) that defines gene-to-pathway-token
  connectivity.
when_to_use_negative:
- ref_adata and query_adata var_names differ in genes or order — the mask is defined
  on reference genes, so prediction requires query_adata = query_adata[:,ref_adata.var_names]
  first
- Reference genes use identifiers absent from the chosen .gmt gene sets, so no gene-to-token
  connections can be formed
- The task is unsupervised clustering without a reference — the mask supports reference-to-query
  label transfer, which the paper contrasts with clustering-plus-manual-annotation
  workflows
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: TOSICA
  role: TOSICA.train consumes gmt_path (pre-prepared mask name or .gmt file path)
    to map input genes to gene sets and write mask.npy and pathway.csv that define
    the masked linear embedding
  repo: https://github.com/JackieHanLab/TOSICA
- name: torch
  role: Implements the masked linear transformation weight (W) that embeds n genes
    into k pathway tokens before multi-head self-attention
- name: scanpy
  role: Loads the reference AnnData (ref_adata) whose var_names define the mask rows
    and must be matched by the query
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:5f49b23ecbf0a985bb1ff60652513d2ad4348b4659cd7ed1d5480c6c88883472
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gene-set-mask-construction

## Summary

Builds the gene-set mask that maps a reference scRNA-seq gene matrix onto pathway/regulon tokens for TOSICA's masked linear embedding, using the repo's pre-prepared human/mouse .gmt masks (GO_bp, Reactome, immune, TF) or custom GSEA/MSigDB gene sets. This mask is what makes the attention layers traceable and interpretable down to individual genes.

## When to use

Use when training TOSICA on a reference AnnData and you must pick or construct the gmt-path mask (human_gobp, human_immune, human_reactome, human_tf, mouse_gobp, mouse_reactome, mouse_tf, or a custom .gmt path) that defines gene-to-pathway-token connectivity.

## When NOT to use

- ref_adata and query_adata var_names differ in genes or order — the mask is defined on reference genes, so prediction requires query_adata = query_adata[:,ref_adata.var_names] first
- Reference genes use identifiers absent from the chosen .gmt gene sets, so no gene-to-token connections can be formed
- The task is unsupervised clustering without a reference — the mask supports reference-to-query label transfer, which the paper contrasts with clustering-plus-manual-annotation workflows

## Inputs

- Reference AnnData object (ref_adata) with gene var_names matching the .gmt gene symbols
- Pre-prepared mask name (human_gobp, human_immune, human_reactome, human_tf, mouse_gobp, mouse_reactome, mouse_tf) or a path to custom .gmt files (e.g. GO_bp.gmt, reactome.gmt, immune.gmt, TF.gmt, m_GO_bp.gmt, m_reactome.gmt, m_TF.gmt, c2.cp.reactome.v7.5.1.symbols.gmt, c3.all.v7.5.1.symbols.gmt)
- Label column name in ref_adata.obs (label_key)

## Outputs

- ./<my_project>/mask.npy — the gene-by-token mask matrix
- ./<my_project>/pathway.csv — the gene set (token) list
- Gene-to-token connectivity later emitted at prediction time as gene2token_weights.csv (weights matrix of genes to tokens)

## How to apply

Choose a mask source: a pre-prepared mask shipped with the repo (human_gobp=GO_bp.gmt, human_immune=immune.gmt, human_reactome=reactome.gmt, human_tf=TF.gmt, mouse_gobp=m_GO_bp.gmt, mouse_reactome=m_reactome.gmt, mouse_tf=m_TF.gmt) or a custom path to GSEA/MSigDB .gmt files such as c2.cp.reactome.v7.5.1.symbols.gmt or c3.all.v7.5.1.symbols.gmt. Pass gmt_path together with the reference AnnData to TOSICA.train, which maps the input genes to the selected gene sets and uses the resulting mask as the linear transformation weight (W) that embeds n genes into k pathway/regulon tokens. Confirm that training writes ./<my_project>/mask.npy (the mask matrix) and pathway.csv (the gene set list) alongside label_dictionary.csv and model-n.pth. Genes must be symbols that appear in the .gmt, and ref_adata.var_names must be fixed, because the README requires query_adata = query_adata[:,ref_adata.var_names] before prediction. For ablation studies, build random masks matched in parameter count to the knowledge-based mask (e.g. reserving 1% and 5% of gene-to-token connections) and record epochs-to-convergence and converged accuracy; expect random masks to approach knowledge-mask accuracy but need more epochs, and to converge lower on mPancreas.

## Related tools

- **TOSICA** (TOSICA.train consumes gmt_path (pre-prepared mask name or .gmt file path) to map input genes to gene sets and write mask.npy and pathway.csv that define the masked linear embedding) — https://github.com/JackieHanLab/TOSICA
- **torch** (Implements the masked linear transformation weight (W) that embeds n genes into k pathway tokens before multi-head self-attention)
- **scanpy** (Loads the reference AnnData (ref_adata) whose var_names define the mask rows and must be matched by the query)

## Examples

```
TOSICA.train(ref_adata, 'human_reactome', project='my_project', label_name='CellType')
```

## Evaluation signals

- After TOSICA.train, ./<my_project>/mask.npy and ./<my_project>/pathway.csv exist, and pathway.csv entries correspond to gene sets present in the source .gmt file
- The mask matrix has one row per reference gene in ref_adata.var_names and one column per gene-set token from the chosen gmt
- At prediction time, ./my_project/gene2token_weights.csv is produced giving the weights matrix of genes to tokens, confirming the mask wired the embedding
- Predicted labels in new_adata.obs['Prediction'] are all present in the label_dictionary.csv generated during training
- For random-mask controls, verify parameter count matches the knowledge-based mask (e.g. 1% and 5% reserved gene-to-token connections) before comparing epochs-to-convergence and converged accuracy

## Limitations

- README warning: the var_names (genes) of ref_adata and query_adata must be consistent and in the same order, or predictions are invalid
- Random masks with the same parameter count usually achieve accuracy similar to knowledge-based masks, so the mask's main benefit is interpretability and convergence; on mPancreas the random mask converged to lower accuracy
- Number of attention heads (H), learning rate, batch size and random seed are not reported in the methods, limiting exact reproduction of mask-dependent training
- Pre-prepared masks cover only human and mouse GO BP, Reactome, immune and TF sets; other species or ontologies require custom .gmt files sourced from GSEA/MSigDB

## Evidence

- [discussion] genes mapped to gmt gene sets: "we map the input genes to selected gene sets (gmt files)"
- [discussion] masked embedding of genes into tokens: [evidence span withheld — anchor status: not_found]
- [discussion] rationale for knowledge-based masks: "The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons"
- [readme] mask source options: "`gmt_path` : default pre-prepared mask or path to .gmt files."
- [readme] mask artifacts written by training: "+ `./my_project/mask.npy` : Mask matrix
+ `./my_project/pathway.csv` : Gene set list"
