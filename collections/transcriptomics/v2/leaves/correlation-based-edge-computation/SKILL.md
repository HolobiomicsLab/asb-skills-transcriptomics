---
name: correlation-based-edge-computation
description: Use when a filtered RNA-seq expression matrix (e.g., soybean samples
  across five fungal-pathogen stress conditions) must be converted into a weighted
  coexpression network whose edges are correlation-based, as a prerequisite for candidate
  gene or module mining.
when_to_use_negative:
- The input is already a coexpression network or edge list rather than a gene-by-sample
  expression matrix.
- The analysis requires a signed network that separates positive from negative correlations
  — this skill explicitly uses absolute correlations in an unsigned network.
- Data are GWAS SNP summary statistics or germplasm resistance scores rather than
  RNA-seq expression values; those feed into downstream prioritization, not edge computation.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: BioNERO
  role: Performs expression preprocessing (quantile normalization/adjustment) and
    computes pairwise Pearson correlations to assemble the unsigned coexpression network
- name: R
  role: Language in which the coexpression network is constructed and packaged for
    reuse
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: SoyFungiGCN
  role: R/Shiny package that distributes and exposes the resulting coexpression network
    and genes_modules data for exploration
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:28e45e181f7608f59fe93ed0ef9b88ba1e4f1e04d74d22695968bc8a1089beea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - BioNERO
  - R
  - SoyFungiGCN
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# correlation-based-edge-computation

## Summary

Builds an unsigned gene coexpression network from soybean fungal-infection RNA-seq expression data by computing pairwise gene-gene Pearson correlations with BioNERO on quantile-normalized, adjusted expression matrices, storing the result as coexpression_network.rds.

## When to use

Use when a filtered RNA-seq expression matrix (e.g., soybean samples across five fungal-pathogen stress conditions) must be converted into a weighted coexpression network whose edges are correlation-based, as a prerequisite for candidate gene or module mining.

## When NOT to use

- The input is already a coexpression network or edge list rather than a gene-by-sample expression matrix.
- The analysis requires a signed network that separates positive from negative correlations — this skill explicitly uses absolute correlations in an unsigned network.
- Data are GWAS SNP summary statistics or germplasm resistance scores rather than RNA-seq expression values; those feed into downstream prioritization, not edge computation.

## Inputs

- filtered soybean RNA-seq expression matrix with samples from the five fungal-pathogen stress conditions (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi)
- quantile-normalized, adjusted expression matrix produced by BioNERO expression preprocessing

## Outputs

- unsigned coexpression network with edge weights corresponding to Pearson correlation values (coexpression_network.rds)
- gene-module assignments consistent with the genes_modules data released in the SoyFungiGCN package

## How to apply

Load the filtered soybean RNA-seq expression matrix containing samples from the five fungal-pathogen stress conditions, then apply quantile normalization and expression adjustment with BioNERO's expression preprocessing so sample-level distributions are comparable. Compute pairwise gene-gene correlations on the adjusted expression matrix using Pearson's r as the correlation method. Assemble the network as unsigned in BioNERO, taking absolute correlation values so that both positively and negatively correlated gene pairs are retained as edges. Finally, store the resulting network object as coexpression_network.rds so it can be explored downstream via the SoyFungiGCN R/Shiny package and web application.

## Related tools

- **BioNERO** (Performs expression preprocessing (quantile normalization/adjustment) and computes pairwise Pearson correlations to assemble the unsigned coexpression network)
- **R** (Language in which the coexpression network is constructed and packaged for reuse) — https://github.com/almeidasilvaf/SoyFungiGCN
- **SoyFungiGCN** (R/Shiny package that distributes and exposes the resulting coexpression network and genes_modules data for exploration) — https://github.com/almeidasilvaf/SoyFungiGCN

## Evaluation signals

- Confirm the network was built on the quantile-normalized, adjusted expression data, not on raw counts or TPM.
- Verify the network is declared unsigned and edge weights correspond to absolute Pearson correlation values.
- Check that downstream gene-module assignments are consistent with the genes_modules data released in the SoyFungiGCN package.
- Confirm the network object is persisted as coexpression_network.rds and is loadable in the SoyFungiGCN Shiny app (run_app()).
- Verify genes span the five fungal-pathogen stress conditions used to build the network.

## Limitations

- The package is labeled experimental (lifecycle badge in the README), so the network resource may change without a changelog.
- The article section notes no changelog is available for the resource and no version is stated for the R package.
- Building an unsigned network from absolute Pearson correlations discards the sign of co-regulation, which may matter for some downstream analyses.

## Evidence

- [abstract] Distribution of the network in an R/Shiny exploration package: "an R/Shiny package ( https://github.com/almeidasilvaf/SoyFungiGCN ) that serve as a public resource to explore soybean-pathogenic fungi interactions"
- [readme] Scope of the underlying network resource: "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
- [abstract] Network construction context: integrating transcriptomic data for candidate gene prioritization: "we integrated publicly available genome-wide association studies and transcriptomic data to prioritize candidate genes associated with resistance"
