---
name: similarity-weighted-term-sampling
description: Use when constructing a synthetic pathway-enrichment benchmark where
  a known set of dysregulated/true-positive terms must be chosen with controlled functional
  redundancy, given a pathway-term dataframe and a similarity matrix.
when_to_use_negative:
- You already have real differential expression data and a known biological ground
  truth — similarity sampling is only needed to synthesize known true positives.
- You want unrelated, maximally diverse active terms; setting rho high deliberately
  biases co-activation toward similar pathways.
- You are running the actual pareg enrichment regression — this skill is the benchmark-generation
  step that precedes it, not the enrichment scoring itself.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: pareg
  role: provides compute_term_similarities to build the Jaccard similarity matrix
    and similarity_sample to draw active terms via the (1-rho)U + rho*S_j probability
    vector
  repo: https://github.com/cbg-ethz/pareg
- name: tidyverse
  role: R data-manipulation toolkit used to construct the synthetic pathway database
    and assemble the gene/pvalue/in_study study table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:9ef158673ebbdaebf6e86afba857eea8995532f61008fde89ba946bea726922b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/pareg
  tools:
  - R
  - pareg
  - tidyverse
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# similarity-weighted-term-sampling

## Summary

Selects a subset of 'active' pathways as synthetic ground-truth true positives by sampling terms from a probability vector (1-rho)U + rho*S_j that mixes uniform weights with rows of a pathway similarity matrix (e.g. Jaccard), so co-activated terms are functionally related rather than random.

## When to use

Use when constructing a synthetic pathway-enrichment benchmark where a known set of dysregulated/true-positive terms must be chosen with controlled functional redundancy, given a pathway-term dataframe and a similarity matrix.

## When NOT to use

- You already have real differential expression data and a known biological ground truth — similarity sampling is only needed to synthesize known true positives.
- You want unrelated, maximally diverse active terms; setting rho high deliberately biases co-activation toward similar pathways.
- You are running the actual pareg enrichment regression — this skill is the benchmark-generation step that precedes it, not the enrichment scoring itself.

## Inputs

- data.frame df_terms of gene-to-pathway memberships (e.g. columns term/gene from a synthetic pathway database)
- pairwise pathway similarity matrix mat_similarities (e.g. computed with pareg::compute_term_similarities using jaccard)
- integer number of active terms to select (e.g. 5)
- similarity factor rho controlling the mixture weight of the similarity row versus uniform sampling

## Outputs

- vector of active term/pathway identifiers serving as ground-truth true positives
- synthetic study table (data.frame with columns gene, pvalue, in_study) exported e.g. as synthetic_study.csv
- pathway similarity matrix and active-terms ground truth exported alongside the study table

## How to apply

First build the pathway-term dataframe (in the pareg vignette: 2 gene groups, 10 pathways per group, each pathway drawing 10 genes from its group's 15 genes plus 10 from a 50-gene background set) and compute the pairwise similarity matrix with pareg::compute_term_similarities(df_terms, similarity_function = jaccard). Then call similarity_sample(mat_similarities, n_active) — e.g. 5 active terms — which seeds from a uniformly sampled term j and draws subsequent terms from the probability vector (1-rho)U + rho*S_j, so similarity factor rho tunes how strongly co-activation clusters related pathways. The resulting active_terms vector is the ground truth of true positives; differentially expressed genes are defined as the union of genes in the active pathways, and p-values are simulated as Beta(0.1,1) for DE genes versus Uniform(0,1) (Beta(1,1)) for all other genes. Set the RNG seed (set.seed(42)) for reproducibility and export the study table, similarity matrix and active-term ground truth (e.g. synthetic_study.csv).

## Related tools

- **pareg** (provides compute_term_similarities to build the Jaccard similarity matrix and similarity_sample to draw active terms via the (1-rho)U + rho*S_j probability vector) — https://github.com/cbg-ethz/pareg
- **tidyverse** (R data-manipulation toolkit used to construct the synthetic pathway database and assemble the gene/pvalue/in_study study table)

## Examples

```
mat_similarities <- pareg::compute_term_similarities(df_terms, similarity_function = jaccard); active_terms <- similarity_sample(mat_similarities, 5)
```

## Evaluation signals

- The similarity matrix is square, symmetric, with diagonal 1 when built with jaccard similarity.
- similarity_sample returns exactly the requested number of terms (e.g. 5) and all returned identifiers exist in df_terms.
- With a fixed seed (set.seed(42)), the sampled active-term set is reproducible run to run.
- Union of genes in the active pathways matches the set flagged differentially expressed, whose p-values are concentrated near 0 (Beta(0.1,1)), while non-DE genes follow Uniform(0,1).
- A plot/inspection of mat_similarities shows elevated similarity among co-sampled active terms, consistent with the (1-rho)U + rho*S_j mixture.

## Limitations

- The ground truth is synthetic: active terms reflect the chosen similarity measure (e.g. Jaccard) and rho, so benchmark conclusions depend on these construction choices.
- The Beta(0.1,1)/Uniform p-value simulation is an idealized noise model and may not capture real differential-expression signal structure.
- Pathway similarity measures can differ substantially between each other (as observed on GO Biological Process), so the sampled active set varies with the chosen similarity_function.

## Evidence

- [methods] select active pathways as true positives: "we select a subset of pathways to be activated. In a performance evaluation, these would be considered to be true positives."
- [methods] similarity_sample call: "active_terms <- similarity_sample(mat_similarities, 5)"
- [methods] compute_term_similarities call: "compute_term_similarities(
  df_terms,
  similarity_function = jaccard
)"
- [methods] p-value simulation scheme: "The p-values of genes considered to be differentially expressed are sampled from a Beta distribution centered at $0$."
- [readme] pareg repo description: "Pathway enrichment computations using a regularized regression approach to incorporate inter-pathway relations in the statistical model."
