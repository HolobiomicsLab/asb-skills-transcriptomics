---
name: expression-data-adjustment-preprocessing
description: Use when filtered RNA-seq expression matrices from multiple infection-condition
  samples must be distribution-adjusted (quantile normalization) before computing
  gene-gene correlations or building coexpression modules.
when_to_use_negative:
- The expression data are already quantile-normalized and adjusted, so re-running
  normalization would distort distributions
- A signed network is required where positive and negative correlations must be distinguished
  rather than collapsed via absolute correlation values
- The goal is candidate-gene prioritization from GWAS resistance alleles without any
  expression preprocessing step upstream
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BioNERO
  role: R package used for quantile normalization and expression adjustment of the
    filtered data, and for assembling the unsigned coexpression network with Pearson's
    r
- name: R
  role: Execution environment in which BioNERO preprocessing and network construction
    are run and coexpression_network.rds is written
- name: SoyFungiGCN
  role: R/Shiny package that distributes the resulting coexpression network and the
    genes_modules data used to validate module assignments
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:0be1c1e333bb5e73991e4755c80c9231719b345fea10f658d4588a97e79fea6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - BioNERO
  - R
  - SoyFungiGCN
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# expression-data-adjustment-preprocessing

## Summary

Preprocesses filtered soybean RNA-seq expression data from five fungal-pathogen stress conditions using BioNERO's expression preprocessing (quantile normalization and adjustment) so that sample-level distributions are comparable before coexpression network construction.

## When to use

Use when filtered RNA-seq expression matrices from multiple infection-condition samples must be distribution-adjusted (quantile normalization) before computing gene-gene correlations or building coexpression modules.

## When NOT to use

- The expression data are already quantile-normalized and adjusted, so re-running normalization would distort distributions
- A signed network is required where positive and negative correlations must be distinguished rather than collapsed via absolute correlation values
- The goal is candidate-gene prioritization from GWAS resistance alleles without any expression preprocessing step upstream

## Inputs

- Filtered soybean RNA-seq expression matrix containing samples from the five fungal-pathogen stress conditions
- Gene-gene adjusted expression matrix (output of BioNERO preprocessing) used for Pearson correlation computation

## Outputs

- Quantile-normalized, adjusted expression matrix
- Unsigned gene coexpression network object stored as coexpression_network.rds
- Gene-module assignments consistent with the genes_modules data in the SoyFungiGCN package

## How to apply

Load the filtered soybean RNA-seq expression matrix containing samples from the five fungal-pathogen stress conditions, then apply quantile normalization and expression adjustment with BioNERO's expression preprocessing so sample-level distributions are comparable. Next, compute pairwise gene-gene correlations on the adjusted expression matrix using Pearson's r as the correlation method. Assemble the unsigned coexpression network in BioNERO, using absolute correlation values so both positively and negatively correlated gene pairs are retained as edges. Finally, store the resulting network object as coexpression_network.rds for downstream candidate gene mining and module assignment.

## Related tools

- **BioNERO** (R package used for quantile normalization and expression adjustment of the filtered data, and for assembling the unsigned coexpression network with Pearson's r)
- **R** (Execution environment in which BioNERO preprocessing and network construction are run and coexpression_network.rds is written)
- **SoyFungiGCN** (R/Shiny package that distributes the resulting coexpression network and the genes_modules data used to validate module assignments) — https://github.com/almeidasilvaf/SoyFungiGCN

## Examples

```
remotes::install_github("almeidasilvaf/SoyFungiGCN"); adj_exp <- BioNERO::exp_preprocess(filt_exp); coexpression_network <- exp2gcn(adj_exp, net_type = "unsigned", cor_method = "pearson")
```

## Evaluation signals

- Verify the network was built on quantile-normalized, adjusted expression data rather than the raw filtered matrix
- Verify the network is declared unsigned, with edge weights corresponding to Pearson correlations (absolute values retaining both positive and negative correlations)
- Verify downstream gene-module assignments are consistent with the genes_modules data released in the SoyFungiGCN package
- Verify samples span all five fungal-pathogen stress conditions after adjustment and that sample-level distributions are comparable
- Verify the output object coexpression_network.rds loads in R and supports the coexpression network distributed via SoyFungiGCN

## Limitations

- The normalization scheme (quantile normalization via BioNERO) is specific to making sample-level distributions comparable across the five fungal-pathogen stress conditions; other experimental designs may require different adjustment
- Using absolute Pearson correlation (unsigned network) loses correlation sign information, so positive vs negative regulation cannot be recovered from the final network alone
- Validation depends on the released genes_modules data in SoyFungiGCN; no package version or changelog is stated in the article or README

## Evidence

- [other] Quantile normalization with BioNERO before unsigned network construction: [evidence span withheld — anchor status: not_found]
- [other] Absolute correlations retain both positive and negative edges: [evidence span withheld — anchor status: not_found]
- [other] Consistency check against SoyFungiGCN genes_modules: [evidence span withheld — anchor status: not_found]
- [readme] SoyFungiGCN distributes the coexpression network via R/Shiny: "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
