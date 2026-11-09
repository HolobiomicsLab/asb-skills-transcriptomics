---
name: multi-input-combination-optimization-config
description: 'Use when reproducing DC3''s Table 1 benchmark: an in silico GM12878/K562
  mixture with scRNA-seq, scATAC-seq, and bulk HiChIP data must be deconvolved under
  four input-combination settings to compare PCC scores.'
when_to_use_negative:
- Input data types are not among the supported combinations of scRNA-seq/scATAC-seq
  and bulk RNA-seq/ATAC-seq/HiChIP from the same heterogeneous population (e.g., jointly-profiled
  sci-CAR data, which the authors deferred to future work)
- The analysis goal is only single-cell clustering without bulk deconvolution — the
  coupled NMF method of Duren et al. alone may suffice
- HiChIP interactions have not been restricted to genes present in the scRNA-seq data
  and enhancers in the scATAC-seq data, as the preprocessing requires
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DC3
  role: Core method executed via dc3.py under each input-combination setting (alpha/beta/gamma
    indicators) to perform coupled clustering and deconvolution of HiChIP, RNA-seq,
    and ATAC-seq profiles
  repo: https://github.com/SUwonglab/DC3
- name: scikit-learn
  role: Dependency for clustering computations within dc3.py
- name: pandas
  role: Dependency for loading and handling the E/O matrices and HiChIP interaction
    files
- name: scipy
  role: Dependency for optimization and correlation computations in dc3.py
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:74644a9cd4ea3a86e8d8862b10771e3fae726140271071da1ecbc9f6941217ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/DC3
  tools:
  - DC3
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# multi-input-combination-optimization-config

## Summary

Configure DC3 (De-Convolution and Coupled-Clustering) to run under different combinations of single-cell (scRNA-seq, scATAC-seq) and bulk (RNA-seq, ATAC-seq, HiChIP) inputs, switching alpha/beta/gamma indicators to test each input-combination setting and benchmark deconvolution accuracy across settings.

## When to use

Use when reproducing DC3's Table 1 benchmark: an in silico GM12878/K562 mixture with scRNA-seq, scATAC-seq, and bulk HiChIP data must be deconvolved under four input-combination settings to compare PCC scores.

## When NOT to use

- Input data types are not among the supported combinations of scRNA-seq/scATAC-seq and bulk RNA-seq/ATAC-seq/HiChIP from the same heterogeneous population (e.g., jointly-profiled sci-CAR data, which the authors deferred to future work)
- The analysis goal is only single-cell clustering without bulk deconvolution — the coupled NMF method of Duren et al. alone may suffice
- HiChIP interactions have not been restricted to genes present in the scRNA-seq data and enhancers in the scATAC-seq data, as the preprocessing requires

## Inputs

- scRNA-seq expression matrix (E.txt) with rows as genes and columns as cells
- scATAC-seq openness matrix (O.txt) and enhancer names (enhancer.txt)
- gene symbol file (gene.txt)
- HiChIP enhancer-promoter interaction file (hichip.txt) with gene index, enhancer index, and loop count columns
- Number of clusters k and NMF hyperparameters lambda1, lambda2 (optional)

## Outputs

- 4x3 mean-PCC table (four input-combination settings x three data types) for comparison against Table 1 (HiChIP 0.78-0.95, RNA-seq 0.85-0.99, ATAC-seq 0.88-0.99)
- scATAC-result.txt and scRNA-result.txt clustering assignments per setting
- cluster-hichip.txt (cluster-specific HiChIP profiles)
- cluster-specific-peaks-genes-pairs.txt with gene, peak, and p-values

## How to apply

Construct the in silico mixture with the paper's cell counts (n1=146 scRNA-seq cells: 73 GM12878 + 73 K562; n2=746 scATAC-seq cells: 373+373; n3=200 scHi-C cells: 100+100) from the packaged exampledata files E.txt, O.txt, gene.txt, enhancer.txt, and hichip.txt. Run dc3.py with k=2 clusters under each of the four input settings by setting the alpha/beta/gamma indicators: (1) scRNA-seq+scATAC-seq+scHi-C, (2) scRNA-seq+scATAC-seq+bulk HiChIP, (3) scRNA-seq+bulk ATAC-seq+bulk HiChIP, (4) bulk RNA-seq+scATAC-seq+bulk HiChIP. Execute 50 independent runs per setting with different initializations, stopping each optimization iteration when the relative error falls below 10^-4. Optional NMF hyperparameters lambda1 and lambda2 (e.g., -lambda1 10000 -lambda2 0.0005) can be supplied; if omitted, DC3 chooses the best parameters automatically. For each run, compute Pearson correlation coefficients (PCC) between true and predicted subpopulation-specific HiChIP, RNA-seq, and ATAC-seq profile vectors, then average over the 50 runs to assemble a 4x3 mean-PCC table.

## Related tools

- **DC3** (Core method executed via dc3.py under each input-combination setting (alpha/beta/gamma indicators) to perform coupled clustering and deconvolution of HiChIP, RNA-seq, and ATAC-seq profiles) — https://github.com/SUwonglab/DC3
- **scikit-learn** (Dependency for clustering computations within dc3.py)
- **pandas** (Dependency for loading and handling the E/O matrices and HiChIP interaction files)
- **scipy** (Dependency for optimization and correlation computations in dc3.py)

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- Every cell of the reproduced 4x3 PCC table falls within the reported Table 1 ranges: HiChIP 0.78-0.95, RNA-seq 0.85-0.99, ATAC-seq 0.88-0.99 (e.g., setting-2 HiChIP PCC of 0.86-0.95)
- All mean PCC values exceed the random-deconvolution null baseline
- Each of the 50 independent runs per setting converges with relative optimization error below 10^-4
- Mixture composition matches the paper's counts (73+73 scRNA-seq, 373+373 scATAC-seq, 100+100 scHi-C cells) and k=2 clusters are recovered for GM12878 and K562
- dc3.py produces all four expected output files (scATAC-result.txt, scRNA-result.txt, cluster-hichip.txt, cluster-specific-peaks-genes-pairs.txt) for each run

## Limitations

- Benchmark requires running 50 independent runs per setting with different initializations, so results vary across initializations and must be averaged
- Jointly-profiled single-cell data types (e.g., sci-CAR) are not yet incorporated into DC3; deferred to future work
- HiChIP inputs must be preprocessed with HiC-pro and hichipper and restricted to scRNA-seq genes and scATAC-seq enhancers, otherwise the interaction file will not align
- If lambda1/lambda2 are not supplied, DC3 auto-selects them, which may differ from the paper's explicit settings (-lambda1 10000 -lambda2 0.0005)

## Evidence

- [readme] README example invocation: "python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt"
- [readme] must-have parameters: "-k, -O, -E, -G_symbol, -E_symbol, -hichip are the must-have parameters; -lambda1, -lamdba2 are optional parameters"
- [readme] HiChIP preprocessing recommendation: "We recommend use HiC-pro and hichipper to preprocess the HiChIP data."
- [abstract] DC3 joint analysis scope: "we introduce DC3 (De-Convolution and Coupled-Clustering) as a method for the joint analysis of various bulk and single-cell data"
- [discussion] joint analysis of RNA-seq, ATAC-seq, HiChIP combinations: "simultaneous deconvolution and coupled clustering based on the joint analysis of different combinations of bulk and single-cell level RNA-seq, ATAC-seq, and HiChIP data"
