---
name: quantile-normalization-of-rna-seq-expression
description: Use when a filtered RNA-seq expression matrix spanning multiple stress
  conditions must be made distribution-comparable across samples prior to computing
  gene-gene correlations for network construction.
when_to_use_negative:
- The expression data are single-condition or single-tissue samples where between-sample
  distribution comparability is not a concern and simpler normalization suffices
- The downstream analysis requires signed networks preserving correlation sign, since
  this workflow deliberately builds an unsigned network using absolute correlation
  values
- Input is raw reads or a count matrix that has not yet been filtered; this skill
  starts from already-filtered expression data
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0085
tools:
- name: BioNERO
  role: Performs quantile normalization and expression adjustment of the filtered
    RNA-seq matrix and constructs the unsigned coexpression network using Pearson's
    r
- name: R
  role: Implementation environment in which BioNERO preprocessing and network construction
    are executed
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: SoyFungiGCN
  role: R/Shiny package that distributes the resulting coexpression network and the
    genes_modules data used for validation
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:99ddb7290fbbc0a14c9fff1b4cbe7385896c9cd531d2736baaa296ebd7eb2927
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - BioNERO
  - R
  - SoyFungiGCN
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# quantile-normalization-of-rna-seq-expression

## Summary

Preprocess filtered soybean RNA-seq expression data from five fungal-pathogen stress conditions with BioNERO's quantile normalization and expression adjustment, so sample-level distributions are comparable before building an unsigned Pearson-based coexpression network in the SoyFungiGCN workflow.

## When to use

Use when a filtered RNA-seq expression matrix spanning multiple stress conditions must be made distribution-comparable across samples prior to computing gene-gene correlations for network construction.

## When NOT to use

- The expression data are single-condition or single-tissue samples where between-sample distribution comparability is not a concern and simpler normalization suffices
- The downstream analysis requires signed networks preserving correlation sign, since this workflow deliberately builds an unsigned network using absolute correlation values
- Input is raw reads or a count matrix that has not yet been filtered; this skill starts from already-filtered expression data

## Inputs

- Filtered soybean RNA-seq expression matrix containing samples from the five fungal-pathogen stress conditions
- Gene-module assignments consistent with the genes_modules data released in the SoyFungiGCN package

## Outputs

- Quantile-normalized, expression-adjusted expression matrix
- Unsigned coexpression network with edge weights corresponding to Pearson correlations, stored as coexpression_network.rds

## How to apply

Load the filtered soybean RNA-seq expression matrix containing samples from the five fungal-pathogen stress conditions (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi). Apply quantile normalization and expression adjustment using BioNERO's expression preprocessing functions so that sample-level distributions become comparable. Compute pairwise gene-gene correlations on the adjusted matrix with Pearson's r, and assemble an unsigned coexpression network in BioNERO using absolute correlation values so both positively and negatively correlated gene pairs are retained as edges. Store the resulting network object as coexpression_network.rds. The rationale is that quantile normalization removes sample-wide distribution shifts that would otherwise bias correlation estimates and downstream gene-module assignments.

## Related tools

- **BioNERO** (Performs quantile normalization and expression adjustment of the filtered RNA-seq matrix and constructs the unsigned coexpression network using Pearson's r)
- **R** (Implementation environment in which BioNERO preprocessing and network construction are executed) — https://github.com/almeidasilvaf/SoyFungiGCN
- **SoyFungiGCN** (R/Shiny package that distributes the resulting coexpression network and the genes_modules data used for validation) — https://github.com/almeidasilvaf/SoyFungiGCN

## Examples

```
library(BioNERO); expr_adjusted <- exp_preprocess(expr_filtered, transform = "none", correction = "quantile"); net <- exp2gcn(expr_adjusted, cor_method = "pearson", net_type = "unsigned")
```

## Evaluation signals

- Verify the network was built on the quantile-normalized, adjusted expression data rather than the raw filtered matrix
- Verify the network is declared unsigned and edge weights correspond to Pearson correlations
- Confirm both positively and negatively correlated gene pairs are retained as edges via absolute correlation values
- Check that downstream gene-module assignments are consistent with the genes_modules data released in the SoyFungiGCN package
- Confirm sample-level distributions are comparable across the five fungal-pathogen stress conditions after adjustment

## Limitations

- The SoyFungiGCN package is marked as experimental lifecycle, so the released network and data may change without a changelog (no changelog found)
- Candidate genes and modules are specific to the five soybean-pathogenic fungi studied; the normalization-network pair is not validated for other organisms or stress conditions in this context

## Evidence

- [methods] BioNERO quantile normalization finding: [evidence span withheld — anchor status: not_found]
- [abstract] SoyFungiGCN distribution: "an R/Shiny package ( https://github.com/almeidasilvaf/SoyFungiGCN ) that serve as a public resource to explore soybean-pathogenic fungi interactions"
- [readme] README installation: "remotes::install_github("almeidasilvaf/SoyFungiGCN")"
