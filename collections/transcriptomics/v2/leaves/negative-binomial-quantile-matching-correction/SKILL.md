---
name: negative-binomial-quantile-matching-correction
description: Use when raw read count matrices from multiple scRNA-seq batches must
  be converted into batch-effect-corrected, dropout-imputed counts for downstream
  analysis (t-SNE, clustering, DE) without further cell-specific normalization.
when_to_use_negative:
- The input contains bulk-RNA-seq or already-normalized expression values rather than
  raw scRNA-seq read counts — BUSseq's negative-binomial and dropout modeling assumes
  raw count data
- The experimental design is completely confounded (e.g. batch 1 measures cell types
  1 and 2, batch 2 measures cell types 3 and 4), for which no batch-correction method
  including BUSseq is applicable
- Cell-type/subtype labels are known a priori and a simple bulk-style correction such
  as ComBat would suffice — BUSseq is designed for unknown cell types
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3170
tools:
- name: BUSseq
  role: Fits the Bayesian hierarchical model (BUSseq_MCMC) whose posterior fits supply
    the negative-binomial CDFs for quantile matching via corrected_read_counts()
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: BUSseq-1.0
  role: C++ MCMC implementation producing posterior sampling and inference folders
    that feed the R quantile-matching step
  repo: https://github.com/songfd2018/BUSseq-1.0
- name: BUSseq_gpu
  role: GPU (CUDA C) implementation of the same BUSseq model for faster MCMC sampling
  repo: https://github.com/Anguscgm/BUSseq_gpu
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:4d265578976a4805c8eac242fe656e8a7c3567618cf4348b8790326c6bcd47eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq-Rpackage
  tools:
  - BUSseq
  - BUSseq-1.0
  - BUSseq_gpu
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# negative-binomial-quantile-matching-correction

## Summary

BUSseq's quantile-matching procedure converts observed scRNA-seq read counts into batch-effect-free negative-binomial counts whose means depend only on cell type, yielding data usable for downstream analysis as if measured in a single batch. It removes batch location/scale effects and imputes dropout events using the full fitted hierarchical model.

## When to use

Use when raw read count matrices from multiple scRNA-seq batches must be converted into batch-effect-corrected, dropout-imputed counts for downstream analysis (t-SNE, clustering, DE) without further cell-specific normalization.

## When NOT to use

- The input contains bulk-RNA-seq or already-normalized expression values rather than raw scRNA-seq read counts — BUSseq's negative-binomial and dropout modeling assumes raw count data
- The experimental design is completely confounded (e.g. batch 1 measures cell types 1 and 2, batch 2 measures cell types 3 and 4), for which no batch-correction method including BUSseq is applicable
- Cell-type/subtype labels are known a priori and a simple bulk-style correction such as ComBat would suffice — BUSseq is designed for unknown cell types

## Inputs

- Fitted BUSseq model object (e.g. sce_BUSseqfit / BUSseqfits_example output from BUSseq_MCMC)
- Raw G-by-N read count matrices across batches (e.g. count_data_demo_v1.txt from the BUSseq demo)
- Dimension file (dim_demo_v1.txt: number of cells, genes, batches, per-batch cell counts)
- Inferred cell-type labels per cell (celltypes(sce_BUSseqfit))

## Outputs

- Corrected read count matrix (via corrected_read_counts(sce_BUSseqfit), exportable as corrected_read_counts.csv)
- t-SNE embedding of corrected counts colored by cell type and batch (tsne_corrected_counts.png)
- Batch-effect-free counts requiring no further cell-specific normalization

## How to apply

Load the fitted BUSseq model object (e.g. BUSseqfits_example from BUSseq_MCMC) together with the raw G-by-N read count matrices in R. For each gene-cell pair, compute the negative-binomial CDF values of the observed count under the full fitted model, which incorporates fitted cell-type means, batch location effects, cell-specific sequencing depth (size factors), overdispersion, and dropout terms. Draw u uniformly between those two CDF values and map u to the quantile of the batch-effect-free negative-binomial distribution whose mean depends only on the cell type, using corrected_read_counts(sce_BUSseqfit); write the corrected matrix (e.g. corrected_read_counts.csv). Retrieve cell labels with celltypes(sce_BUSseqfit), run t-SNE on the corrected matrix coloring by cell type and batch, and confirm cells group by cell type with no batch separation, validating that corrected values can be analyzed as if all cells were sequenced in a single batch.

## Related tools

- **BUSseq** (Fits the Bayesian hierarchical model (BUSseq_MCMC) whose posterior fits supply the negative-binomial CDFs for quantile matching via corrected_read_counts()) — https://github.com/songfd2018/BUSseq-Rpackage
- **BUSseq-1.0** (C++ MCMC implementation producing posterior sampling and inference folders that feed the R quantile-matching step) — https://github.com/songfd2018/BUSseq-1.0
- **BUSseq_gpu** (GPU (CUDA C) implementation of the same BUSseq model for faster MCMC sampling) — https://github.com/Anguscgm/BUSseq_gpu

## Examples

```
R: corrected_counts <- corrected_read_counts(sce_BUSseqfit); celltypes(sce_BUSseqfit); write.csv(corrected_counts, 'corrected_read_counts.csv')
```

## Evaluation signals

- t-SNE of corrected_read_counts colored by batch shows no separation by batch while coloring by cell type shows clean grouping by cell type (tsne_corrected_counts.png)
- Corrected counts require no further cell-specific normalization — their cell-type means should match the batch-effect-free negative-binomial distribution
- Dropout-caused zero-inflation present in raw counts is absent in corrected_read_counts output (imputed values)
- The same corrected counts support downstream analyses (clustering, DE) as if all cells were sequenced in a single batch
- Running the quantile mapping on the demo (450 cells, 1000 genes, 3 batches of 150) reproduces batch-free embeddings comparable to the uncorrected-data t-SNE controls

## Limitations

- The correction depends on the fitted model and inferred cell labels; clustering errors in prior stages can propagate and lead to false positives and false negatives
- MCMC inference is computationally heavy; most steps are parallelizable but runtime can be substantial (demo: ~12 min for 4 K values on 4 cores; GPU version exists for acceleration)
- Not applicable to completely confounded experimental designs where no method can separate batch from biology
- The C++ source supports Linux only (tested on Ubuntu 18.04 and CentOS 7, R 3.6.0) and requires installing RngStreams

## Evidence

- [abstract] BUSseq can simultaneously correct batch effects, cluster cell types, impute dropout-caused missing data, and detect differentially expressed genes without preliminary normalization: [evidence span withheld — anchor status: not_found]
- [methods] Corrected counts usable downstream: "the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch"
- [methods] corrected_read_counts call: "- `corrected_read_counts(sce_BUSseqfit)`"
- [methods] MCMC entry point: "`BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p,"
- [readme] Model handles count nature and dropout: "takes care of the count data nature, the overdispersion, the dropout events, and the cell-specific sequencing depth of scRNA-seq data"
