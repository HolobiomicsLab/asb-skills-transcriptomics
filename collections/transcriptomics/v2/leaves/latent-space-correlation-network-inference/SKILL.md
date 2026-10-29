---
name: latent-space-correlation-network-inference
description: Use when the input is a high-dimensional, sparse and redundant omics
  matrix (e.g. scRNA-seq cell-by-gene counts such as GSE75748, or Human Protein Atlas
  / PRIDE proteomics) and the goal is a ranked protein-protein association network
  rather than raw co-expression links.
when_to_use_negative:
- If the data are not sparse/redundant high-dimensional omics profiles — simple co-expression
  analysis on the original representation may suffice, and the VAE adds unnecessary
  complexity
- If the goal is literature-curated networks for well-studied proteins rather than
  bias-free coverage of understudied proteins — existing high-quality networks already
  have many more interactions for well-studied proteins
- If input is already a low-dimensional embedding or an existing feature table rather
  than raw cell/sample-by-protein measurements
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: FAVA (favapy)
  role: Trains the VAE, computes latent-space Pearson correlations, and ranks protein
    pairs via pairs_after_cutoff
  repo: https://github.com/mikelkou/fava
- name: AnnData
  role: Input data container; FAVA functions within the scverse ecosystem and expects
    AnnData objects
- name: Python
  role: Language for the fava.py module providing VAE, cook, pairs_after_cutoff, and
    load_data functions
  repo: https://github.com/mikelkou/fava
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:93d9a757fbb7c689b5e70babe59e8baec397233bb1b745ba30dbd594d84d0ac9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA (favapy)
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# latent-space-correlation-network-inference

## Summary

Infers protein functional association networks by training a variational autoencoder (FAVA) that compresses high-dimensional scRNA-seq or proteomics data into a dense low-dimensional latent space, then ranking protein pairs by correlations computed in that latent space.

## When to use

Use when the input is a high-dimensional, sparse and redundant omics matrix (e.g. scRNA-seq cell-by-gene counts such as GSE75748, or Human Protein Atlas / PRIDE proteomics) and the goal is a ranked protein-protein association network rather than raw co-expression links.

## When NOT to use

- If the data are not sparse/redundant high-dimensional omics profiles — simple co-expression analysis on the original representation may suffice, and the VAE adds unnecessary complexity
- If the goal is literature-curated networks for well-studied proteins rather than bias-free coverage of understudied proteins — existing high-quality networks already have many more interactions for well-studied proteins
- If input is already a low-dimensional embedding or an existing feature table rather than raw cell/sample-by-protein measurements

## Inputs

- scRNA-seq or proteomics data as a TSV file (e.g. GSE75748 from GEO) loadable via favapy load_data
- AnnData object (scverse ecosystem input source)
- Optional train/test cell split (67%/33%) for generalization checks

## Outputs

- Ranked protein-protein association list (~100000 interactions, both proteinA-proteinB and proteinB-proteinA directions) from pairs_after_cutoff
- Pearson correlation matrix of protein pairs computed in the latent space
- Benchmark scores against KEGG pathway co-membership gold standard (e.g. train vs test split comparison)

## How to apply

Load the data with favapy's load_data into an AnnData object (scverse ecosystem, 'tsv' input type), then run the FAVA VAE — default 50 epochs, batch size 32, hidden and latent dimensions sized automatically from the input — to compress the high-dimensional matrix into a dense latent space. Compute Pearson correlations between protein embeddings in the latent space; rationale is that correlations in this space outperform correlations on the original sparse/redundant representation. Rank protein pairs with pairs_after_cutoff, keeping the default 100000 interactions written in both directions (proteinA-proteinB and proteinB-proteinB reversed) to produce the final association list. For quality control, split cells into a 67% train / 33% test partition, train on the training split only, and evaluate both ranked lists against a KEGG pathway co-membership gold standard; similar performance on both splits (Supplementary Fig. S5) confirms the VAE does not overfit.

## Related tools

- **FAVA (favapy)** (Trains the VAE, computes latent-space Pearson correlations, and ranks protein pairs via pairs_after_cutoff) — https://github.com/mikelkou/fava
- **AnnData** (Input data container; FAVA functions within the scverse ecosystem and expects AnnData objects)
- **Python** (Language for the fava.py module providing VAE, cook, pairs_after_cutoff, and load_data functions) — https://github.com/mikelkou/fava

## Examples

```
favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- Train/test split (67%/33%) yields similar KEGG co-membership benchmark performance, reproducing Supplementary Fig. S5 (no overfitting)
- pairs_after_cutoff returns the default 100000 interactions written in both directions (proteinA-proteinB and proteinB-proteinA)
- Latent-space correlations yield networks with higher accuracy than correlations computed on the original high-dimensional representation
- Input data loads as a valid AnnData object via favapy load_data with 'tsv' input type
- Latent and hidden dimensions are sized automatically from the input after 50 epochs of training at batch size 32

## Limitations

- Simple co-expression on raw data fails due to high sparseness (transcripts/proteins not consistently observed per cell) and redundancy (many similar cells analyzed), which is why the latent compression is required — skipping the VAE step degrades network quality
- Existing high-quality protein networks are heavily biased by data availability, so comparisons against them may under-represent FAVA's coverage of understudied proteins
- Generalization must be verified empirically; the overfitting check relies on comparing training vs test performance on held-out cells

## Evidence

- [abstract] FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space: "FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space"
- [intro] Calculating correlations in this latent space results in much improved networks compared to the original representation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [methods] Run FAVA from the command line as follows: favapy <path-to-data-file> <path-to-save-output>: "Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>"
- [readme] You can install FAVA using pip: pip install favapy: "You can install FAVA using pip:

.. code-block:: bash

   pip install favapy"
