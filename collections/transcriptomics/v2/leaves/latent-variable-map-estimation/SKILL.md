---
name: latent-variable-map-estimation
description: Use when bulk HiChIP enhancer-promoter loop counts must be deconvolved
  into subpopulation-specific profiles and a model-based alternative to DC3's simple
  plug-in method is desired, with single-cell RNA/ATAC profiles available as known
  inputs.
when_to_use_negative:
- When runtime is constrained — this method costs ~24 h versus ~3 min for the simple
  plug-in method
- When single-cell data type profiles cannot be treated as known (this inference assumes
  scRNA-seq E and scATAC-seq O profiles are available and fixed)
- When no HiChIP data linking enhancers to target genes exists and only coupled clustering
  of single cells is needed
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: DC3
  role: Framework providing coupled clustering and the deconvolution context; the
    Poisson model-based MAP inference is an alternative model-based approach within
    DC3 for deconvolving bulk HiChIP into subpopulation-specific profiles
  repo: github.com/SUwonglab/DC3
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:4e69d3e6895cf03a8384f5544fbc1010238fe03784c15b32f7f833722f481780
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# latent-variable-map-estimation

## Summary

A Poisson model-based iterative MAP inference approach within DC3 that deconvolves bulk H3K27ac HiChIP loop counts into subpopulation-specific profiles by treating single-cell scRNA-seq and scATAC-seq profiles as known and estimating the latent loop-count variable M_ij conditional on all other latent variables until convergence.

## When to use

Use when bulk HiChIP enhancer-promoter loop counts must be deconvolved into subpopulation-specific profiles and a model-based alternative to DC3's simple plug-in method is desired, with single-cell RNA/ATAC profiles available as known inputs.

## When NOT to use

- When runtime is constrained — this method costs ~24 h versus ~3 min for the simple plug-in method
- When single-cell data type profiles cannot be treated as known (this inference assumes scRNA-seq E and scATAC-seq O profiles are available and fixed)
- When no HiChIP data linking enhancers to target genes exists and only coupled clustering of single cells is needed

## Inputs

- scRNA-seq expression matrix E (genes x cells, 146 cells)
- scATAC-seq openness matrix O (enhancers x cells, 746 cells)
- Bulk H3K27ac HiChIP enhancer-promoter interaction file with loop counts C_ij (gene index, enhancer index, count)
- Cluster labels for single cells used to compute cluster-mean profiles P1 and P2

## Outputs

- Estimated latent loop counts C_ijk per subpopulation k
- Subpopulation-specific HiChIP profiles
- Mean Pearson Correlation Coefficient (PCC) between true and predicted subpopulation-specific HiChIP vectors over 50 runs
- Runtime comparison against the simple plug-in baseline

## How to apply

Load in-silico mixture matrices (scRNA-seq E with 146 cells, scATAC-seq O with 746 cells, bulk HiChIP down-sampled from GM12878/K562) and compute cluster-mean subpopulation profiles P1 and P2 by averaging cells sharing a cluster label. Compute multinomial proportions rho_ijk = sqrt(O_ik * O_jk) / sum_k sqrt(O_ik * O_jk) and initialize latent loop counts C_ijk by multinomial allocation of each observed C_ij under rho_ijk. Iteratively compute the MAP estimate of the latent multinomial variable M_ij conditional on all other latent variables, using the Poisson gene-expression likelihood E_ik ~ Poisson(beta * sum_q C_iqk), until convergence, then reconstruct subpopulation-specific HiChIP profiles from the estimated C_ijk. Evaluate as mean Pearson Correlation Coefficient (PCC) between true and predicted subpopulation-specific HiChIP vectors over 50 runs against the simple plug-in baseline p_ijk^3 = alpha * d_ij * lambda_k * p_ik^1 * p_jk^2; expect comparable or slightly better PCC but a much higher runtime (24 h vs 3 min).

## Related tools

- **DC3** (Framework providing coupled clustering and the deconvolution context; the Poisson model-based MAP inference is an alternative model-based approach within DC3 for deconvolving bulk HiChIP into subpopulation-specific profiles) — github.com/SUwonglab/DC3

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- Mean PCC between true and predicted subpopulation-specific HiChIP vectors over 50 runs should be comparable to or slightly better than the simple plug-in baseline, consistent with Supplementary Fig. 3 / Supplementary Table 2
- Iterative MAP update of M_ij must converge before reconstructing subpopulation-specific profiles from C_ijk
- Initial C_ijk must be allocated via multinomial sampling with proportions rho_ijk = sqrt(O_ik * O_jk) / sum_k sqrt(O_ik * O_jk)
- Runtime should be recorded and is expected to be ~24 h, versus ~3 min for the plug-in method; large deviations suggest implementation issues
- Reconstructed profiles should yield sensible subpopulation-specific loop counts consistent with the Poisson likelihood E_ik ~ Poisson(beta * sum_q C_iqk)

## Limitations

- Much higher time cost than the simple method (24 h vs 3 min)
- Only slightly better performance in simulations than the plug-in baseline, so the added complexity may not always be justified
- Treats single-cell data type profiles as known, so errors in upstream scRNA-seq/scATAC-seq clustering propagate to the inference
- Jointly-profiled single-cell data types (e.g., sci-CAR) are not yet incorporated; deferred to future work
- The model-based method of Lin et al. is not yet incorporated into DC3's inference; deferred to future work

## Evidence

- [supplementary] Poisson model treats single-cell profiles as known; better interpretation, slightly better performance, 24 h vs 3 min runtime: [evidence span withheld — anchor status: not_found]
- [abstract] DC3 as joint-analysis method for bulk and single-cell data: "we introduce DC3 (De-Convolution and Coupled-Clustering) as a method for the joint analysis of various bulk and single-cell data"
- [readme] dcHiChIP deconvolves bulk HiChIP loop counts into subpopulation-specific signals: "it is of considerable scientific significance to be able to deconvolve the bulk sample HiChIP signal (i.e., bulk sample loop counts) into subpopulation-specific HiChIP signals"
- [discussion] Model-based subpopulation inference method deferred to future work: "and incorporate the model-based method into the inference"
