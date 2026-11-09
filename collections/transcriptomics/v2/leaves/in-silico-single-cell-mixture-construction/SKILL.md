---
name: in-silico-single-cell-mixture-construction
description: Use when benchmarking deconvolution or coupled clustering of bulk/single-cell
  RNA-seq, ATAC-seq, and HiChIP data and true subpopulation-specific profiles are
  needed as ground truth for computing PCC against predicted profiles.
when_to_use_negative:
- The dataset is a truly heterogeneous in vivo population (e.g., retinoic-acid induced
  mouse embryonic stem cells) without ground-truth subpopulation profiles — validation
  must rely on FACS-marker consistency rather than direct PCC against known profiles
- Single-cell and bulk data come from different, non-matched populations, so the coupled-clustering
  assumption that samples share the same subpopulations is violated
- Jointly profiled single-cell multiomics data (e.g., sci-CAR) are available — DC3
  does not yet incorporate such data types per the discussion
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: DC3
  role: De-convolution and coupled-clustering method run on the in silico mixture
    via dc3.py to produce subpopulation-specific profiles whose PCC against ground
    truth is evaluated
  repo: https://github.com/SUwonglab/DC3
- name: scikit-learn
  role: Required dependency of dc3.py used for NMF-based coupled clustering of the
    pooled single-cell matrices
  repo: https://github.com/SUwonglab/DC3
- name: pandas
  role: Required dependency for loading/merging E, O, and hichip interaction files
    during mixture assembly
  repo: https://github.com/SUwonglab/DC3
- name: scipy
  role: Required dependency for numerical optimization and PCC computation across
    the 50 independent runs
  repo: https://github.com/SUwonglab/DC3
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:02ad917e9f1531ae12e06c62033513666396060b5ee043d01101b6da801d9b55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/DC3
  tools:
  - DC3
  - pandas
  - scikit-learn
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# in-silico-single-cell-mixture-construction

## Summary

Constructs a synthetic heterogeneous cell population by pooling single cells from two known subpopulations (e.g., 73+73 GM12878/K562 scRNA-seq cells, 373+373 scATAC-seq cells, 100+100 scHi-C cells), providing ground-truth subpopulation profiles to benchmark deconvolution methods like DC3.

## When to use

Use when benchmarking deconvolution or coupled clustering of bulk/single-cell RNA-seq, ATAC-seq, and HiChIP data and true subpopulation-specific profiles are needed as ground truth for computing PCC against predicted profiles.

## When NOT to use

- The dataset is a truly heterogeneous in vivo population (e.g., retinoic-acid induced mouse embryonic stem cells) without ground-truth subpopulation profiles — validation must rely on FACS-marker consistency rather than direct PCC against known profiles
- Single-cell and bulk data come from different, non-matched populations, so the coupled-clustering assumption that samples share the same subpopulations is violated
- Jointly profiled single-cell multiomics data (e.g., sci-CAR) are available — DC3 does not yet incorporate such data types per the discussion

## Inputs

- scRNA-seq expression matrix E (genes x cells; exampledata/E.txt) from GM12878 and K562 cells
- scATAC-seq openness matrix O (exampledata/O.txt) and enhancer symbol file (exampledata/enhancer.txt) from the two cell lines
- Bulk H3K27ac HiChIP enhancer-promoter interaction counts (gene index, enhancer index, loop counts; exampledata/hichip.txt)
- Gene symbol file (exampledata/gene.txt)
- Number of subpopulations k (k=2 for the GM12878/K562 mixture)

## Outputs

- In silico two-subpopulation mixture with known cell-to-subpopulation assignments (73+73 scRNA-seq cells, 373+373 scATAC-seq cells, 100+100 scHi-C cells)
- Mean PCC values (averaged over 50 runs) for subpopulation-specific HiChIP, RNA-seq, and ATAC-seq deconvolution per input setting
- 4x3 mean-PCC table (four input combinations x three data types; e.g., HiChIP 0.78-0.95, RNA-seq 0.85-0.99, ATAC-seq 0.88-0.99)

