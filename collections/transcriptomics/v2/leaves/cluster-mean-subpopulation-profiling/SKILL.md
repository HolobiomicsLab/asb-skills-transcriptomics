---
name: cluster-mean-subpopulation-profiling
description: Use when scRNA-seq (E) and scATAC-seq (O) matrices from the same heterogeneous
  population already carry cell-cluster labels and a bulk data type (e.g., HiChIP
  loop counts) must be seeded with per-cluster mean profiles before deconvolution.
when_to_use_negative:
- 'Single-cell samples are jointly profiled (e.g., sci-CAR-style multi-ome): the article
  defers incorporating such data types to future work, so cluster-mean seeding across
  separate samples is not the right setup'
- No bulk data type is available to deconvolve — if all data types are already at
  single-cell resolution, per-cluster means are unnecessary and discard single-cell
  resolution
- A fully data-driven coupled clustering is required from scratch rather than seeding
  from pre-assigned cluster labels
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DC3
  role: Runs the coupled clustering and deconvolution whose per-cluster mean profiles
    (P1, P2) and multinomial-initialized latent loop counts C_ijk are computed in
    this skill; invoked as dc3.py with -k, -E, -O, -hichip and optional lambda1/lambda2
    hyper-parameters
  repo: github.com/SUwonglab/DC3
- name: HiC-pro / hichipper
  role: Recommended preprocessors to produce the HiChIP enhancer-promoter interaction
    file (gene index, enhancer index, loop counts) that supplies the bulk counts C_ij
    to be allocated by multinomial proportions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:05094d7064e41adaf62c288f7fa4ca93f3f7ee6cf08831bd4450d526c01e6f49
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  - HiC-pro / hichipper
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cluster-mean-subpopulation-profiling

## Summary

Computes per-subpopulation mean profiles (e.g., expression matrix P1 from scRNA-seq and accessibility matrix P2 from scATAC-seq) by averaging single cells sharing a cluster label, then uses multinomial proportions rho_ijk = sqrt(O_ik * O_jk) / sum_k sqrt(O_ik * O_jk) to initialize latent loop counts for deconvolving bulk HiChIP into subpopulation-specific signals.

## When to use

Use when scRNA-seq (E) and scATAC-seq (O) matrices from the same heterogeneous population already carry cell-cluster labels and a bulk data type (e.g., HiChIP loop counts) must be seeded with per-cluster mean profiles before deconvolution.

## When NOT to use

- Single-cell samples are jointly profiled (e.g., sci-CAR-style multi-ome): the article defers incorporating such data types to future work, so cluster-mean seeding across separate samples is not the right setup
- No bulk data type is available to deconvolve — if all data types are already at single-cell resolution, per-cluster means are unnecessary and discard single-cell resolution
- A fully data-driven coupled clustering is required from scratch rather than seeding from pre-assigned cluster labels

## Inputs

- scRNA-seq expression matrix E (rows = genes, columns = cells) with per-cell cluster labels
- scATAC-seq openness matrix O (cells x accessible enhancer regions) with per-cell cluster labels
- Bulk HiChIP enhancer-promoter interaction file (gene index, enhancer index, loop counts), preprocessed with HiC-pro and hichipper and restricted to the scRNA-seq genes and scATAC-seq enhancers
- Gene symbol file and enhancer symbol file
- Number of clusters k (e.g., -k 2)

## Outputs

- Cluster-mean subpopulation profiles P1 (expression) and P2 (accessibility) used as fixed inputs to deconvolution
- Initialized latent loop counts C_ijk (subpopulation-specific HiChIP loop counts)
- DC3 output files: scATAC-result.txt, scRNA-result.txt, cluster-hichip.txt, cluster-specific-peaks-genes-pairs.txt

## How to apply

