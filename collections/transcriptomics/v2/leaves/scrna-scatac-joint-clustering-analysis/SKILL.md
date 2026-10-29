---
name: scrna-scatac-joint-clustering-analysis
description: Use when scRNA-seq expression matrix (E) and scATAC-seq openness matrix
  (O) come from separate samples of the same mixed cell population, and clustering
  should be informed across datatypes — optionally with bulk H3K27ac HiChIP loops.
when_to_use_negative:
- The single-cell samples are not from the same heterogeneous cell population (no
  shared subpopulations to couple across scRNA-seq and scATAC-seq).
- Only bulk HiChIP deconvolution is needed without any single-cell data — DC3 requires
  the E and O matrices.
- Single-cell data types are jointly profiled in the same cells (e.g. sci-CAR), which
  DC3's cost function does not yet incorporate.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
tools:
- name: DC3
  role: Performs simultaneous deconvolution and coupled clustering of scRNA-seq, scATAC-seq,
    and bulk HiChIP data to assign single cells to subpopulations
  repo: github.com/SUwonglab/DC3
- name: HiC-pro
  role: Recommended preprocessing of HiChIP data into enhancer-promoter interactions
    before DC3 input
- name: hichipper
  role: Recommended preprocessing of HiChIP data into enhancer-promoter interactions
    before DC3 input
- name: MACS
  role: Listed as a dependency of the DC3 pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:20cc7dc466aab555e4c2610abc8c0a4265a2a59a13b35ff9277b445d25505470
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  - HiC-pro
  - MACS
  - hichipper
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# scrna-scatac-joint-clustering-analysis

## Summary

Coupled clustering of scRNA-seq and scATAC-seq samples from the same heterogeneous cell population using DC3, which jointly analyzes the two single-cell matrices together with bulk HiChIP loop counts to assign single cells to subpopulations more accurately than per-datatype NMF.

## When to use

Use when scRNA-seq expression matrix (E) and scATAC-seq openness matrix (O) come from separate samples of the same mixed cell population, and clustering should be informed across datatypes — optionally with bulk H3K27ac HiChIP loops.

## When NOT to use

- The single-cell samples are not from the same heterogeneous cell population (no shared subpopulations to couple across scRNA-seq and scATAC-seq).
- Only bulk HiChIP deconvolution is needed without any single-cell data — DC3 requires the E and O matrices.
- Single-cell data types are jointly profiled in the same cells (e.g. sci-CAR), which DC3's cost function does not yet incorporate.

## Inputs

- scRNA-seq expression matrix E (rows = genes, columns = cells)
- scATAC-seq openness matrix O (rows = enhancers, columns = cells), produced by the provided preprocessing script from per-cell .bam files
- gene symbol file (G_symbol)
- enhancer symbol file (E_symbol)
- bulk H3K27ac HiChIP enhancer-promoter interaction file (gene index, enhancer index, loop counts), restricted to genes in E and enhancers in O
- number of clusters k

## Outputs

- scRNA-result.txt — scRNA-seq clustering (cell-to-subpopulation) assignments
- scATAC-result.txt — scATAC-seq clustering assignments
- cluster-hichip.txt — cluster-specific (subpopulation-specific) HiChIP loop counts
- cluster-specific-peaks-genes-pairs.txt — cluster-specific peak-gene pairs with p-values for gene and peak

## How to apply

Build the scRNA-seq expression matrix E (genes × cells) and the scATAC-seq openness matrix O (enhancers × cells) from the same heterogeneous population, plus a HiChIP enhancer-promoter interaction file whose gene/enhancer indices are restricted to those in E and O (preprocessed with HiC-pro and hichipper). Run DC3 via dc3.py with -k set to the expected number of subpopulations, passing -E, -O, -G_symbol, -E_symbol, and -hichip, optionally tuning lambda1 (NMF term weight) and lambda2 (coupling term weight); if omitted, DC3 selects the best hyperparameters automatically. Because NMF is initialization-dependent, repeat the run (e.g. 50 independent runs) and average results. Compare cluster assignments against known labels using classification error rate against coupled NMF (no loop data) and plain NMF (per-datatype clustering) baselines; in the GM12878/K562 in silico mixture, adding loop data cut error more than four-fold at ≥80% dropout.

## Related tools

- **DC3** (Performs simultaneous deconvolution and coupled clustering of scRNA-seq, scATAC-seq, and bulk HiChIP data to assign single cells to subpopulations) — github.com/SUwonglab/DC3
- **HiC-pro** (Recommended preprocessing of HiChIP data into enhancer-promoter interactions before DC3 input)
- **hichipper** (Recommended preprocessing of HiChIP data into enhancer-promoter interactions before DC3 input)
- **MACS** (Listed as a dependency of the DC3 pipeline)

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- Classification error rate between true subpopulation labels and predicted cluster assignments, averaged over scRNA-seq and scATAC-seq samples and over 50 independent runs, should drop when loop data are included (DC3 vs coupled NMF).
- At ≥80% single-cell dropout in the GM12878/K562 in silico mixture, DC3 error should be more than four-fold lower than clustering without loop data.
- DC3 must output all four files (scRNA-result.txt, scATAC-result.txt, cluster-hichip.txt, cluster-specific-peaks-genes-pairs.txt) with consistent cluster count -k across outputs.
- Input HiChIP file indices must fall within the gene and enhancer lists supplied via -G_symbol and -E_symbol (restriction check).

## Limitations

- DC3 assumes scRNA-seq and scATAC-seq are measured on separate samples of the same population; jointly profiled single-cell data (e.g. sci-CAR) are not yet supported in its cost function.
- NMF-based clustering is initialization-sensitive, so results vary across runs and require multiple independent runs (e.g. 50) to estimate stable error rates.
- At high single-cell dropout rates, clustering without loop data degrades sharply; loop information is what stabilizes assignments at ≥80% dropout.

## Evidence

- [abstract] DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data: "DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data"
- [readme] README on inputs and parameters: "restrict HiChIP enhancer-promoter interactions in the genes in scRNA-seq data and enhancers in scATAC-seq data. The first column indicates the index of gene, the second column indicates the index of"
- [readme] README example invocation: "python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt"
- [results] Task card on dropout benchmark: [evidence span withheld — anchor status: not_found]
- [discussion] Discussion on future data types: "In the future we will modify the cost function of DC3 to incorporate data types from such emerging single cell experiments"
