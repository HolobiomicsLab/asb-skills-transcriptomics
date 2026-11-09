---
name: de-method-benchmark-evaluation
description: Use when evaluating which DE detection method performs on scRNA-Seq data
  and a simulator (SymSim) can provide gold-standard differential gene labels and
  true log fold changes for the comparison.
when_to_use_negative:
- The goal is trajectory inference rather than DE detection — use trajectory evaluation
  frameworks such as dynwrap/dynmethods with dyneval instead.
- Only real data with no gold-standard DE labels are available — simulation-based
  truth (differential EVFs defining DE genes) is required for AUROC and logFC correlation
  scoring.
- Data are UMI-based — this benchmark uses nonUMI protocol counts (True2ObservedCounts
  with protocol='nonUMI'); UMI-based simulation settings are a different configuration.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_3170
tools:
- name: SymSim
  role: Simulates true transcript counts with differential EVFs (SimulateTrueCounts),
    gold-standard DE via getDEgenes, and observed non-UMI counts via True2ObservedCounts
  repo: https://github.com/YosefLab/SymSim
- name: edgeR
  role: Differential expression detection method benchmarked; best overall DE detection
    performance
- name: DESeq2
  role: Differential expression detection method benchmarked on simulated observed
    counts
- name: RJAGS
  role: Gibbs sampler used in the SymSim workflow to estimate kinetic parameters from
    experimental data (MCMC, 2000 iterations)
- name: PCA
  role: Dimensionality reduction method applied in the article to cluster cells simulated
    from multiple discrete populations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:627c02ed2dd179c856f8d6a91ec3d0a2710ce7085f5721da241ee71149516d2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - DESeq2
  - PCA
  - RJAGS
  - SymSim
  - edgeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# de-method-benchmark-evaluation

## Summary

Benchmark differential expression detection methods (edgeR, DESeq2, Wilcoxon rank-sum, t-test) on SymSim-simulated single-cell RNA-Seq data with gold-standard DE labels derived from true transcript counts, scoring each method by AUROC and negative Spearman correlation of p-values against true log fold change.

## When to use

Use when evaluating which DE detection method performs best on scRNA-Seq data and a simulator (SymSim) can provide gold-standard differential gene labels and true log fold changes for the comparison.

## When NOT to use

- The goal is trajectory inference rather than DE detection — use trajectory evaluation frameworks such as dynwrap/dynmethods with dyneval instead.
- Only real data with no gold-standard DE labels are available — simulation-based truth (differential EVFs defining DE genes) is required for AUROC and logFC correlation scoring.
- Data are UMI-based — this benchmark uses nonUMI protocol counts (True2ObservedCounts with protocol='nonUMI'); UMI-based simulation settings are a different configuration.

## Inputs

- SymSim true transcript counts simulated from five discrete populations (SimulateTrueCounts output)
- Gold-standard DE gene sets per population pair from getDEgenes (nDiffEVF, logFC_theoretical, adjusted Wilcoxon p-value on true counts)
- Observed (non-UMI) counts from True2ObservedCounts across grids of cell number N and capture efficiency alpha_mean
- Gene lengths sampled from the gene_len_pool dataset of human transcript lengths

## Outputs

- AUROC values for each of edgeR, DESeq2, Wilcoxon rank-sum, and t-test per population comparison and simulation setting
- Negative Spearman correlation between each method's p-values and true log fold change
- Performance plots versus N and alpha with DE gene set sizes (Fig. 6c-e style PNG figures)
- Ranked verdict of DE method performance (edgeR best, t-test second, Wilcoxon third)

## How to apply