## How to apply

Pool single cells from two well-characterized cell lines (GM12878 and K562) into an in silico mixture using the paper's counts: n1=146 scRNA-seq cells (73 GM12878 + 73 K562) for the expression matrix E, n2=746 scATAC-seq cells (373+373) for the openness matrix O, and n3=200 scHi-C cells (100+100). Preprocess scRNA-seq into a gene-by-cell expression matrix and scATAC-seq .bam files into an openness matrix O via the provided preprocessing script, then restrict HiChIP enhancer-promoter interactions to the genes in the scRNA-seq data and enhancers in the scATAC-seq data (gene index, enhancer index, loop counts in hichip.txt). Run DC3 (dc3.py) with k=2 clusters under each of the four input-combination settings (scRNA/scATAC/scHi-C; scRNA/scATAC/bulk HiChIP; scRNA/bulk ATAC/bulk HiChIP; bulk RNA/scATAC/bulk HiChIP), setting lambda1 and lambda2 hyperparameters (e.g., lambda1=10000, lambda2=0.0005). Execute 50 independent runs per setting, stopping iterations when the relative error falls below 10^-4, then compute Pearson correlation coefficients between true and predicted subpopulation-specific HiChIP, RNA-seq, and ATAC-seq profiles and average over runs to form a 4x3 mean-PCC table.

## Related tools

- **DC3** (De-convolution and coupled-clustering method run on the in silico mixture via dc3.py to produce subpopulation-specific profiles whose PCC against ground truth is evaluated) — https://github.com/SUwonglab/DC3
- **scikit-learn** (Required dependency of dc3.py used for NMF-based coupled clustering of the pooled single-cell matrices) — https://github.com/SUwonglab/DC3
- **pandas** (Required dependency for loading/merging E, O, and hichip interaction files during mixture assembly) — https://github.com/SUwonglab/DC3
- **scipy** (Required dependency for numerical optimization and PCC computation across the 50 independent runs) — https://github.com/SUwonglab/DC3

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- Each cell of the reproduced 4x3 mean-PCC table falls within the reported ranges: HiChIP 0.78-0.95, RNA-seq 0.85-0.99, ATAC-seq 0.88-0.99 (e.g., setting-2 HiChIP PCC 0.86-0.95)
- All mean PCC values exceed the random-deconvolution null baseline
- Convergence achieved in each of the 50 runs, with iterations stopped when relative error < 10^-4
- Cell-count balance of the mixture matches the paper's design: 73+73 scRNA-seq cells, 373+373 scATAC-seq cells, 100+100 scHi-C cells
- DC3 output files (scRNA-result.txt, scATAC-result.txt, cluster-hichip.txt, cluster-specific-peaks-genes-pairs.txt) are produced for each run

## Limitations

- Benchmark results depend on initialization; hence 50 independent runs per setting are required and individual-run PCC varies across the reported ranges
- DC3 does not yet handle jointly profiled single-cell data types (e.g., sci-CAR); incorporating them is deferred to future work
- The model-based subpopulation inference method of Lin et al. is not yet incorporated into the inference, also deferred to future work

## Evidence

- [abstract] DC3 simultaneously identifies subpopulations and deconvolves bulk: "DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data"
- [discussion] DC3 decomposes bulk profiles and enhances clustering: "We showed that DC3 can decompose bulk profiles into subpopulation-specific profiles and at the same time enhance clustering performance of the single-cell data."
- [readme] Simulation experiments show HiChIP deconvolution works: "Based on simulation experiments and biological experiments, it is shown that this method can decompose bulk HiChIP loop counts into subpopulation-specific loop counts."
- [readme] HiChIP preprocessing restricts interactions to genes and enhancers: "For preprocessing the HiChIP data, please restrict HiChIP enhancer-promoter interactions in the genes in scRNA-seq data and enhancers in scATAC-seq data."
