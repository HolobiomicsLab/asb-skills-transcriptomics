---
name: interferon-signaling-pathway-geneset-annotation
description: Use when evaluating or annotating a gene embedding learned from interferon-stimulated
  PBMC scRNA-seq data against the Reactome Interferon Signaling pathway R-HSA-913531
  or MSigDB C8 co-membership, e.g. comparing MI vs Pearson training objectives.
when_to_use_negative:
- The goal is to determine direction of regulation — GeneVector builds MI that is
  agnostic to whether a dependency is activation or repression
- The input is not interferon-stimulated PBMC scRNA-seq (e.g. TICA, HGSOC, or TNBC
  PDX datasets) and the pathway of interest is not interferon signaling / ISG expression
- Genes have not passed quality control (e.g. entropy_threshold=1.0 filtering); sparse,
  low-entropy genes will degrade MI estimation
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GeneVector
  role: Trains MI- and Pearson-target gene embeddings, computes cosine similarities
    via compute_similarities, and supplies GeneEmbedding(vector='average') for neighbor
    extraction of IFIT1
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Loads and normalizes/log-transforms the PBMC interferon-beta AnnData used
    as GeneVectorDataset input
- name: AnnData
  role: Container format for the PBMC count matrix with raw counts in .X
- name: SciPy
  role: Regression of binary pathway co-membership on cosine similarity to obtain
    r² values
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:79fde156eb32671c285e1afbad146de4c2677e1e9aa0dadec02f3ab85ec51504
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nceglia/genevector
  tools:
  - AnnData
  - GeneVector
  - NumPy
  - Python
  - Scanpy
  - SciPy
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# interferon-signaling-pathway-geneset-annotation

## Summary

Annotates GeneVector gene embeddings against the Reactome Interferon Signaling pathway (R-HSA-913531) plus MSigDB C8 co-membership by comparing embedding cosine similarities to binary pathway membership, quantifying how well MI-trained vectors capture interferon-stimulated gene (ISG) relationships in the PBMC interferon-beta dataset.

## When to use

Use when evaluating or annotating a gene embedding learned from interferon-stimulated PBMC scRNA-seq data against the Reactome Interferon Signaling pathway R-HSA-913531 or MSigDB C8 co-membership, e.g. comparing MI vs Pearson training objectives.

## When NOT to use

- The goal is to determine direction of regulation — GeneVector builds MI that is agnostic to whether a dependency is activation or repression
- The input is not interferon-stimulated PBMC scRNA-seq (e.g. TICA, HGSOC, or TNBC PDX datasets) and the pathway of interest is not interferon signaling / ISG expression
- Genes have not passed quality control (e.g. entropy_threshold=1.0 filtering); sparse, low-entropy genes will degrade MI estimation

## Inputs

- Raw count AnnData (.h5ad) of PBMCs subjected to interferon beta stimulation (Scanpy object with raw counts in .X)
- MI-trained GeneVector gene embedding (genes.vec, from task_001)
- Pearson-target GeneVector gene embedding (trained with target='pearson')
- Reactome Interferon Signaling gene set R-HSA-913531
- MSigDB C8 pathway gene sets

## Outputs

- pathway_similarity_comparison.csv containing r² values for MI vs Pearson (expected 0.233 vs 0.093)
- IFIT1 top-16 nearest-neighbor lists per embedding with R-HSA-913531 overlap counts (expected 12/16 MI vs 6/16 Pearson)
- All-pairs gene cosine similarity matrices per embedding

## How to apply

Normalize and log-transform PBMC interferon-beta stimulation counts, then build a GeneVectorDataset and train the gene embedding (emb_dimension=100, 1000 iterations, threshold=1e-6). Load embeddings via GeneEmbedding(vector='average') and compute all-pairs gene cosine similarities with compute_similarities. Regress binary Reactome + MSigDB C8 pathway co-membership on cosine similarity and record r² (expected 0.233 for MI vs 0.093 for Pearson). As a targeted validation, extract the top-16 nearest neighbors of IFIT1 in each embedding and count overlap with Reactome Interferon Signaling R-HSA-913531 (expected 12/16 for MI vs 6/16 for Pearson), then write r² values and neighbor overlap lists to pathway_similarity_comparison.csv.

## Related tools

- **GeneVector** (Trains MI- and Pearson-target gene embeddings, computes cosine similarities via compute_similarities, and supplies GeneEmbedding(vector='average') for neighbor extraction of IFIT1) — https://github.com/nceglia/genevector
- **Scanpy** (Loads and normalizes/log-transforms the PBMC interferon-beta AnnData used as GeneVectorDataset input)
- **AnnData** (Container format for the PBMC count matrix with raw counts in .X)
- **SciPy** (Regression of binary pathway co-membership on cosine similarity to obtain r² values)

## Examples

```
from genevector.embedding import GeneEmbedding; gene_embed = GeneEmbedding("genes.vec", dataset, vector="average"); sims = gene_embed.compute_similarities("IFIT1")
```

## Evaluation signals

- Co-membership regression r² should be ≈0.233 for the MI-trained embedding and ≈0.093 for the Pearson-trained embedding
- Exactly 12 of IFIT1's top-16 nearest neighbors in the MI embedding belong to Reactome R-HSA-913531, vs 6/16 for the Pearson embedding
- Both r² values and overlap lists are present in pathway_similarity_comparison.csv
- Training converged within 1000 iterations at threshold=1e-6 with emb_dimension=100 for both objectives
- Embeddings loaded with vector='average' (input + output weight averaging), per README recommendation

## Limitations

- GeneVector cannot discern repression from activation — MI is agnostic to the direction of the statistical dependency
- Empirical MI estimation is limited by available number of cells, fidelity of counts, and discretization strategy; no improved estimator is provided
- MI computation is computationally expensive; without the numba/rust/gpu backends a 2,000-gene × 5,000-cell run can take hours (~26h legacy, ~3h NumPy vs minutes with acceleration)

## Evidence

- [intro] Objective: identify metagenes corresponding to cell-specific transcriptional processes in PBMCs, including interferon-stimulated gene (ISG) expression: "GeneVector can identify metagenes corresponding to cell-specific transcriptional processes in PBMCs, including interferon activated gene expression (ISG)"
- [methods] Training parameters for both MI and Pearson companion embeddings: "model = GeneVector(dataset, output_file="genes.vec", emb_dimension=100)
   model.train(1000, threshold=1e-6)"
- [readme] Embedding loading uses averaged input/output weights, the recommended best-result configuration: [evidence span withheld — anchor status: external_doc]
- [discussion] MI embeddings cannot encode direction of regulation, limiting pathway interpretation: "GeneVector is unable to discern repression from activation, as it builds MI that is agnostic to the direction of the statistical dependency"
- [intro] PBMC interferon-beta dataset grounds the ISG pathway annotation: "peripheral blood mononuclear cells (PBMCs) subjected to interferon beta stimulation19"
