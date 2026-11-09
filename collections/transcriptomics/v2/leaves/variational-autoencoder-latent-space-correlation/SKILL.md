---
name: variational-autoencoder-latent-space-correlation
description: Use when you have a high-dimensional scRNA-seq or proteomics expression
  matrix (e.g. tsv/csv of gene x condition values, AnnData-compatible) and need a
  ranked protein-protein association network, especially when simple co-expression
  on the raw matrix is degraded by sparseness and redundancy.
when_to_use_negative:
- When the goal is to retrieve existing literature-curated interactions rather than
  infer new associations from omics data — existing high-quality networks are heavily
  biased toward well-studied proteins.
- When the expression matrix is small/sparse such that plain Pearson correlation on
  the raw data is adequate and a VAE adds unnecessary complexity (though the paper
  shows raw PCC underperforms on typical scRNA-seq).
- When input is not a high-dimensional expression matrix (e.g. a pre-computed network,
  sequence data, or an already dimensionality-reduced feature table).
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0121
tools:
- name: favapy
  role: pip-installable CLI that runs the VAE and outputs ranked protein pairs by
    latent-space Pearson correlation; options include -c (score range 1 to -1) and
    -n (number of output interactions)
  repo: https://github.com/mikelkou/fava
- name: FAVA
  role: 'The method implementation: Functional Associations using Variational Autoencoders
    that compresses high-dimensional data into a low-dimensional latent space'
  repo: https://github.com/mikelkou/fava
- name: AnnData
  role: Input data structure used by FAVA within the scverse ecosystem
  repo: https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:fb4e4398e176166e035d49efe0ebc799f015a0b87c2aad4c9a8961182df65e22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA
  - PCC
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# variational-autoencoder-latent-space-correlation

## Summary

Infers protein functional association networks by compressing high-dimensional scRNA-seq or proteomics data into a dense, low-dimensional VAE latent space (via favapy) and ranking protein pairs by Pearson correlation of their latent representations. Rationale: raw omics data are too sparse and redundant for direct co-expression analysis.

## When to use

Use when you have a high-dimensional scRNA-seq or proteomics expression matrix (e.g. tsv/csv of gene x condition values, AnnData-compatible) and need a ranked protein-protein association network, especially when simple co-expression on the raw matrix is degraded by sparseness and redundancy.

## When NOT to use

- When the goal is to retrieve existing literature-curated interactions rather than infer new associations from omics data — existing high-quality networks are heavily biased toward well-studied proteins.
- When the expression matrix is small/sparse such that plain Pearson correlation on the raw data is adequate and a VAE adds unnecessary complexity (though the paper shows raw PCC underperforms on typical scRNA-seq).
- When input is not a high-dimensional expression matrix (e.g. a pre-computed network, sequence data, or an already dimensionality-reduced feature table).

## Inputs

- scRNA-seq or proteomics expression matrix (tsv/csv) of protein/gene abundances across conditions or cells
- AnnData object (favapy functions within the scverse ecosystem using AnnData as its input source)

## Outputs

- Ranked protein-pair list with latent-space Pearson correlation scores from +1 to -1 (via -c)
- Top-N interaction list controlled by -n (both directions proteinA-proteinB and proteinB-proteinA counted)
- Saved output file from the favapy run at the specified output path

## How to apply

Install favapy from pip and run the CLI as 'favapy <path-to-data-file> <path-to-save-output>' on the expression matrix (AnnData is the input source within the scverse ecosystem); keep the defaults of 50 epochs and batch size 32, with hidden/latent dimensions auto-scaled. The VAE compresses the high-dimensional input into a dense latent space, then Pearson correlation scores between protein pairs in that latent space are used to rank interactions from +1 (high correlation) to -1 (high anti-correlation) via the -c option. Truncate the ranked list to a fixed depth (e.g. top 10000 interactions using the -n option, noting favapy counts both proteinA-proteinB and proteinB-proteinA directions) before any downstream benchmarking. Validate by benchmarking the top-N list against curated references such as KEGG (Release 95) or BioGRID (Release 4.4.217), computing TP/FP ratios and comparing against baselines like scLink, hdWGCNA, and plain PCC on the raw data.

## Related tools

- **favapy** (pip-installable CLI that runs the VAE and outputs ranked protein pairs by latent-space Pearson correlation; options include -c (score range 1 to -1) and -n (number of output interactions)) — https://github.com/mikelkou/fava
- **FAVA** (The method implementation: Functional Associations using Variational Autoencoders that compresses high-dimensional data into a low-dimensional latent space) — https://github.com/mikelkou/fava
- **AnnData** (Input data structure used by FAVA within the scverse ecosystem) — https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy

## Examples

```
pip install favapy; favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- The output is a ranked protein-pair list with correlation scores spanning +1 (high correlation) to -1 (high anti-correlation).
- Default run parameters match the paper: 50 epochs, batch size 32, auto-scaled hidden/latent dimensions.
- Truncating to the same top 10000 interactions should reproduce benchmark TP/FP behavior, e.g. KEGG TP/FP ratio 3.9 (4588/1168) for FAVA on the small PBMC dataset vs 1.0 (1878/1860) for scLink.
- FAVA should still complete on very large inputs (over 0.5 million conditions) where baselines like scLink fail (scLink could not process the large PBMC dataset).
- Latent-space correlations should outperform correlations computed on the original high-dimensional representation.

## Limitations

- Performance depends on reference quality: benchmarks against KEGG and BioGRID inherit the literature bias where well-studied proteins have many more interactions.
- Inferring networks directly from raw omics co-expression does not work well due to sparseness (transcripts/proteins not consistently observed per cell) and redundancy (many similar cells/samples); the VAE step is essential, not optional.
- Datasets lacking sufficient conditions or coherent protein identifiers (as expected by AnnData/favapy input) may not map to meaningful protein-pair correlations.

## Evidence

- [abstract] FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space: "we have developed FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space"
- [intro] latent-space correlation outperforms original representation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [methods] favapy CLI invocation: "Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>"
- [abstract] AnnData as input source: "FAVA functions within the scverse ecosystem, employing AnnData as its input source"
- [intro] raw co-expression rationale: "this approach does not work well due to high sparseness (many transcripts/proteins are not consistently observed in each cell/sample) and redundancy (many similar cells/samples are analyzed)"
