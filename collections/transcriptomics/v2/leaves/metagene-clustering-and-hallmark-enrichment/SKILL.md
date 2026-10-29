---
name: metagene-clustering-and-hallmark-enrichment
description: Use when a trained GeneVector embedding must be turned into interpretable
  gene modules and ranked against a directional vector (e.g., Vadnexa − Vbowel in
  HGSOC cancer cells) to identify Hallmark-enriched programs.
when_to_use_negative:
- The goal is to infer directed regulatory relationships (activation vs. repression)
  — GeneVector's mutual-information target is agnostic to the direction of statistical
  dependency, so metagenes cannot distinguish these.
- Batches differ substantially in cell type composition, since batch correction vectors
  and downstream metagene interpretation become unreliable and must be manually verified
  against cell type signatures.
- Input is not a raw-count AnnData or no trained gene embedding exists — clustering
  metagenes presumes a completed GeneVector training run producing genes.vec.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GeneVector
  role: Provides GeneEmbedding.get_adata(resolution=40) for co-expression graph clustering,
    get_metagenes for module extraction, and compute_similarities for scoring genes
    against directional vectors
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Hosts the gene AnnData returned by get_adata and the cell AnnData to which
    metagene scores are written before export to .h5ad
  repo: https://github.com/nceglia/genevector
- name: PyTorch
  role: Neural-embedding backend that trains the gene vectors subsequently clustered
    into metagenes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:e8e79849e28ea3fc2ea1b7a7208735c242507439325ba37640dae632ec3b8dd9
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# metagene-clustering-and-hallmark-enrichment

## Summary

Decomposes a trained GeneVector gene embedding (genes.vec) into unsupervised metagenes by clustering the gene co-expression graph at a fixed Leiden resolution, then scores metagenes against a condition-direction vector to surface Hallmark-enriched transcriptional programs (e.g., EMT gain, MHC-I loss).

## When to use

Use when a trained GeneVector embedding must be turned into interpretable gene modules and ranked against a directional vector (e.g., Vadnexa − Vbowel in HGSOC cancer cells) to identify Hallmark-enriched programs.

## When NOT to use

- The goal is to infer directed regulatory relationships (activation vs. repression) — GeneVector's mutual-information target is agnostic to the direction of statistical dependency, so metagenes cannot distinguish these.
- Batches differ substantially in cell type composition, since batch correction vectors and downstream metagene interpretation become unreliable and must be manually verified against cell type signatures.
- Input is not a raw-count AnnData or no trained gene embedding exists — clustering metagenes presumes a completed GeneVector training run producing genes.vec.

## Inputs

- Trained GeneVector output file genes.vec (input and output gene weight vectors)
- GeneVectorDataset built from a raw-count Scanpy AnnData (.X) with entropy-based QC (entropy_threshold=1.0)
- Directional condition vector computed by latent-space arithmetic over cell embeddings (e.g., Vadnexa − Vbowel restricted to cancer cells)

## Outputs

- Leiden-clustered gene AnnData from get_adata(resolution=40) with metagene cluster assignments
- Dictionary of metagenes (gene modules) from get_metagenes
- Metagene scores against the directional vector with Hallmark pathway enrichment calls (e.g., EMT gain, MHC-I loss)
- Annotated AnnData with metagene scores saved as .h5ad (e.g., hgsoc_cell_annotations.h5ad)

## How to apply

Train the GeneVector model (emb_dimension=100, 1000 epochs, threshold=1e-6, signed mutual-information target cached in ~/.genevector/cache/) and load the two weight files into a GeneEmbedding with vector='average', since averaging input and output weights is recommended for best results. Cluster the embedding with gene_embed.get_adata(resolution=40) to build a gene co-expression graph and partition it into metagene clusters via gene_embed.get_metagenes. Restrict the arithmetic vector of interest (e.g., Vadnexa − Vbowel) to the relevant cells (EPCAM/CD24-positive cancer cells), then score each metagene against that directional vector to rank programs. Interpret top-scoring metagenes by checking membership of Hallmark pathway genes — in the HGSOC SPECTRUM analysis this recovered EMT genes gained in bowel metastasis and loss of MHC Class I genes (HLA-A, HLA-B, HLA-C, HLA-E, HLA-F) plus B2M. Persist metagene scores in the AnnData (e.g., hgsoc_cell_annotations.h5ad) alongside cell type probabilities.

## Related tools

- **GeneVector** (Provides GeneEmbedding.get_adata(resolution=40) for co-expression graph clustering, get_metagenes for module extraction, and compute_similarities for scoring genes against directional vectors) — https://github.com/nceglia/genevector
- **Scanpy** (Hosts the gene AnnData returned by get_adata and the cell AnnData to which metagene scores are written before export to .h5ad) — https://github.com/nceglia/genevector
- **PyTorch** (Neural-embedding backend that trains the gene vectors subsequently clustered into metagenes)

## Examples

```
gene_embed = GeneEmbedding('genes.vec', dataset, vector='average'); gene_adata = gene_embed.get_adata(resolution=40); metagenes = gene_embed.get_metagenes(gene_adata)
```

## Evaluation signals

- get_adata(resolution=40) yields a partitioned gene co-expression graph and get_metagenes returns non-empty gene modules covering the embedded genes
- Top-ranked metagenes against Vadnexa − Vbowel contain Hallmark EMT pathway genes (gained in bowel) and the expected loss of MHC Class I genes HLA-A, HLA-B, HLA-C, HLA-E, HLA-F plus B2M
- Metagene composition is reproducible when genes.vec is loaded with vector='average' (averaging input and output weights), matching the published HGSOC programs
- Metagene scores are present as columns in the output AnnData obs and the file exports cleanly as .h5ad
- Gene similarity spot-checks (e.g., compute_similarities on a marker like CD8A) return biologically coherent neighbors, indicating the embedding clusters are meaningful

## Limitations

- GeneVector's MI-based embedding cannot discern repression from activation, so metagenes reflect co-variation without regulatory direction.
- Empirical MI estimation is limited by the available number of cells, count fidelity, and the discretization strategy (zero → bin 0, nonzero → quantile bins, default 10); no improved estimator is provided, and MI computation is expensive without the fast backends.
- Metagene sets are data- and resolution-dependent; the resolution=40 setting and entropy_threshold=1.0 QC choices are dataset-specific defaults from the SPECTRUM HGSOC workflow.

## Evidence

- [discussion] unsupervised co-expression graph clustering: "The resulting vectors can be used to generate a gene co-expression graph, and can be clustered to predict transcriptional programs, or metagenes, in an unsupervised fashion"
- [methods] training parameters: "model = GeneVector(dataset, output_file="genes.vec", emb_dimension=100)
   model.train(1000, threshold=1e-6)"
- [readme] metagene clustering invocation: "gene_adata = gene_embed.get_adata(resolution=40)
metagenes = gene_embed.get_metagenes(gene_adata)"
- [readme] average weights recommendation: [evidence span withheld — anchor status: external_doc]
