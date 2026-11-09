---
name: zinq-reconstruction-loss-optimization
description: Use when integrating scRNA-seq batches where some (10/20/50%) or all
  cell type labels are available and the latent embedding must be guided by a cell
  type classifier loss on top of an adversarial VAE ELBO.
when_to_use_negative:
- No cell type annotations are available for any cells — use the unsupervised scDREAMER
  backbone instead of scDREAMER-Sup
- The number of batches is unknown and cannot be supplied — the model assumes prior
  knowledge of the number of batches
- Multiomic (e.g. RNA + ATAC) datasets — the authors state the multiomic extension
  is not yet implemented and is planned future work
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scDREAMER-Sup
  role: Supervised extension adding VAE AY and cell type classifier C on top of the
    scDREAMER adversarial VAE; trained with the extended ELBO plus cross-entropy loss
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: scDREAMER
  role: Backbone adversarial variational autoencoder (encoder E, ZINB decoder D, discriminator
    with Bhattacharyya loss, batch classifier B) whose latent embedding zi feeds classifier
    C
  repo: https://github.com/Zafar-Lab/scDREAMER
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:057f5d2320f3d38a02fdf6a90d0a04985deacfe614de0faefb5d6d3bede0f994
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - scDREAMER
  - scDREAMER-Sup
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# zinq-reconstruction-loss-optimization

## Summary

Reconstructs the scDREAMER-Sup objective: an adversarial VAE backbone whose ZINB decoder reconstruction and KL terms are maximized via an extended ELBO, augmented with a second VAE (AY = {Ey, Dy}) and a feed-forward cell type classifier C trained by cross-entropy on available annotations. This couples latent-space batch correction with annotation-guided (supervised/semi-supervised) inference.

## When to use

Use when integrating scRNA-seq batches where some (10/20/50%) or all cell type labels are available and the latent embedding must be guided by a cell type classifier loss on top of an adversarial VAE ELBO.

## When NOT to use

- No cell type annotations are available for any cells — use the unsupervised scDREAMER backbone instead of scDREAMER-Sup
- The number of batches is unknown and cannot be supplied — the model assumes prior knowledge of the number of batches
- Multiomic (e.g. RNA + ATAC) datasets — the authors state the multiomic extension is not yet implemented and is planned future work

## Inputs

- Scanpy AnnData object with raw count expression matrix (e.g. human pancreas dataset, 16,382 cells)
- Batch label vector per cell
- Cell type annotation vector with labels masked on a held-out fraction of cells (10%, 20%, or 50%) for semi-supervised inference
- Top 2000 highly variable genes selected with scanly.pp.highly_variable_genes (flavor 'seurat')

## Outputs

- Informative latent cellular embeddings z for all cells
- Predicted cell type labels for unlabeled cells from classifier C
- Single h5ad file containing embeddings and predicted labels

## How to apply

Start from Scanpy AnnData of raw counts, normalized with scanly.pp.normalize_total, log-transformed, and reduced to the top 2000 highly variable genes (scanly.pp.highly_variable_genes, flavor 'seurat'). Instantiate the scDREAMER backbone (encoder E mapping {x, s} to {μz, σz, μl, σl}, ZINB decoder D, discriminator trained with Bhattacharyya loss, adversarially trained batch classifier B) plus the additional VAE AY = {Ey, Dy} and the feed-forward classifier C, all as multi-layer dense networks with linear relu activation. Train with the ADAM optimizer at batch size 128, maximizing the extended ELBO — KL terms for z, l, and the y-prior conditioned on cell type label c — plus the cross-entropy loss for C computed only on cells with available annotations (mask labels on held-out cells for the semi-supervised setting). Use the trained C to predict cell type labels for unlabeled cells. Save the informative latent embeddings z for all cells and the predicted labels to a single h5ad file.

## Related tools

- **scDREAMER-Sup** (Supervised extension adding VAE AY and cell type classifier C on top of the scDREAMER adversarial VAE; trained with the extended ELBO plus cross-entropy loss) — https://github.com/Zafar-Lab/scDREAMER
- **scDREAMER** (Backbone adversarial variational autoencoder (encoder E, ZINB decoder D, discriminator with Bhattacharyya loss, batch classifier B) whose latent embedding zi feeds classifier C) — https://github.com/Zafar-Lab/scDREAMER

## Examples

```
git clone https://github.com/Zafar-Lab/scDREAMER.git; cd scDREAMER/Environments; conda env create -f scDREAMER.yml; conda activate scdreamer
```

## Evaluation signals

- Trained model must produce latent embeddings that support both batch mixing and biological conservation of the annotated cell types
- Predicted labels for masked cells scored with macro F1-score against held-out annotations
- Louvain clustering on the embedding at the resolution that maximizes NMI should recover biologically conserved clusters
- Held-out cell type experiments (e.g. alpha and delta cells held out during training) should still separate the novel cell type in the learned embedding
- Ablation of the adversarial discriminator or batch classifier should degrade batch mixing, confirming both adversarial components contribute to the loss landscape being optimized

## Limitations

- The model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored
- Hierarchical structure between different batch information is not yet utilized
- As a deep learning-based method, scDREAMER may require fine-tuning of some parameters for extracting the best performance
- Multiomic dataset extension is not yet implemented (planned future work)

## Evidence

- [intro] The supervised version, scDREAMER-Sup, employs an additional variational autoencoder and a cell type classifier (another feed-forward neural network) to utilize available cell type annotations: "The supervised version, scDREAMER-Sup, employs an additional variational autoencoder and a cell type classifier (another feed-forward neural network) to utilize available cell type annotations"
- [methods] The batch size of 128 is used while training the model.: "The batch size of 128 is used while training the model."
- [methods] we selected the top 2000 highly variable genes using the function “scanly.pp.highly_variable_genes” (with flavor parameter as “seurat”) as input to scDREAMER: [evidence span withheld — anchor status: not_found]
- [methods] We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value.: "We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value."
- [readme] scDREAMER-SUP can also be when cell type annotations are missing in the datasets i.e., 10%, 20%, 50%: "scDREAMER-SUP can also be when cell type annotations are missing in the datasets i.e., 10%, 20%, 50%"
- [discussion] Our current model assumes prior knowledge of the number of batches.: "Our current model assumes prior knowledge of the number of batches."
