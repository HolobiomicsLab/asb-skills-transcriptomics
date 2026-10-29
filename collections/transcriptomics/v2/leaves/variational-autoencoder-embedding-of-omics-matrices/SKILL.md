---
name: variational-autoencoder-embedding-of-omics-matrices
description: Use when the input is a large, sparse, redundant omics matrix (scRNA-seq
  tsv/csv or AnnData, >0.5 million conditions/cells, or bulk proteomics) and the goal
  is protein functional association network inference that outperforms Pearson correlation.
when_to_use_negative:
- The goal is standard differential expression or cell-type annotation on an AnnData
  object — FAVA produces protein interaction networks, not per-gene statistics.
- The dataset is too small or dense for the sparseness/redundancy problem to matter
  — simple co-expression (PCC) may suffice and is cheaper.
- The input is not gene/protein-by-condition omics data (e.g. already a feature table
  or an existing interaction network), since FAVA expects high-dimensional raw omics
  matrices.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0203
tools:
- name: FAVA (favapy)
  role: Variational autoencoder tool that compresses high-dimensional scRNA-seq/proteomics
    data into a low-dimensional latent space and infers functional association networks
  repo: https://github.com/mikelkou/fava
- name: AnnData
  role: Input data structure; FAVA functions within the scverse ecosystem and uses
    AnnData as its input source
  repo: https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy
- name: Python
  role: Language used to run the favapy CLI and the fava.py module (VAE, cook, pairs_after_cutoff)
  repo: https://github.com/mikelkou/fava
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:42cedbfe304d912d840df5e41d6e93014bf051394873a8dc9d311e2580c63439
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA (favapy)
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Variational autoencoder embedding of omics matrices

## Summary

Compress high-dimensional scRNA-seq or proteomics matrices (e.g. 566109 HPA cells, 633 PRIDE experiments) into a dense low-dimensional latent space with FAVA's variational autoencoder, so that correlations computed in the latent space yield far more accurate functional association networks than plain co-expression (PCC) while running ~9x faster.

## When to use

Use when the input is a large, sparse, redundant omics matrix (scRNA-seq tsv/csv or AnnData, >0.5 million conditions/cells, or bulk proteomics) and the goal is protein functional association network inference that outperforms Pearson correlation.

## When NOT to use

- The goal is standard differential expression or cell-type annotation on an AnnData object — FAVA produces protein interaction networks, not per-gene statistics.
- The dataset is too small or dense for the sparseness/redundancy problem to matter — simple co-expression (PCC) may suffice and is cheaper.
- The input is not gene/protein-by-condition omics data (e.g. already a feature table or an existing interaction network), since FAVA expects high-dimensional raw omics matrices.

## Inputs

- Human Protein Atlas scRNA-seq matrix (566109 cells, tsv/csv or AnnData)
- PRIDE bulk proteomics matrix (633 experiments, 18846 proteins)
- high-dimensional omics matrices (gene/protein x condition), scverse AnnData as input source

## Outputs

- top 100000 predicted protein interactions with both directions (proteinA-proteinB and proteinB-proteinA)
- low-dimensional latent embedding of the input omics matrix
- ranked functional association network for benchmarking against KEGG

## How to apply

Install the tool with 'pip install favapy' and load the omics matrix (FAVA functions within the scverse ecosystem, employing AnnData as its input source; tsv/csv is accepted). Run FAVA from the command line as 'favapy <path-to-data-file> <path-to-save-output>' with defaults: epochs=50, batch size=32, hidden and latent dimensions auto-scaled to input size. The VAE compresses the high-dimensional data into a dense, low-dimensional latent space, addressing the sparseness (many transcripts/proteins not consistently observed per cell/sample) and redundancy (many similar cells/samples) that break plain co-expression analysis. Correlations are then calculated in this latent space, and outputs are the top 100000 interactions with both directions (proteinA-proteinB and proteinB-proteinA); internal fava.py members include VAE, cook, pairs_after_cutoff, load_data, and create_protein_pairs. Validate by benchmarking the ranked pairs against KEGG: on HPA scRNA-seq, expect >5000 true positives with <500 false positives (>90% precision), and on PRIDE bulk proteomics FAVA should beat a PCC baseline swept across the full Pearson range (1 to -1) by a wide margin at ≥9x lower wall-clock runtime.

## Related tools

- **FAVA (favapy)** (Variational autoencoder tool that compresses high-dimensional scRNA-seq/proteomics data into a low-dimensional latent space and infers functional association networks) — https://github.com/mikelkou/fava
- **AnnData** (Input data structure; FAVA functions within the scverse ecosystem and uses AnnData as its input source) — https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy
- **Python** (Language used to run the favapy CLI and the fava.py module (VAE, cook, pairs_after_cutoff)) — https://github.com/mikelkou/fava

## Examples

```
pip install favapy; favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- On HPA scRNA-seq, the FAVA network benchmarked against KEGG yields >5000 true positives with <500 false positives (>90% precision), outperforming PCC by a wide margin
- FAVA wall-clock runtime is at least 9x faster than PCC (1 day vs ~9 days) on the large datasets
- Output contains the top 100000 interactions, each listed in both directions (proteinA-proteinB and proteinB-proteinA)
- Hidden and latent dimensions are auto-scaled to input size; run uses defaults epochs=50, batch size=32
- Input is accepted via AnnData / tsv or csv matrix without QC failures on >0.5M conditions

## Limitations

- Plain co-expression analysis on the original representation performs poorly due to high sparseness and redundancy of scRNA-seq/proteomics data, which is why the latent-space embedding is required
- Existing high-quality protein networks are heavily biased by data availability — well-studied proteins have far more interactions than understudied proteins; FAVA's value is better coverage of the 1039 understudied proteins (4210 predicted interactions), not per-interaction experimental validation
- Networks derived from omics inputs should be free of literature bias but are limited to protein pairs present in the input matrix

## Evidence

- [abstract] FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space: "we have developed FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space"
- [intro] Calculating correlations in this latent space results in much improved networks compared to the original representation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [methods] Run FAVA from the command line as favapy <path-to-data-file> <path-to-save-output>: "Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>"
- [abstract] FAVA functions within the scverse ecosystem, employing AnnData as its input source: "FAVA functions within the scverse ecosystem, employing AnnData as its input source"
- [abstract] FAVA can process large datasets with over 0.5 million conditions: "FAVA can process large datasets with over 0.5 million conditions"