Simulate true transcript counts for cells from five discrete populations using SymSim's SimulateTrueCounts with evf_type='discrete', nevf=10, n_de_evf=9, vary='s', Sigma=0.4, phyla=Phyla5(), and randseed=0, so differential extrinsic variation factors (EVFs) define the biological signal. Assign gene lengths by sampling without replacement from the gene_len_pool dataset of human transcript lengths, and extract gold-standard DE information for population pairs 1v2, 1v4, and 2v4 with getDEgenes (recording nDiffEVF, logFC_theoretical, and the adjusted Wilcoxon p-value on true counts). Convert true counts to observed counts with True2ObservedCounts under a nonUMI protocol (alpha_mean=0.1, alpha_sd=0.05, depth_mean=1e5, depth_sd=3e3), sweeping cell number N and capture efficiency alpha_mean across a grid. Run edgeR, DESeq2, Wilcoxon rank-sum, and t-test on each observed-count dataset and score them by AUROC and by the negative Spearman correlation between predicted p-values and true log fold change, plotting performance versus N and alpha (Fig. 6c-e). The gold standard is trustworthy because SymSim explicitly models intrinsic transcription noise, extrinsic cell-state variation, and technical variation, so DE truth is known by construction from the simulated EVFs.

## Related tools

- **SymSim** (Simulates true transcript counts with differential EVFs (SimulateTrueCounts), gold-standard DE via getDEgenes, and observed non-UMI counts via True2ObservedCounts) — https://github.com/YosefLab/SymSim
- **edgeR** (Differential expression detection method benchmarked; best overall DE detection performance)
- **DESeq2** (Differential expression detection method benchmarked on simulated observed counts)
- **RJAGS** (Gibbs sampler used in the SymSim workflow to estimate kinetic parameters from experimental data (MCMC, 2000 iterations))
- **PCA** (Dimensionality reduction method applied in the article to cluster cells simulated from multiple discrete populations)

## Examples

```
devtools::install_github('YosefLab/SymSim'); true_counts_res <- SimulateTrueCounts(evf_type='discrete', nevf=10, n_de_evf=9, vary='s', Sigma=0.4, phyla=Phyla5(), randseed=0); de <- getDEgenes(true_counts_res, pop1=1, pop2=2)
```

## Evaluation signals

- Output figures exist as valid PNG files reproducing the qualitative pattern of Fig. 6c-e.
- AUROC / negative Spearman trends separate the three population comparisons (1v2, 1v4, 2v4) in the order expected from their DiffEVF counts.
- DE gene set sizes tabulated from getDEgenes (nDiffEVF-based) match the population-pair separability implied by the tree with n_de_evf=9.
- edgeR ranks best overall, t-test second, Wilcoxon third — consistent with the prior evaluation of 36 DE methods for single-cell RNA-Seq.
- Observed counts reproduce non-UMI properties (increased noise, high zero abundance) via alpha_mean=0.1, alpha_sd=0.05, depth_mean=1e5, depth_sd=3e3.

## Limitations

- The SymSim simulation pipeline omits several steps, including reverse transcription and library cleanup.
- SymSim is only as informative as its fitted parameters; kinetic parameter fitting assumes all cells in a cluster share the same kinetic parameters.
- Results reflect the specific nonUMI simulation settings and the SymSim generative model (differential EVFs, discrete populations); rankings may differ on real datasets without gold standards.

## Evidence

- [results] edgeR has the best overall DE detection performance, with the t-test ranked second followed by the Wilcoxon test, consistent with a prior evaluation of 36 DE methods for single-cell RNA-Seq.: [evidence span withheld — anchor status: not_found]
- [discussion] AUROC and negative Spearman metrics: "We use two metrics to evaluate the performance of a DE method: (a) AUROC (area under receiver operating characteristic curve)"
- [methods] SimulateTrueCounts and True2ObservedCounts as core functions: "**SimulateTrueCounts( )** and **True2ObservedCounts( )** are the two major functions to generate datasets."
- [methods] getDEgenes provides gold-standard DE information: "Function **getDEgenes( )** can be used to obtain differential expression information for genes, given two populations."
- [intro] SymSim recapitulates data properties without forcing them: "SymSim recapitulates properties of the data (e.g., high abundance of zeros or increased noise in non-UMI protocols) without the need to explicitly force them as factors in a distributional model"
- [readme] SymSim citation and install path: [evidence span withheld — anchor status: external_doc]
