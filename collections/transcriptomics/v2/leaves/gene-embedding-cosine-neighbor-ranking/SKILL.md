---
name: gene-embedding-cosine-neighbor-ranking
description: Use when a trained GeneVector embedding (genes.vec) is available and
  you must rank a query gene's nearest neighbors by cosine similarity to confirm pathway
  co-membership or seed a metagene.
when_to_use_negative:
- 'You need the direction of regulation: GeneVector''s MI is agnostic to activation
  vs repression, so cosine neighbors cannot separate co-activation from co-repression.'
- Query or pathway genes were removed by entropy-based QC (entropy_threshold=1.0)
  or HVG subsetting and are absent from genes.vec — they cannot be ranked.
- 'The embedding was trained with target=''pearson'' and pathway fidelity matters:
  expected IFIT1 top-16 overlap with Reactome R-HSA-913531 falls to 6/16 vs 12/16
  with the MI target.'
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GeneVector
  role: GeneEmbedding.compute_similarities and plot_similarities rank all embedded
    genes by cosine similarity to a query gene using the averaged input/output vectors
  repo: https://github.com/nceglia/genevector
- name: PyTorch
  role: trains the neural gene embedding (emb_dimension=100, 1000 iterations, threshold=1e-6)
    whose vector geometry is queried
- name: Scanpy
  role: supplies the AnnData raw-count matrix (.X) from which the GeneVectorDataset
    and gene embedding are built
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:b69a16f09755931e6685c5b5ab58fb91e3191cfd08b7cc4f5311d30a3732367b
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
  - SciPy
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# gene-embedding-cosine-neighbor-ranking

## Summary

Rank a query gene's nearest neighbors in a trained GeneVector gene embedding by cosine similarity (compute_similarities on input+output-averaged vectors) to verify that the embedding geometry recovers pathway co-members, e.g., interferon signaling genes around IFIT1.

## When to use

Use when a trained GeneVector embedding (genes.vec) is available and you must rank a query gene's nearest neighbors by cosine similarity to confirm pathway co-membership or seed a metagene.

## When NOT to use

- You need the direction of regulation: GeneVector's MI is agnostic to activation vs repression, so cosine neighbors cannot separate co-activation from co-repression.
- Query or pathway genes were removed by entropy-based QC (entropy_threshold=1.0) or HVG subsetting and are absent from genes.vec — they cannot be ranked.
- The embedding was trained with target='pearson' and pathway fidelity matters: expected IFIT1 top-16 overlap with Reactome R-HSA-913531 falls to 6/16 vs 12/16 with the MI target.

## Inputs

- Scanpy AnnData with raw counts in .X (e.g., interferon beta-stimulated PBMCs) used to build the GeneVectorDataset
- Trained GeneVector gene embedding file genes.vec (input + output weight vectors)
- Query gene symbol present in the embedding (e.g., IFIT1)
- Optional reference gene sets for validation: Reactome Interferon Signaling R-HSA-913531, MSigDB C8 pathways

## Outputs

- Ranked cosine-similarity dataframe for the query gene covering every embedded gene
- Top-k nearest-neighbor gene list (e.g., top 16 IFIT1 neighbors)
- Optional neighbor similarity plot (plot_similarities, n_genes=10)

## How to apply

Train the embedding with GeneVector(dataset, output_file='genes.vec', emb_dimension=100) and model.train(1000, threshold=1e-6) on the default signed-MI target, then load it with GeneEmbedding('genes.vec', dataset, vector='average') — the README recommends averaging the input and output weight files for best results. Call compute_similarities(query_gene) to rank every embedded gene by cosine similarity to the query (e.g., IFIT1) and optionally visualize with plot_similarities(query_gene, n_genes=10). Validate that the geometry reflects biology by regressing binary Reactome + MSigDB C8 pathway co-membership on cosine similarity (expected r² ≈ 0.233 for the MI target vs ≈ 0.093 for a Pearson-target embedding) and by checking that the top-16 IFIT1 neighbors overlap Reactome Interferon Signaling R-HSA-913531 at ~12/16 (MI) vs 6/16 (Pearson). The ranked neighbors then seed a Leiden-clustered metagene (get_adata(resolution=40), get_metagenes), such as the ISG metagene containing IFIT1/IFIT2/IFIT3/ISG15/ISG20/RSAD2/OAS1/MX1. If neighbors include unrelated markers, revisit gene QC (entropy_threshold=1.0) and the training target before trusting the ranking.

## Related tools

- **GeneVector** (GeneEmbedding.compute_similarities and plot_similarities rank all embedded genes by cosine similarity to a query gene using the averaged input/output vectors) — https://github.com/nceglia/genevector
- **PyTorch** (trains the neural gene embedding (emb_dimension=100, 1000 iterations, threshold=1e-6) whose vector geometry is queried)
- **Scanpy** (supplies the AnnData raw-count matrix (.X) from which the GeneVectorDataset and gene embedding are built)

## Examples

```
gene_embed = GeneEmbedding('genes.vec', dataset, vector='average'); sims = gene_embed.compute_similarities('IFIT1')
```

## Evaluation signals

- Top-16 IFIT1 cosine neighbors overlap Reactome Interferon Signaling R-HSA-913531 at ~12/16 for the MI-trained embedding (vs 6/16 for a Pearson-trained one).
- Regression of binary Reactome + MSigDB C8 pathway co-membership on cosine similarity yields r² ≈ 0.233 (MI) vs ≈ 0.093 (Pearson).
- Ranked neighbors are interferon pathway genes (IFIT2, IFIT3, ISG15, ISG20, RSAD2, OAS1, MX1), not unrelated cell-type markers such as PRF1 or LAG3.
- Embedding was loaded with vector='average' (mean of input and output weights), as the README recommends.
- Schema check: compute_similarities returns a gene–similarity dataframe covering every gene in the embedding.

## Limitations

- GeneVector is unable to discern repression from activation, as it builds MI that is agnostic to the direction of the statistical dependency.
- Empirical MI from binned expression counts is limited by the available number of cells, count fidelity, and the discretization strategy (default 10 quantile bins, zero expression maps to bin 0).
- The MI calculation is computationally expensive; for 2,000 genes × 5,000 cells it takes ~3 hours on NumPy and ~26 hours legacy, so ranking should reuse cached scores or numba/rust/GPU backends.
- Neighbor lists depend on which genes survived QC and on the embedding's resolution settings; genes dropped during preprocessing cannot be retrieved.

## Evidence

- [readme] compute_similarities ranks genes by cosine similarity to a query gene, with a companion plot of top neighbors: "similarities = gene_embed.compute_similarities("CD8A")
gene_embed.plot_similarities("CD8A", n_genes=10)"
- [readme] Averaging input and output weight vectors is the recommended way to load the embedding: [evidence span withheld — anchor status: external_doc]
- [methods] Embedding training parameters used in the paper: emb_dimension=100, 1000 iterations, threshold=1e-6: "model = GeneVector(dataset, output_file="genes.vec", emb_dimension=100)
   model.train(1000, threshold=1e-6)"
- [discussion] Gene vectors can be turned into a co-expression graph and clustered into metagenes, the downstream use of ranked neighbors: "The resulting vectors can be used to generate a gene co-expression graph, and can be clustered to predict transcriptional programs, or metagenes, in an unsupervised fashion"
- [abstract] Embedding geometry is constructed from mutual information between gene expression: "We describe GeneVector, a scalable framework for dimensionality reduction implemented as a vector space model using mutual information between gene expression."
