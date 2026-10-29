---
name: coupled-nmf-multi-omic-clustering
description: Use when scRNA-seq, scATAC-seq, and bulk H3K27ac HiChIP samples from
  the same heterogeneous cell population are available and the goal is coupled clustering
  plus deconvolution of bulk HiChIP/RNA/ATAC signals into subpopulation-specific profiles.
when_to_use_negative:
- Bulk and single-cell data do not come from the same heterogeneous cell population,
  so coupled clustering across samples is not justified
- Only a single data type is available (e.g. scRNA-seq alone) with no coupled bulk/single-cell
  ATAC or HiChIP signal to constrain the joint NMF
- Jointly-profiled single-cell multi-omic data (e.g. sci-CAR) is the input — the article
  explicitly defers incorporating such data to future work
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3611
tools:
- name: DC3
  role: 'Core method: simultaneous deconvolution and coupled clustering of bulk and
    single-cell RNA-seq, ATAC-seq, and HiChIP data via coupled NMF'
  repo: https://github.com/SUwonglab/DC3
- name: scikit-learn
  role: Dependency used by dc3.py for clustering/NMF computations
- name: pandas
  role: Dependency for loading and manipulating the E, O, and HiChIP matrices in dc3.py
- name: scipy
  role: Dependency for numerical routines in dc3.py
- name: hichipper
  role: Recommended upstream preprocessing of HiChIP data before restricting interactions
    to genes and enhancers
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:1f4a261e02b9506864eaa3d32bc0066ca3781ede5b4cb83508d06ae8925cd878
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/DC3
  tools:
  - DC3
  - hichipper
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3611
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# coupled-nmf-multi-omic-clustering

## Summary

DC3 (De-Convolution and Coupled-Clustering) jointly analyzes bulk and single-cell RNA-seq, ATAC-seq, and HiChIP data from the same heterogeneous population, simultaneously clustering single cells and deconvolving bulk profiles into subpopulation-specific gene expression, chromatin accessibility, and enhancer-promoter contact profiles.

## When to use

Use when scRNA-seq, scATAC-seq, and bulk H3K27ac HiChIP samples from the same heterogeneous cell population are available and the goal is coupled clustering plus deconvolution of bulk HiChIP/RNA/ATAC signals into subpopulation-specific profiles.

## When NOT to use

- Bulk and single-cell data do not come from the same heterogeneous cell population, so coupled clustering across samples is not justified
- Only a single data type is available (e.g. scRNA-seq alone) with no coupled bulk/single-cell ATAC or HiChIP signal to constrain the joint NMF
- Jointly-profiled single-cell multi-omic data (e.g. sci-CAR) is the input — the article explicitly defers incorporating such data to future work

## Inputs

- scRNA-seq expression matrix E (genes x cells), e.g. exampledata/E.txt
- scATAC-seq openness matrix O plus enhancer symbol file, e.g. exampledata/O.txt and enhancer.txt
- gene symbol file, e.g. exampledata/gene.txt
- HiChIP enhancer-promoter interaction file (gene index, enhancer index, loop counts), e.g. exampledata/hichip.txt

## Outputs

- scATAC-result.txt — clustering assignments for scATAC-seq cells
- scRNA-result.txt — clustering assignments for scRNA-seq cells
- cluster-hichip.txt — cluster (subpopulation)-specific HiChIP loop counts
- cluster-specific-peaks-genes-pairs.txt — cluster-specific peak-gene pairs with gene and peak p-values

## How to apply

Prepare the expression matrix E (rows = genes, columns = cells), the openness matrix O with enhancer names from the provided scATAC preprocessing script, and a HiChIP interaction file restricted to genes in E and enhancers in O (columns: gene index, enhancer index, loop counts; HiC-pro and hichipper recommended upstream). Run dc3.py with -k set to the expected number of subpopulations (k=2 for the GM12878/K562 benchmark), passing E.txt, O.txt, gene.txt, enhancer.txt, and hichip.txt; optionally set the NMF hyper-parameters -lambda1 (E-matrix term) and -lambda2 (coupled term), e.g. lambda1=10000 and lambda2=0.0005, or omit them to let dcHiChIP choose the best parameters automatically. Optimization iterations stop when the relative error falls below 10^-4; for benchmarking, execute multiple independent runs (e.g. 50) with different initializations under the four input-combination settings and average the per-run Pearson correlation coefficients against true subpopulation-specific profiles. Correct application yields four outputs per run: scRNA and scATAC cluster assignments, cluster-specific HiChIP, and cluster-specific peak-gene pairs with p-values.

## Related tools

- **DC3** (Core method: simultaneous deconvolution and coupled clustering of bulk and single-cell RNA-seq, ATAC-seq, and HiChIP data via coupled NMF) — https://github.com/SUwonglab/DC3
- **scikit-learn** (Dependency used by dc3.py for clustering/NMF computations)
- **pandas** (Dependency for loading and manipulating the E, O, and HiChIP matrices in dc3.py)
- **scipy** (Dependency for numerical routines in dc3.py)
- **hichipper** (Recommended upstream preprocessing of HiChIP data before restricting interactions to genes and enhancers)

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- Pearson correlation coefficients between true and predicted subpopulation-specific profiles fall within the Table 1 ranges: HiChIP 0.78-0.95, RNA-seq 0.85-0.99, ATAC-seq 0.88-0.99 across the four input-combination settings in the in silico GM12878/K562 mixture
- Optimization stops only when the relative error per iteration drops below 10^-4
- All four expected output files (scATAC-result.txt, scRNA-result.txt, cluster-hichip.txt, cluster-specific-peaks-genes-pairs.txt) are produced per run
- Clustering assignments for scRNA-seq and scATAC-seq cells are consistent across the coupled samples (e.g. 73+73 RNA cells and 373+373 ATAC cells split between k=2 subpopulations)
- Deconvolved values exceed a random-deconvolution null baseline

## Limitations

- Jointly-profiled single-cell data types (e.g., sci-CAR) are not yet incorporated into the cost function; deferred to future work
- A model-based subpopulation inference method (Lin et al.) is not yet incorporated into the inference; deferred to future work
- Results depend on hyper-parameters lambda1 (E-matrix NMF term) and lambda2 (coupled term), though DC3 can auto-select them; benchmark scores require multiple independent runs (e.g. 50) with different initializations

## Evidence

- [abstract] we introduce DC3 (De-Convolution and Coupled-Clustering) as a method for the joint analysis of various bulk and single-cell data: "we introduce DC3 (De-Convolution and Coupled-Clustering) as a method for the joint analysis of various bulk and single-cell data"
- [abstract] DC3 simultaneously identifies distinct subpopulations, assigns single cells to subpopulations (clustering), and de-convolves bulk data into subpopulation-specific data: "DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data"
- [abstract] DC3 yields subpopulation-specific profiles of gene expression, chromatin accessibility and enhancer-promoter contact: "The subpopulation-specific profiles of gene expression, chromatin accessibility and enhancer-promoter contact obtained by DC3 provide a comprehensive characterization of the gene regulatory system"
- [readme] README dc3.py example invocation with lambda1 and lambda2 parameters: "python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt  -lambda1 10000 -lambda2 0.0005"
- [readme] README output file list: "DC3 outputs 4 files: * scATAC-result.txt the clustering results for scATAC-seq * scRNA-result.txt the clustering results for scRNA-seq"
