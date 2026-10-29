---
name: poisson-model-hichip-deconvolution
description: Use when bulk HiChIP enhancer-promoter loop counts must be deconvolved
  into subpopulation-specific profiles using scRNA-seq and scATAC-seq profiles from
  the same population, and a principled model-based alternative to the plug-in baseline
  is acceptable despite ~24 h runtime.
when_to_use_negative:
- 'When runtime matters (e.g. quick exploratory analysis): the Poisson MAP inference
  takes ~24 h versus ~3 min for the simple plug-in method, with only slightly better
  PCC'
- When scRNA-seq and scATAC-seq profiles cannot be treated as known (no reliable cluster
  labels / subpopulation profiles P1, P2 available)
- When the goal is not HiChIP deconvolution but plain coupled clustering of single
  cells
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DC3
  role: Joint analysis framework whose DC3 context hosts the Poisson model-based HiChIP
    deconvolution; alternative to dcHiChIP's simple plug-in deconvolution
  repo: github.com/SUwonglab/DC3
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:33ef32a65e8825c696c092d45d8e360a6f76e2303edfedd23126879b6846c7b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# poisson-model-hichip-deconvolution

## Summary

A Poisson model-based MAP inference approach for deconvolving bulk H3K27ac HiChIP loop counts into subpopulation-specific profiles within DC3, treating single-cell expression/accessibility profiles as known and iteratively estimating latent multinomial loop-count variables. It offers better interpretation and slightly better simulation performance than the simple plug-in method, at much higher time cost (24 h vs 3 min).

## When to use

Use when bulk HiChIP enhancer-promoter loop counts must be deconvolved into subpopulation-specific profiles using scRNA-seq and scATAC-seq profiles from the same population, and a principled model-based alternative to the plug-in baseline is acceptable despite ~24 h runtime.

## When NOT to use

- When runtime matters (e.g. quick exploratory analysis): the Poisson MAP inference takes ~24 h versus ~3 min for the simple plug-in method, with only slightly better PCC
- When scRNA-seq and scATAC-seq profiles cannot be treated as known (no reliable cluster labels / subpopulation profiles P1, P2 available)
- When the goal is not HiChIP deconvolution but plain coupled clustering of single cells

## Inputs

- scRNA-seq expression matrix E (146 cells, rows = genes, columns = cells) with cluster labels
- scATAC-seq openness matrix O (746 cells) with cluster labels
- Bulk H3K27ac HiChIP enhancer-promoter interaction file (gene index, enhancer index, loop counts), e.g. down-sampled from GM12878/K562
- Cluster-mean subpopulation profiles P1 and P2 computed by averaging cells with the same cluster label
- Gene symbol and enhancer symbol files

## Outputs

- Estimated latent loop counts C_ijk and reconstructed subpopulation-specific HiChIP profiles
- Mean Pearson Correlation Coefficient (PCC) between true and predicted subpopulation-specific HiChIP vectors over 50 runs
- Runtime comparison against the simple plug-in baseline

## How to apply

Load the in-silico mixture matrices (scRNA-seq E with 146 cells, scATAC-seq O with 746 cells, and bulk HiChIP down-sampled from GM12878/K562) and compute cluster-mean subpopulation profiles P1 and P2 by averaging cells sharing a cluster label. Compute the multinomial proportions rho_ijk = sqrt(O_ik * O_jk) / sum_k sqrt(O_ik * O_jk) and initialize latent loop counts C_ijk by multinomial allocation of each observed C_ij with these proportions. Iteratively compute the MAP estimate of the latent multinomial variable M_ij conditional on all other latent variables, using the Poisson gene-expression likelihood E_ik ~ Poisson(beta * sum_q C_iqk), until convergence. Reconstruct subpopulation-specific HiChIP profiles from the estimated C_ijk and evaluate deconvolution as the mean Pearson Correlation Coefficient (PCC) between true and predicted vectors over 50 runs, benchmarking against the simple plug-in baseline (p_ijk^3 = alpha * d_ij * lambda_k * p_ik^1 * p_jk^2). Accept the implementation if its mean PCC is comparable to or slightly better than the plug-in baseline, consistent with Supplementary Fig. 3 / Supplementary Table 2.

## Related tools

- **DC3** (Joint analysis framework whose DC3 context hosts the Poisson model-based HiChIP deconvolution; alternative to dcHiChIP's simple plug-in deconvolution) — github.com/SUwonglab/DC3

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- Mean PCC between true and predicted subpopulation-specific HiChIP vectors over 50 simulation runs is comparable to or slightly better than the plug-in baseline
- Comparison consistent with reported Supplementary Fig. 3 / Supplementary Table 2 results
- Iterative MAP update of M_ij converges under the Poisson likelihood E_ik ~ Poisson(beta * sum_q C_iqk)
- Initialization via multinomial allocation with proportions rho_ijk = sqrt(O_ik * O_jk) / sum_k sqrt(O_ik * O_jk) reproduces the described setup
- Reconstructed subpopulation loop counts sum consistently with observed bulk C_ij allocations

## Limitations

- Much higher time cost than the simple method (24 h vs 3 min) for only slightly better performance in simulations
- Single-cell data type profiles (P1, P2) are treated as known, so accuracy depends on the upstream coupled clustering quality
- The model-based method of Lin et al. was not incorporated into DC3's inference in the published version; deferred to future work

## Evidence

- [supplementary] Poisson inference treats sc profiles as known: [evidence span withheld — anchor status: not_found]
- [supplementary] runtime and performance tradeoff: [evidence span withheld — anchor status: not_found]
- [readme] dcHiChIP purpose statement: "Here, we introduce dcHiChIP (for deconvolution of HiChIP) for the solution of this problem."
- [supplementary] iterative MAP estimation with Poisson likelihood: [evidence span withheld — anchor status: not_found]
- [supplementary] validation criterion: [evidence span withheld — anchor status: not_found]
