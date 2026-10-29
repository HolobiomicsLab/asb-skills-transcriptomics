---
name: scrna-seq-functional-network-inference
description: Use when building or benchmarking protein functional association networks
  from sparse, redundant scRNA-seq or proteomics expression matrices (tsv/csv), especially
  when simple co-expression correlation underperforms or the dataset exceeds what
  tools like scLink can process.
when_to_use_negative:
- When the input is bulk RNA-seq or already-aggregated sample-level data rather than
  the high-dimensional, sparse single-cell expression data the VAE latent compression
  targets
- When a curated, literature-derived network (e.g. KEGG/BioGRID themselves) already
  suffices and no data-driven network inference is needed
- 'When requiring a comparison method that must run on every dataset: scLink was unable
  to process the large PBMC dataset, so it cannot serve there'
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0605
tools:
- name: FAVA (favapy)
  role: Core variational-autoencoder tool that compresses high-dimensional scRNA-seq/proteomics
    data into a low-dimensional latent space and outputs ranked functional association
    networks; run via CLI or pip-installed Python module
  repo: https://github.com/mikelkou/fava
- name: AnnData
  role: Input data structure used by favapy within the scverse ecosystem
- name: scLink
  role: Baseline method run on the same datasets to generate comparable ranked interaction
    lists; failed on the large PBMC dataset
- name: hdWGCNA
  role: Baseline co-expression network method run on the same five datasets for TP/FP
    benchmarking
- name: PCC
  role: Plain Pearson correlation baseline used to produce comparable ranked interaction
    lists
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:61200d314c174070bf571257e756946c78163a39332663fa9542675fdc5c8f30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA
  - FAVA (favapy)
  - PCC
  - Python
  - favapy
  - hdWGCNA
  - scLink
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# scrna-seq-functional-network-inference

## Summary

Infer protein functional association networks from high-dimensional scRNA-seq data by compressing it into a dense, low-dimensional latent space with a variational autoencoder (FAVA/favapy), then benchmarking the ranked interaction lists against KEGG and BioGRID to quantify TP/FP quality versus co-expression baselines.

## When to use

Use when building or benchmarking protein functional association networks from sparse, redundant scRNA-seq or proteomics expression matrices (tsv/csv), especially when simple co-expression correlation underperforms or the dataset exceeds what tools like scLink can process.

## When NOT to use

- When the input is bulk RNA-seq or already-aggregated sample-level data rather than the high-dimensional, sparse single-cell expression data the VAE latent compression targets
- When a curated, literature-derived network (e.g. KEGG/BioGRID themselves) already suffices and no data-driven network inference is needed
- When requiring a comparison method that must run on every dataset: scLink was unable to process the large PBMC dataset, so it cannot serve there

## Inputs

- scRNA-seq or proteomics expression matrix (tsv/csv) for each dataset (e.g. GBM, lung SCC, small PBMC, pancreatic tumor, large PBMC), usable as AnnData input
- Reference interaction databases: KEGG Release 95 and BioGRID Release 4.4.217

## Outputs

- Ranked protein-protein interaction list per method, truncated to the top 10000 interactions (fava -n), ordered by latent-space Pearson correlation score (-c: 1 high correlation to -1 high anti-correlation)
- Method x dataset x reference TP/FP ratio table written to benchmark_tpfp_ratios.csv

## How to apply

Install favapy via pip and load each expression matrix (e.g. GBM, lung SCC, small PBMC, pancreatic tumor, large PBMC) as tsv/csv input, with AnnData as the scverse-compatible input source. Run favapy from the command line (favapy <path-to-data-file> <path-to-save-output>), keeping default 50 epochs, batch size 32, and auto-scaled hidden/latent dimensions; the VAE compresses the high-dimensional data to reduce sparseness and redundancy, after which protein pairs are ranked by Pearson correlation in the latent space (-c, from 1 for high correlation to -1 for high anti-correlation), with the -n option controlling the number of output interactions (counting both directions proteinA-proteinB and proteinB-proteinA). Truncate every method's ranked list to the same top 10000 interactions so comparisons are fair. Run baselines (scLink, hdWGCNA, plain PCC) on the same matrices — expecting scLink to fail on the large PBMC dataset — and benchmark each top-10000 list against KEGG Release 95 and BioGRID Release 4.4.217 by counting true and false positives and computing TP/FP ratios. Verify outputs against the paper's reference values, e.g. FAVA TP/FP of 3.9 (4588/1168) versus scLink's 1.0 (1878/1860) on small PBMC/KEGG, and FAVA 0.55 (KEGG) / 0.04 (BioGRID) on large PBMC.

## Related tools

- **FAVA (favapy)** (Core variational-autoencoder tool that compresses high-dimensional scRNA-seq/proteomics data into a low-dimensional latent space and outputs ranked functional association networks; run via CLI or pip-installed Python module) — https://github.com/mikelkou/fava
- **AnnData** (Input data structure used by favapy within the scverse ecosystem)
- **scLink** (Baseline method run on the same datasets to generate comparable ranked interaction lists; failed on the large PBMC dataset)
- **hdWGCNA** (Baseline co-expression network method run on the same five datasets for TP/FP benchmarking)
- **PCC** (Plain Pearson correlation baseline used to produce comparable ranked interaction lists)

## Examples

```
pip install favapy; favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- Small PBMC / KEGG: FAVA TP/FP ratio should be 3.9 (4588 TP / 1168 FP), about 3.9x the second-best method scLink (1878/1860 = 1.0)
- Large PBMC: FAVA should reach TP/FP ratios of 0.55 (KEGG) and 0.04 (BioGRID), while scLink is absent on this dataset
- All compared methods' ranked lists must be truncated to exactly the top 10000 interactions before benchmarking (favapy -n counts both directions proteinA-proteinB and proteinB-proteinA)
- Correlation ranking must span -c from 1 (high correlation) to -1 (high anti-correlation) in the VAE latent space
- The full method x dataset x reference ratio table is emitted as benchmark_tpfp_ratios.csv

## Limitations

- FAVA's inferred networks are functional associations from omics data, not direct physical interactions; quality depends on the reference (BioGRID TP/FP ratios are far lower than KEGG's, e.g. 0.04 vs 0.55 on large PBMC)
- Simple co-expression analysis on raw scRNA-seq data underperforms because of high sparseness (transcripts/proteins not consistently observed) and redundancy (many similar cells/samples), which is what the VAE addresses
- Existing high-quality networks are heavily biased by data availability, so benchmarks against them undercount interactions for understudied proteins
- Baseline methods may fail on very large datasets: scLink could not process the large PBMC dataset

## Evidence

- [abstract] FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space: "we have developed FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space"
- [intro] Latent-space correlation improves networks: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [methods] Command-line invocation: "Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>"
- [methods] Installation via pip: "You can install FAVA using pip:

.. code-block:: bash

   pip install favapy"
- [intro] Sparseness/redundancy rationale: "this approach does not work well due to high sparseness (many transcripts/proteins are not consistently observed in each cell/sample) and redundancy (many similar cells/samples are analyzed) of such"
- [abstract] Large dataset capacity: "FAVA can process large datasets with over 0.5 million conditions"
- [abstract] AnnData input source: "FAVA functions within the scverse ecosystem, employing AnnData as its input source"
