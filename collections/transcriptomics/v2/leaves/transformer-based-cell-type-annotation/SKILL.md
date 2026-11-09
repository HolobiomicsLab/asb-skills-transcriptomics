---
name: transformer-based-cell-type-annotation
description: Use when a reference scRNA-seq AnnData with known cell labels must annotate
  an unlabeled query dataset, when rare/unseen cell types should be flagged as 'Unknown',
  or when batch-insensitive integration with interpretable attention is needed.
when_to_use_negative:
- The reference and query AnnData have different or unordered var_names (genes) —
  the README warns they must be consistent and in the same order; run the subsetting
  step first
- You need unsupervised de novo clustering without a labeled reference — TOSICA is
  a label-transfer tool between a reference and a query dataset
- Seurat-based comparator methods are required for very large numbers of batches (they
  were inapplicable for >85 batches in the pan-cancer scIB benchmark)
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: TOSICA
  role: 'Primary model: pathway-masked Transformer for cell type annotation, ''Unknown''
    abstention, and attention-based integration'
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Loads reference/query AnnData, gene reordering, and downstream PCA/UMAP on
    the normalized attention matrix
  repo: https://github.com/JackieHanLab/TOSICA
- name: torch
  role: Deep learning backend (version 1.7.1) for the multi-head self-attention training
    and prediction
  repo: https://github.com/JackieHanLab/TOSICA
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:06abbc2d7eeb055919a7b1e80c024d1c55abde5234aebfbf1f1f6da939f712dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# transformer-based-cell-type-annotation

## Summary

TOSICA is a multi-head self-attention Transformer model for one-stop interpretable cell type annotation and label transfer from a reference scRNA-seq dataset to a query dataset, using pathway/regulon-masked tokens so attention layers remain traceable to input genes.

## When to use

Use when a reference scRNA-seq AnnData with known cell labels must annotate an unlabeled query dataset, when rare/unseen cell types should be flagged as 'Unknown', or when batch-insensitive integration with interpretable attention is needed.

## When NOT to use

- The reference and query AnnData have different or unordered var_names (genes) — the README warns they must be consistent and in the same order; run the subsetting step first
- You need unsupervised de novo clustering without a labeled reference — TOSICA is a label-transfer tool between a reference and a query dataset
- Seurat-based comparator methods are required for very large numbers of batches (they were inapplicable for >85 batches in the pan-cancer scIB benchmark)

## Inputs

- Reference AnnData (ref_adata) with cell type labels in ref_adata.obs[label_key]
- Query AnnData (query_adata) with genes matched to ref_adata.var_names
- Gene-set mask: pre-prepared name (human_gobp, human_immune, human_reactome, human_tf, mouse_gobp, mouse_reactome, mouse_tf) or path to a .gmt file (e.g. reactome.gmt, GO_bp.gmt, immune.gmt, TF.gmt)

## Outputs

- Project folder with mask.npy, pathway.csv, label_dictionary.csv, model-n.pth
- new_adata.X: attention matrix per query cell
- new_adata.obs['Prediction']: predicted cell type labels
- new_adata.obs['Probability']: prediction probabilities
- new_adata.var['pathway_index']: gene set for each column
- ./my_project/gene2token_weights.csv: gene-to-token weights matrix

## How to apply

Train with TOSICA.train(ref_adata, gmt_path, project=<my_project>, label_name=<label_key>) on the reference AnnData using a pre-prepared gene-set mask (e.g. human_reactome / reactome.gmt, human_gobp, human_immune, human_tf, or mouse equivalents); training writes mask.npy, pathway.csv, label_dictionary.csv and model-n.pth into the project folder. Before prediction, align genes exactly with query_adata = query_adata[:, ref_adata.var_names] since var_names must be consistent and in the same order. Run new_adata = TOSICA.pre(query_adata, project=<my_project>) to obtain the attention matrix (new_adata.X), predicted labels (new_adata.obs['Prediction']), probabilities (new_adata.obs['Probability']), and gene2token_weights.csv. For integration, normalize the attention matrix by library-size correction (default size factor 10,000) and run PCA, nearest-neighbor graph, and UMAP. The architecture embeds n genes into k pathway tokens via masked linear transformation, concatenates a learnable CLS token, applies multi-head self-attention (Q/K/V linear projections), and feeds the CLS output through a fully connected layer with softmax for cell type probabilities; training uses SGD with cosine learning rate decay and cross-entropy loss, splitting studies/biological states into train/test with 30% of training as validation.

## Related tools

- **TOSICA** (Primary model: pathway-masked Transformer for cell type annotation, 'Unknown' abstention, and attention-based integration) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Loads reference/query AnnData, gene reordering, and downstream PCA/UMAP on the normalized attention matrix) — https://github.com/JackieHanLab/TOSICA
- **torch** (Deep learning backend (version 1.7.1) for the multi-head self-attention training and prediction) — https://github.com/JackieHanLab/TOSICA

## Examples

```
TOSICA.train(ref_adata, 'human_reactome', project='my_project', label_name='celltype'); query_adata = query_adata[:, ref_adata.var_names]; new_adata = TOSICA.pre(query_adata, project='my_project')
```

## Evaluation signals

- Project folder after TOSICA.train contains exactly mask.npy, pathway.csv, label_dictionary.csv, and model-n.pth
- ref_adata.var_names and query_adata.var_names are identical and in the same order before TOSICA.pre
- Held-out cells of a cell type deleted from training are predominantly labeled 'Unknown' (reported ~76% of held-out alpha cells in hPancreas), unlike SingleR/SciBet/ACTINN which misassign
- Prediction probabilities in new_adata.obs['Probability'] are valid softmax outputs and predictions agree with known labels on a held-out test split
- For integration benchmarks, the normalized attention matrix yields a reproducible scIB overall score (0.6 × biology conservation + 0.4 × batch removal) comparable across methods, e.g. an 11-row scib_ranking.csv

## Limitations

- The number of attention heads (H), learning rate value, batch size, and random seed are not specified in the methods, complicating exact reproduction
- Abstention on unseen cell types is probabilistic — competitors force wrong labels, but the reported abstention rate depends on the reference composition (e.g. alpha cells removed from hPancreas training)
- Genes of ref_adata and query_adata must be identical and ordered; mismatches cause silent errors unless query_adata[:, ref_adata.var_names] is applied

## Evidence

- [abstract] TOSICA, a multi-head self-attention deep learning model based on Transformer that enables interpretable cell type annotation: "Here, we present TOSICA, a multi-head self-attention deep learning model based on Transformer that enables interpretable cell type annotation"
- [readme] Gene ordering requirement before prediction: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order"
- [discussion] Mask construction rationale: "we map the input genes to selected gene sets (gmt files)"
- [discussion] Attention normalization for integration: "the matrix is normalized by library-size correction using default size factor 10,000"
- [results] Abstention finding on deleted alpha cells: [evidence span withheld — anchor status: not_found]
- [discussion] Training recipe: "Stochastic gradient descent (SGD) is chosen as optimizer, and we use cosine learning rate decay to avoid too large steps in late stage of training"
