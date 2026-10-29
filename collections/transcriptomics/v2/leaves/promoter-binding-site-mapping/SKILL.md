---
name: promoter-binding-site-mapping
description: Use when you need promoter-restricted TF motif activity from scRNA-seq
  data — e.g. reproducing ISMARA-like motif dynamics or correlating a TF's motif activity
  with its own mRNA across a cell cycle trajectory.
when_to_use_negative:
- The dataset lacks unspliced read counts (no scVelo 'Mu' layer), since the linear
  model is fit on unspliced counts
- No genome-mapped binding-site annotations (GFF) are available for the organism,
  so promoter windows cannot be defined
- Cells are not ordered along a dynamic process (e.g. no cell_cycle_theta), so motif-activity
  dynamics cannot be correlated with expression
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3170
tools:
- name: DeepCycle
  role: Provides the continuous cell cycle trajectory (adata.obs['cell_cycle_theta'])
    used to order cells for motif-activity dynamics
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Supplies the imputed unspliced ('Mu') layer used to build the promoter-by-motif
    count matrix
- name: numpy
  role: Numerical implementation of the ridge-regularized ISMARA-like linear model
    on unspliced counts
- name: scipy
  role: Exact test of no-correlation and Pearson correlation between motif activity
    and mRNA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:c56e596a70021cfba8849fff679d543623e2b05775499c3b23edbf50013f31e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - DeepCycle
  - Python
  - numpy
  - scVelo
  - scipy
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# promoter-binding-site-mapping

## Summary

Maps transcription factor binding motifs to promoters by restricting motif-predicted binding sites (e.g. Swiss Regulon Motevo GFF annotations) to TSS +/- 1 kb windows, then quantifying motif activity from scRNA-seq unspliced counts so promoter-level motif activity can be correlated with TF mRNA dynamics (e.g. Ybx1 across the cell cycle).

## When to use

Use when you need promoter-restricted TF motif activity from scRNA-seq data — e.g. reproducing ISMARA-like motif dynamics or correlating a TF's motif activity with its own mRNA across a cell cycle trajectory.

## When NOT to use

- The dataset lacks unspliced read counts (no scVelo 'Mu' layer), since the linear model is fit on unspliced counts
- No genome-mapped binding-site annotations (GFF) are available for the organism, so promoter windows cannot be defined
- Cells are not ordered along a dynamic process (e.g. no cell_cycle_theta), so motif-activity dynamics cannot be correlated with expression

## Inputs

- Swiss Regulon mm10/hg19 GFF files with Motevo-predicted TF binding sites
- scRNA-seq dataset with scVelo moments (unspliced 'Mu' layer), e.g. ductal cell progenitor h5ad
- Cell cycle phase/theta annotations (e.g. adata.obs['cell_cycle_theta'] from DeepCycle)

## Outputs

- Promoter-by-motif count matrix restricted to TSS +/- 1 kb promoter windows
- Ridge-regularized motif activity estimates per cell (ISMARA-like linear model)
- Pearson correlation between TF motif activity and TF mRNA (e.g. Ybx1 r = 0.91, p = 4e-19)

## How to apply

Download the Swiss Regulon mm10/hg19 GFF files containing Motevo-predicted transcription factor binding sites and restrict annotations to promoter windows spanning the TSS +/- 1 kb. Load the scRNA-seq dataset (e.g. ductal cell progenitors) and extract unspliced read counts from the scVelo 'Mu' layer to build the promoter-by-motif count matrix. Fit an ISMARA-style linear model on the unspliced counts using ridge regularization, tuning the ridge penalty via 10-fold cross-validation over cells. Finally compute the Pearson correlation between a TF's motif activity and its mRNA abundance across cells (e.g. Ybx1, expected r = 0.91, p = 4e-19, with both constantly increasing from G1 to M).

## Related tools

- **DeepCycle** (Provides the continuous cell cycle trajectory (adata.obs['cell_cycle_theta']) used to order cells for motif-activity dynamics) — https://github.com/andreariba/DeepCycle
- **scVelo** (Supplies the imputed unspliced ('Mu') layer used to build the promoter-by-motif count matrix)
- **numpy** (Numerical implementation of the ridge-regularized ISMARA-like linear model on unspliced counts)
- **scipy** (Exact test of no-correlation and Pearson correlation between motif activity and mRNA)

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- Reproduced Ybx1 correlation matches the reported r = 0.91 (p = 4e-19 exact test) within tolerance (Fig. 4B)
- Ybx1 mRNA and motif activity both monotonically increase from G1 to M in ductal cells
- Motif activity table is complete for all promoters in the TSS +/- 1 kb restricted GFF set
- Ridge penalty is selected by 10-fold cross-validation over cells rather than fixed arbitrarily

## Limitations

- Motevo-predicted binding sites are precomputed promoter annotations; motif calls depend on the quality of the Swiss Regulon GFF and promoter window choice
- If cycling cells or cycling genes are absent (e.g. insufficient sequencing depth), the downstream correlation with cell cycle phases will not be reliable
- Results are organism/assembly-specific (mm10/hg19 GFFs); other genomes require corresponding binding-site annotations

## Evidence

- [results] In ductal cells, Ybx1 mRNA level and its motif activity show a very high correlation (r = 0.91, p = 4e-19 exact test of no-correlation), with both constantly increasing from G1 to M.: [evidence span withheld — anchor status: not_found]
- [supplementary] Swiss Regulon GFF restriction to promoter windows: [evidence span withheld — anchor status: not_found]
- [supplementary] scVelo Mu layer as model input: "the layers 'Ms' and 'Mu' from scVelo"
- [readme] Preprocessing requirement for unspliced/spliced moments: "The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo (https://scvelo.readthedocs.io/) to generate imputed unspliced (Mu) and spliced (Ms) expressions."
- [readme] Reliability caveat at low sequencing depth: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
