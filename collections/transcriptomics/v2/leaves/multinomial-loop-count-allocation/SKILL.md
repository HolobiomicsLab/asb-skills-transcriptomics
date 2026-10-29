---
name: multinomial-loop-count-allocation
description: Use when you have bulk HiChIP loop counts (gene-enhancer pairs) plus
  scRNA-seq (E) and scATAC-seq (O) matrices with cluster labels from the same heterogeneous
  population, and you want subpopulation-specific HiChIP profiles with better interpretation
  than the plug-in baseline.
when_to_use_negative:
- When wall-clock runtime matters (~3 min) or a first-pass result suffices — the simple
  plug-in baseline achieves comparable PCC at a fraction of the ~24 h cost
- When the HiChIP profile for each subpopulation can be measured directly, e.g. on
  FACS-isolated marker-positive cells
- When scRNA-seq and scATAC-seq samples do not come from the same cell population,
  since cluster-mean profiles P1/P2 from these samples anchor the multinomial proportions
edam_operation: http://edamontology.org/operation_3659
tools:
- name: DC3
  role: Reference implementation and plug-in baseline framework; dcHiChIP in the DC3
    repo preprocesses scRNA-seq E, scATAC-seq O, and HiChIP interaction files that
    this multinomial/Poisson allocation scheme consumes
  repo: github.com/SUwonglab/DC3
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:528a7294ce9448f3a7c245398f046a9e1f4c9e5dd9f5d6bfc6f395c5dddbc4e3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# multinomial-loop-count-allocation

## Summary

Model-based inference that decomposes bulk H3K27ac HiChIP loop counts into subpopulation-specific loop counts by initializing latent counts C_ijk via multinomial allocation and iteratively refining them with MAP estimation under a Poisson gene-expression likelihood, as an alternative to DC3's simple plug-in method.

## When to use

Use when you have bulk HiChIP loop counts (gene-enhancer pairs) plus scRNA-seq (E) and scATAC-seq (O) matrices with cluster labels from the same heterogeneous population, and you want subpopulation-specific HiChIP profiles with better interpretation than the plug-in baseline.

## When NOT to use

- When wall-clock runtime matters (~3 min) or a first-pass result suffices — the simple plug-in baseline achieves comparable PCC at a fraction of the ~24 h cost
- When the HiChIP profile for each subpopulation can be measured directly, e.g. on FACS-isolated marker-positive cells
- When scRNA-seq and scATAC-seq samples do not come from the same cell population, since cluster-mean profiles P1/P2 from these samples anchor the multinomial proportions

## Inputs

- scRNA-seq expression matrix E (rows = genes, columns = cells, e.g. 146 cells) with per-cell cluster labels
- scATAC-seq openness matrix O (rows = enhancers/peaks, columns = cells, e.g. 746 cells) with per-cell cluster labels
- Bulk H3K27ac HiChIP enhancer-promoter interaction file (gene index, enhancer index, loop counts), e.g. down-sampled GM12878/K562

## Outputs

- Estimated latent loop counts C_ijk per subpopulation k
- Reconstructed subpopulation-specific HiChIP profiles
- Mean Pearson Correlation Coefficient (PCC) between true and predicted subpopulation HiChIP vectors over 50 runs

## How to apply

Load the in-silico mixture matrices (scRNA-seq E, scATAC-seq O, bulk HiChIP restricted to genes in E and enhancers in O) and compute cluster-mean profiles P1 and P2 by averaging cells sharing a cluster label. Compute multinomial proportions rho_ijk = sqrt(O_ik * O_jk) / sum_k sqrt(O_ik * O_jk) and initialize latent loop counts C_ijk by multinomial allocation of each observed bulk count C_ij according to rho_ijk. Iteratively compute the MAP estimate of each latent multinomial M_ij conditional on all other latent variables, using the Poisson likelihood E_ik ~ Poisson(beta * sum_q C_iqk), until convergence. Reconstruct subpopulation-specific HiChIP profiles from the estimated C_ijk and score deconvolution as the mean Pearson Correlation Coefficient (PCC) between true and predicted subpopulation-specific HiChIP vectors over 50 runs, benchmarking against the plug-in baseline p_ijk^3 = alpha * d_ij * lambda_k * p_ik^1 * p_jk^2. The Poisson treatment treats single-cell profiles as known and yields slightly better simulation performance but at far higher cost (~24 h vs ~3 min).

## Related tools

- **DC3** (Reference implementation and plug-in baseline framework; dcHiChIP in the DC3 repo preprocesses scRNA-seq E, scATAC-seq O, and HiChIP interaction files that this multinomial/Poisson allocation scheme consumes) — github.com/SUwonglab/DC3

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- Mean PCC between true and predicted subpopulation-specific HiChIP vectors is comparable to or slightly better than the plug-in baseline (consistent with Supplementary Fig. 3 / Supplementary Table 2)
- Iterative MAP estimates of M_ij converge under the Poisson likelihood E_ik ~ Poisson(beta * sum_q C_iqk)
- Estimated C_ijk reconstruct valid non-negative subpopulation HiChIP loop counts over all gene-enhancer pairs
- Multinomial proportions rho_ijk sum to 1 over k for every gene-enhancer pair (i,j)
- Reported runtime is on the order of ~24 h, distinguishing it from the ~3 min plug-in method

## Limitations

- Much higher time cost than the simple plug-in method (24 h vs 3 min) for only slightly better simulation performance
- Treats single-cell data type profiles as known, so errors in scRNA-seq/scATAC-seq clustering propagate into the allocation
- Requires the three data types from separate samples of the same population; jointly profiled single-cell data (e.g. sci-CAR) and incorporation of the Lin et al. model-based method were deferred to future work

## Evidence

- [methods] Poisson model-based allocation workflow: [evidence span withheld — anchor status: not_found]
- [methods] MAP with Poisson likelihood: [evidence span withheld — anchor status: not_found]
- [results] Performance trade-off: [evidence span withheld — anchor status: not_found]
- [readme] Motivation for deconvolving bulk HiChIP: "it is of considerable scientific significance to be able to deconvolve the bulk sample HiChIP signal (i.e., bulk sample loop counts)"
- [readme] HiChIP input restriction: "please restrict HiChIP enhancer-promoter interactions in the genes in scRNA-seq data and enhancers in scATAC-seq data"
