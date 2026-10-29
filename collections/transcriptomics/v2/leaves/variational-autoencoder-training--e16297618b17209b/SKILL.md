---
name: variational-autoencoder-training
description: Use when the goal is to infer protein functional association networks
  from sparse, redundant high-dimensional omics data, or to verify that a FAVA VAE
  model does not overfit by benchmarking train vs. test splits against KEGG pathway
  co-membership.
when_to_use_negative:
- Simple co-expression analysis is sufficient — the article notes inferring networks
  directly from omics data via co-expression does not work well due to sparseness
  and redundancy, which is precisely why the VAE is needed
- Input data is already a low-dimensional embedding or an existing functional association
  network — the VAE targets high-dimensional, sparse omics representations
- The goal requires literature-curated network coverage for well-studied proteins
  rather than bias-free networks with coverage of understudied proteins
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FAVA (favapy)
  role: Provides the VAE, load_data ('tsv' input type), and pairs_after_cutoff functions
    used to train the autoencoder and rank protein pairs
  repo: https://github.com/mikelkou/fava
- name: AnnData
  role: Input data structure for favapy within the scverse ecosystem
  repo: https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy
- name: Python
  role: Language in which the fava.py module (VAE, cook, pairs_after_cutoff, load_data,
    create_protein_pairs) is implemented and invoked
  repo: https://github.com/mikelkou/fava
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:61a991b78bd57d4d69132ae46c8ab2a4325a6a60316ef9aa6004fa0ffad99fd7
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# variational-autoencoder-training

## Summary

Trains the FAVA variational autoencoder (favapy) to compress high-dimensional scRNA-seq (e.g. GSE75748) data into a dense low-dimensional latent space from which functional protein association networks are inferred, and checks generalization by comparing held-out test performance against training performance (Supplementary Fig. S5).

## When to use

Use when the goal is to infer protein functional association networks from sparse, redundant high-dimensional omics data, or to verify that a FAVA VAE model does not overfit by benchmarking train vs. test splits against KEGG pathway co-membership.

## When NOT to use

- Simple co-expression analysis is sufficient — the article notes inferring networks directly from omics data via co-expression does not work well due to sparseness and redundancy, which is precisely why the VAE is needed
- Input data is already a low-dimensional embedding or an existing functional association network — the VAE targets high-dimensional, sparse omics representations
- The goal requires literature-curated network coverage for well-studied proteins rather than bias-free networks with coverage of understudied proteins

## Inputs

- scRNA-seq gene-expression matrix in tsv format (e.g. GEO dataset GSE75748), loadable by favapy load_data
- AnnData object (favapy input source within the scverse ecosystem)

## Outputs

- Trained FAVA VAE model embedding cells in a low-dimensional latent space
- Latent-space Pearson correlations between proteins
- Ranked protein pair list from pairs_after_cutoff (default 100000 interactions, both directions)
- Train vs. test benchmark scores against KEGG pathway co-membership gold standard (Supplementary Fig. S5 analogue)

## How to apply

Load the scRNA-seq dataset (e.g. GSE75748) via favapy's load_data in AnnData format using the 'tsv' input type. Randomly partition cells into a 67% training and 33% test split, then train the VAE class on the training split only with documented defaults (50 epochs, batch size 32, hidden and latent dimensions sized automatically from the input). Run the trained model on both splits to compute Pearson correlations in the latent space, then rank protein pairs with pairs_after_cutoff (default 100000 interactions, keeping both directions proteinA-proteinB and proteinB-proteinA). Evaluate both ranked lists against a KEGG pathway co-membership gold standard; rationale is that compressing the sparse original data into a dense latent space yields much improved networks, and similar training vs. test scores (Supplementary Fig. S5) demonstrate the model does not overfit.

## Related tools

- **FAVA (favapy)** (Provides the VAE, load_data ('tsv' input type), and pairs_after_cutoff functions used to train the autoencoder and rank protein pairs) — https://github.com/mikelkou/fava
- **AnnData** (Input data structure for favapy within the scverse ecosystem) — https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy
- **Python** (Language in which the fava.py module (VAE, cook, pairs_after_cutoff, load_data, create_protein_pairs) is implemented and invoked) — https://github.com/mikelkou/fava

## Examples

```
pip install favapy; favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- Similar benchmark performance on the 67% training split and the 33% test split against the KEGG gold standard, indicating no overfitting (Supplementary Fig. S5)
- pairs_after_cutoff returns the default 100000 interactions covering both directions (proteinA-proteinB and proteinB-proteinA)
- Latent-space Pearson correlations yield improved networks relative to correlations computed in the original high-dimensional representation
- Input loads as a valid AnnData object via favapy load_data with the 'tsv' input type
- Training completes with default settings (50 epochs, batch size 32, hidden/latent dimensions auto-sized from input)

## Limitations

- Overfitting is only ruled out by train/test comparison on a single dataset (GSE75748); a 67/33 random split should be re-verified for other datasets
- Networks inferred from scRNA-seq/proteomics input are literature-bias-free but only as good as the latent-space correlation ranking; coverage of understudied proteins (e.g. 4210 interactions among 1039 understudied proteins) reflects prediction, not experimental validation
- Simple co-expression on the original representation fails due to sparseness and redundancy, so results cannot be cross-checked against that naive baseline for equivalent quality

## Evidence

- [abstract] FAVA VAE compresses data: "we have developed FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space"
- [intro] latent correlations improve networks: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [methods] favapy installation: "You can install FAVA using pip:

.. code-block:: bash

   pip install favapy"
- [supplementary] train/test generalization: "Similar performance was obtained for both training and test sets (Supplementary Fig. S5), suggesting that our models do not overfit the data"
