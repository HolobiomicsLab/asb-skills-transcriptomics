---
name: soybean-fungal-stress-transcriptomics
description: Use when you have a filtered soybean RNA-seq expression matrix spanning
  multiple fungal-pathogen stress conditions and need to build an unsigned coexpression
  network whose gene-module structure supports GWAS-guided candidate resistance gene
  prioritization.
when_to_use_negative:
- When the goal is a signed coexpression network that preserves correlation sign,
  since this skill explicitly assembles an unsigned network on absolute Pearson correlations
- When the input is GWAS SNP summary data without RNA-seq expression; this skill only
  covers the transcriptomic/coexpression half of the paper's integration strategy
- When the expression data come from a single condition or non-soybean species, since
  the network was built specifically from five fungal-pathogen stress conditions in
  soybean
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BioNERO
  role: Quantile normalization and expression adjustment of the filtered RNA-seq matrix,
    then construction of the unsigned coexpression network using Pearson's r
- name: SoyFungiGCN
  role: R/Shiny package that packages the resulting coexpression network for interactive
    exploration of soybean-pathogenic fungi interactions
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: remotes
  role: Installs SoyFungiGCN from GitHub so the network and app can be loaded locally
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:a95ab347e672b284484dbc6a7e9700ca30b3e5916d9931a1fd31c1264a6693cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - BioNERO
  - R
  - SoyFungiGCN
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# soybean-fungal-stress-transcriptomics

## Summary

Reconstructs an unsigned soybean gene coexpression network from RNA-seq data of tissues infected with five phytopathogenic fungi (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi), using BioNERO preprocessing and the SoyFungiGCN R package to enable downstream candidate resistance gene mining.

## When to use

Use when you have a filtered soybean RNA-seq expression matrix spanning multiple fungal-pathogen stress conditions and need to build an unsigned coexpression network whose gene-module structure supports GWAS-guided candidate resistance gene prioritization.

## When NOT to use

- When the goal is a signed coexpression network that preserves correlation sign, since this skill explicitly assembles an unsigned network on absolute Pearson correlations
- When the input is GWAS SNP summary data without RNA-seq expression; this skill only covers the transcriptomic/coexpression half of the paper's integration strategy
- When the expression data come from a single condition or non-soybean species, since the network was built specifically from five fungal-pathogen stress conditions in soybean

## Inputs

- Filtered soybean RNA-seq expression matrix with samples from five fungal-pathogen stress conditions (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi)
- SoyFungiGCN R package data objects (genes_modules, hubs, mod_enrich, plotdata, scaled_degree) for validation

## Outputs

- Quantile-normalized, adjusted expression matrix
- Unsigned coexpression network object stored as coexpression_network.rds
- Gene-to-module assignments consistent with the genes_modules dataset in SoyFungiGCN
- Interactive exploration surface via the SoyFungiGCN Shiny app and web application (https://soyfungigcn.venanciogroup.uenf.br/)

## How to apply

Load the filtered soybean RNA-seq expression matrix containing samples from the five fungal-pathogen stress conditions and apply quantile normalization plus expression adjustment with BioNERO so that sample-level distributions are comparable. Compute pairwise gene-gene correlations on the adjusted expression matrix using Pearson's r as the correlation measure. Assemble the network as unsigned (absolute correlation values) so both positively and negatively correlated genes are retained as edges, and store the result as coexpression_network.rds. The resulting network is explored via the SoyFungiGCN R/Shiny package (installed with remotes from GitHub and launched with run_app()), and module assignments must stay consistent with the genes_modules data released in that package.

## Related tools

- **BioNERO** (Quantile normalization and expression adjustment of the filtered RNA-seq matrix, then construction of the unsigned coexpression network using Pearson's r)
- **SoyFungiGCN** (R/Shiny package that packages the resulting coexpression network for interactive exploration of soybean-pathogenic fungi interactions) — https://github.com/almeidasilvaf/SoyFungiGCN
- **remotes** (Installs SoyFungiGCN from GitHub so the network and app can be loaded locally) — https://github.com/almeidasilvaf/SoyFungiGCN

## Examples

```
remotes::install_github("almeidasilvaf/SoyFungiGCN"); library(SoyFungiGCN); run_app()
```

## Evaluation signals

- Verify the network was built on the quantile-normalized, adjusted expression data rather than raw counts
- Confirm the network is declared unsigned and that edge weights correspond to Pearson correlation values
- Check that downstream gene-module assignments are consistent with the genes_modules data released in the SoyFungiGCN package
- Confirm the network object serializes as coexpression_network.rds and can be loaded into the SoyFungiGCN Shiny app via run_app()
- Confirm samples from all five fungal-pathogen stress conditions are represented in the adjusted expression matrix used for network construction

## Limitations

- The SoyFungiGCN package is marked as experimental lifecycle (badge in README) and no changelog or package version is stated in the article
- Supplementary figures and tables referenced by the article are not included/accessible in the provided text, limiting method reproducibility detail
- The public web resource is specific to soybean-pathogenic fungi interactions and its URL is not given in the discussion section itself

## Evidence

- [readme] README description of the packaged network: "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
- [methods] Package installation path: "You can install the released version of SoyFungiGCN from GitHub with"
- [discussion] Network dissemination as web resource and R package: "The coexpression network generated here was also made available in a web resource and R package to help in future studies on soybean-pathogenic fungi interactions"
- [intro] Public web application availability: "publicly available web application (https://soyfungigcn.venanciogroup.uenf.br/)"
- [abstract] Candidate genes from the integrated pipeline: "We identified 188, 56, 11, 8, and 3 high-confidence candidates for resistance to F. virguliforme, F. graminearum, C. gregata, M. phaseolina and P. pachyrhizi"