Load the scRNA-seq expression matrix E (e.g., 146 cells) and the scATAC-seq openness matrix O (e.g., 746 cells) together with per-cell cluster labels, then average cells sharing the same label to obtain the cluster-mean subpopulation profiles P1 and P2. Preprocess HiChIP by restricting enhancer-promoter interactions to the genes in the scRNA-seq data and enhancers in the scATAC-seq data (columns: gene index, enhancer index, HiChIP loop counts). Compute the multinomial proportions rho_ijk = sqrt(O_ik * O_jk) / sum_k sqrt(O_ik * O_jk) from the openness matrix, and allocate each observed bulk loop count C_ij across subpopulations k by multinomial sampling with these proportions to initialize latent loop counts C_ijk. These initialized C_ijk and the fixed cluster-mean profiles then feed the iterative MAP (or simple plug-in p_ijk^3 = alpha * d_ij * lambda_k * p_ik^1 * p_jk^2) estimation to reconstruct subpopulation-specific HiChIP profiles.

## Related tools

- **DC3** (Runs the coupled clustering and deconvolution whose per-cluster mean profiles (P1, P2) and multinomial-initialized latent loop counts C_ijk are computed in this skill; invoked as dc3.py with -k, -E, -O, -hichip and optional lambda1/lambda2 hyper-parameters) — github.com/SUwonglab/DC3
- **HiC-pro / hichipper** (Recommended preprocessors to produce the HiChIP enhancer-promoter interaction file (gene index, enhancer index, loop counts) that supplies the bulk counts C_ij to be allocated by multinomial proportions)

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- Deconvolution quality: mean Pearson Correlation Coefficient (PCC) between true and predicted subpopulation-specific HiChIP vectors across 50 simulation runs; the Poisson implementation is accepted if its mean PCC is comparable to or slightly better than the plug-in baseline (Supplementary Fig. 3 / Supplementary Table 2)
- DC3 run completes producing all four expected output files (scATAC-result.txt, scRNA-result.txt, cluster-hichip.txt, cluster-specific-peaks-genes-pairs.txt)
- Validation against FACS: the inferred subpopulation HiChIP profile should be consistent with HiChIP data on FACS-isolated marker-positive cells
- Multinomial sanity check: for each observed loop count C_ij, the allocations over k sum to C_ij with proportions rho_ijk = sqrt(O_ik * O_jk) / sum_k sqrt(O_ik * O_jk)

## Limitations

- The model-based (Poisson MAP) inference is far more expensive than the simple plug-in method — ~24 h versus ~3 min — despite only slightly better simulated PCC
- Emerging jointly-profiled single-cell data types (e.g., sci-CAR) are not yet incorporated; the cost function would need modification, per the discussion
- HiChIP preprocessing must restrict interactions to the genes in the scRNA-seq data and enhancers in the scATAC-seq data, so the skill depends on compatible feature spaces across the three data types

## Evidence

- [abstract] DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data: "DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data"
- [abstract] The subpopulation-specific profiles of gene expression, chromatin accessibility and enhancer-promoter contact obtained by DC3: "The subpopulation-specific profiles of gene expression, chromatin accessibility and enhancer-promoter contact obtained by DC3 provide a comprehensive characterization of the gene regulatory system"
- [readme] it is of considerable scientific significance to be able to deconvolve the bulk sample HiChIP signal (i.e., bulk sample loop counts) into subpopulation-specific HiChIP signals: "Therefore, it is of considerable scientific significance to be able to deconvolve the bulk sample HiChIP signal (i.e., bulk sample loop counts) into subpopulation-specific HiChIP signals"
- [readme] The first column indicates the index of gene, the second column indicates the index of enhancer, the third column indicates the HiChIP loop counts.: "The first column indicates the index of gene, the second column indicates the index of enhancer, the third column indicates the HiChIP loop counts."
- [discussion] the accessibility, expression and loop profiles inferred by DC3 can serve as a foundation for further analyses of the regulatory systems: "the accessibility, expression and loop profiles inferred by DC3 can serve as a foundation for further analyses of the regulatory systems, such as the extraction of core subnetworks"
