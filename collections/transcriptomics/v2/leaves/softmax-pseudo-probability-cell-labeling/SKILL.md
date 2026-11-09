---
name: softmax-pseudo-probability-cell-labeling
description: Use when cells must be labeled into known types from curated marker gene
  lists (e.g. CD3D/TRAC for T cells, CD79A/MZB1 for B/Plasma, LYZ/CST3 for Myeloid)
  after training GeneVector gene embeddings on an AnnData object.
when_to_use_negative:
- Input is not raw single-cell expression in AnnData .X (e.g. already a feature table,
  bulk RNA-seq, or pre-normalized embeddings) — GeneVectorDataset requires raw counts.
- No validated marker gene sets exist for the target cell types — the method depends
  entirely on curated marker lists, not de novo clustering.
- Cell types differ at finer granularity than the marker sets distinguish (e.g. CD4
  vs CD8 T cell subtypes in the TICA three-type scheme) — use metagene clustering
  or reference-mapping tools instead.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GeneVector
  role: Provides CellEmbedding.phenotype_probability, which computes SoftMax pseudo-probabilities
    and argmax labels from marker gene sets over trained gene embeddings
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Loads the AnnData object with raw counts in .X and visualizes pseudo-probability
    columns on the UMAP
- name: AnnData
  role: Container format holding raw counts in .X and receiving pseudo-probability
    and label columns in .obs
- name: PyTorch
  role: Backend for GeneVector's neural embedding training that produces the gene
    vectors used for cell labeling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:e414c3447d2b30de52f5937bccd11829c86f5f938099753103957aef98f77f04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nceglia/genevector
  tools:
  - AnnData
  - GeneVector
  - NumPy
  - PyTorch
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# softmax-pseudo-probability-cell-labeling

## Summary

Assigns cell type labels by computing SoftMax pseudo-probabilities from curated marker gene sets over GeneVector cell embeddings, then taking the argmax per cell. Used in the TICA workflow to classify T, B/Plasma, and Myeloid cells against CellTypist-derived reference annotations.

## When to use

Use when cells must be labeled into known types from curated marker gene lists (e.g. CD3D/TRAC for T cells, CD79A/MZB1 for B/Plasma, LYZ/CST3 for Myeloid) after training GeneVector gene embeddings on an AnnData object.

## When NOT to use

- Input is not raw single-cell expression in AnnData .X (e.g. already a feature table, bulk RNA-seq, or pre-normalized embeddings) — GeneVectorDataset requires raw counts.
- No validated marker gene sets exist for the target cell types — the method depends entirely on curated marker lists, not de novo clustering.
- Cell types differ at finer granularity than the marker sets distinguish (e.g. CD4 vs CD8 T cell subtypes in the TICA three-type scheme) — use metagene clustering or reference-mapping tools instead.

## Inputs

- Scanpy AnnData object with raw counts in .X (e.g. TICA subset, 23,764 cells, 2,000 highly variable genes)
- GeneEmbedding loaded from genes.vec with vector='average'
- CellEmbedding constructed from the GeneVectorDataset and GeneEmbedding
- Dictionary of cell type to marker gene lists (e.g. {'T Cell': ['CD3D','CD3G','CD3E','TRAC','IL32','CD2'], ...})

## Outputs

- Annotated AnnData with SoftMax pseudo-probability columns ('Pseudo-probability' prefixed) in .obs
- Argmax cell type label per cell in the 'genevector' column of .obs
- Confusion matrix / per-type accuracies when cross-tabulated against reference annotations

## How to apply

Train a GeneVector model (emb_dimension=100, 1000 iterations, threshold 1e-6) on a GeneVectorDataset built from raw counts in AnnData .X (e.g. TICA subset, 23,764 cells, 2,000 highly variable genes, signed_mi=True, 10 quantile bins). Construct a GeneEmbedding with vector='average' from genes.vec, aggregate into a CellEmbedding, then call cell_embed.phenotype_probability(adata, markers) with a dictionary mapping each cell type to its marker genes (T Cell: CD3D/CD3G/CD3E/TRAC/IL32/CD2; B/Plasma: CD79A/CD79B/MZB1/CD19/BANK1; Myeloid: LYZ/CST3/AIF1/CD68/C1QA/C1QB/C1QC). The method computes SoftMax pseudo-probabilities per cell and assigns the argmax label (stored in the 'genevector' obs column and 'Pseudo-probability' columns). Rationale: averaging input and output gene weights and projecting marker vectors onto cell embeddings leverages latent-space arithmetic rather than raw expression, which is robust to scRNA-seq sparsity. Validate by cross-tabulating predicted labels against reference annotations (e.g. CellTypist-derived) and computing per-type accuracies.

## Related tools

- **GeneVector** (Provides CellEmbedding.phenotype_probability, which computes SoftMax pseudo-probabilities and argmax labels from marker gene sets over trained gene embeddings) — https://github.com/nceglia/genevector
- **Scanpy** (Loads the AnnData object with raw counts in .X and visualizes pseudo-probability columns on the UMAP)
- **AnnData** (Container format holding raw counts in .X and receiving pseudo-probability and label columns in .obs)
- **PyTorch** (Backend for GeneVector's neural embedding training that produces the gene vectors used for cell labeling)

## Examples

```
annotated_adata = cell_embed.phenotype_probability(adata_embedded, markers)
```

## Evaluation signals

- On the TICA subset, per-type accuracies should reproduce ~97.1% (T cells), 95% (myeloid), and 94.3% (B/Plasma) versus CellTypist-derived reference annotations, with ~3.977% overall misclassification.
- The annotated AnnData must contain one 'Pseudo-probability' column per cell type in .obs plus a 'genevector' argmax label column.
- Pseudo-probabilities should be SoftMax-normalized (sum to 1 across cell types per cell) and the argmax label must correspond to the maximum pseudo-probability.
- For comparison, CellAssign baseline on the same data gave 85.6% myeloid accuracy with 7.54% misclassification — GeneVector labeling should outperform this.
- Marker genes used must be expressed/detected in the dataset; missing markers will degrade the corresponding pseudo-probability signal.

## Limitations

- Accuracy depends on the quality and completeness of the curated marker gene sets; the method assigns only types covered by the supplied markers.
- GeneVector builds mutual information that is agnostic to direction of statistical dependency and cannot discern repression from activation.
- MI estimation quality is limited by cell number, count fidelity, and discretization strategy (10 quantile bins, zero expression mapped to bin 0).
- Batch correction derived from cell embeddings is unreliable when batches differ in cell type composition and must be manually verified against cell type signatures.

## Evidence

- [readme] Automated cell type assignment using marker gene sets: [evidence span withheld — anchor status: not_found]
- [readme] phenotype_probability invocation: [evidence span withheld — anchor status: external_doc]
- [methods] SoftMax pseudo-probabilities and argmax labels: "adata = cell_embed.phenotype_probability(adata, markers)"
- [abstract] Latent space arithmetic for cell classification: "GeneVector uses latent space arithmetic in a lower dimensional gene embedding to identify transcriptional programs and classify cell types."
- [intro] Validate cell type predictions against published annotations: "We demonstrate that latent space arithmetic can be used to accurately label cell types in the TICA dataset and validate our cell type predictions against published annotations"
- [readme] Average of input and output weights recommended: [evidence span withheld — anchor status: external_doc]
