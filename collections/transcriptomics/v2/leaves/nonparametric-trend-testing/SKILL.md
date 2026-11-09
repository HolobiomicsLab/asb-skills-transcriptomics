---
name: nonparametric-trend-testing
description: Use when METAFlux community fluxes (e.g., glucose exchange reaction HMR_9034
  from a 4-cell-type TME model) must be tested against a pre-specified monotonic ordering
  across cell types, with non-normal flux distributions.
when_to_use_negative:
- The groups have no biologically pre-specified order (use Kruskal-Wallis or pairwise
  Wilcoxon instead, since Cuzick tests a monotonic trend only).
- Input is bulk RNA-seq with only two conditions rather than an ordered multi-group
  single-cell community flux comparison.
- Fluxes are already pathway-aggregated means, losing the per-bootstrap/per-reaction
  resolution needed for the trend test.
edam_operation: http://edamontology.org/operation_2239
tools:
- name: METAFlux
  role: Predicts per-cell-type glucose uptake fluxes via compute_sc_flux (num_cell=4,
    human_blood medium) that feed the trend test
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Environment in which the METAFlux fluxes are extracted, cubic-root-normalized
    (cbrt), and the Cuzick trend test is run
- name: Seurat
  role: Provides the scRNA-seq Seurat object split by patient_id as the cell-type
    expression source for calculate_avg_exp
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:ad511f6be91dbe00876a7b6b72d3c9555ac9030f9a16dfd3cfe79d0b6b7b9ff7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# nonparametric-trend-testing

## Summary

Applies a Cuzick nonparametric trend test to ordered cell-type groups (e.g., Epithelial < T cell < B cell < Myeloid) using glucose-uptake fluxes predicted by METAFlux, to test for a statistically significant monotonic ordering of metabolic phenotypes in the tumor microenvironment.

## When to use

Use when METAFlux community fluxes (e.g., glucose exchange reaction HMR_9034 from a 4-cell-type TME model) must be tested against a pre-specified monotonic ordering across cell types, with non-normal flux distributions.

## When NOT to use

- The groups have no biologically pre-specified order (use Kruskal-Wallis or pairwise Wilcoxon instead, since Cuzick tests a monotonic trend only).
- Input is bulk RNA-seq with only two conditions rather than an ordered multi-group single-cell community flux comparison.
- Fluxes are already pathway-aggregated means, losing the per-bootstrap/per-reaction resolution needed for the trend test.

## Inputs

- METAFlux single-cell flux matrix for glucose exchange reaction HMR_9034, dimension (num_cell*13082+1648)*number_of_bootstrap
- Cell-type labels in fixed order (Epithelial, T cell, B cell, Myeloid) matching the score-matrix column order
- Cell-type fractions vector summing to 1 (e.g., c(0.1,0.3,0.3,0.3))

## Outputs

- Cuzick trend test statistic and P value for the ordered cell-type comparison (expected P < 0.0001)
- Cubic-root-normalized glucose uptake flux values per cell type

## How to apply

First obtain per-cell-type fluxes: split the Seurat object by patient_id, build stratified bootstrapped average expression with METAFlux::calculate_avg_exp (myident='Cell_type', n_bootstrap 50-100, seed=1), compute MRAS with calculate_reaction_score against the 13,082-reaction Human-GEM, then run compute_sc_flux with num_cell=4, cell-type fractions summing to 1 in the same column order as the score matrix, and the human_blood medium (64 blood metabolites). Extract the internal_medium glucose exchange reaction HMR_9034 from the flux matrix and apply cubic-root normalization, sign(x)*|x|^(1/3), to tame the skewed flux scale. Then order groups by the hypothesis (Epithelial < T cell < B cell < Myeloid) and run the Cuzick nonparametric trend test on the normalized uptake values, expecting a significant monotonic increase Myeloid > B cell > T cell > Epithelial with P < 0.0001. The trend test requires an a priori ordered alternative; group order must be justified biologically, not chosen post hoc.

## Related tools

- **METAFlux** (Predicts per-cell-type glucose uptake fluxes via compute_sc_flux (num_cell=4, human_blood medium) that feed the trend test) — https://github.com/KChen-lab/METAFlux
- **R** (Environment in which the METAFlux fluxes are extracted, cubic-root-normalized (cbrt), and the Cuzick trend test is run)
- **Seurat** (Provides the scRNA-seq Seurat object split by patient_id as the cell-type expression source for calculate_avg_exp)

## Examples

```
flux=compute_sc_flux(num_cell=4,fraction=c(0.1,0.3,0.3,0.3),fluxscore=scores,medium=human_blood); glu=cbrt(flux['HMR_9034',])
```

## Evaluation signals

- Cuzick trend test yields P < 0.0001 with monotonic ordering Myeloid > B cell > T cell > Epithelial on glucose uptake fluxes
- Flux matrix dimension equals (num_cell*13082+1648)*number_of_bootstrap as stated for the single-cell setting
- Cubic-root normalization applied as sign(x)*|x|^(1/3) before testing
- Cell-type fraction vector sums to 1 and column order matches the Cell_type order used in calculate_avg_exp
- Glucose exchange reaction identified as HMR_9034 from the nutrient lookup file's 1648 exchange reactions

## Limitations

- No software version numbers are stated for METAFlux, R, or Seurat in the provided text, so reproducibility of exact flux values is uncertain.
- Glucose uptake prediction is not uniformly reliable: in the NCI-60 panel 'the predicted glucose uptake does not correlate as consistently as other metabolites'.
- Cuzick test assumes a monotonic alternative; it cannot detect non-monotonic differences in uptake across the four cell types.

## Evidence

- [other] expected trend ordering: [evidence span withheld — anchor status: not_found]
- [methods] community flux computation grounding: "flux=compute_sc_flux(num_cell = 4,fraction =c(0.1,0.3,0.3,0.3),fluxscore=scores,medium = human_blood)"
- [methods] exchange reaction lookup: "If one is strictly interested in metabolite uptake or release, one can search “nutrient lookup file” to obtain the metabolite exchange Reaction ID"
- [methods] cubic-root normalization function: "cbrt <- function(x) {
    sign(x) * abs(x)^(1/3)
}"
- [readme] README scope of single-cell capability: [evidence span withheld — anchor status: external_doc]
- [methods] underlying metabolic model: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
