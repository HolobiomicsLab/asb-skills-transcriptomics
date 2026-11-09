---
name: fdr-multiple-testing-correction
description: Use when performing large-scale single-cell hypothesis testing (co-expression
  edges, DE, CRISPR screen P values, or pathway enrichment over network edges) where
  thousands to millions of simultaneous tests require explicit FDR or Bonferroni correction
  before calling significance.
when_to_use_negative:
- When hypothesis counts are small (e.g. a handful of candidate genes), where Bonferroni/Q-value
  correction wastes power without controlling anything meaningful
- When analyzing arbitrary nonlinear associations — the article states Normalisr is
  not designed for hypothesis testing of these, so its exact P-values and FDR framework
  do not apply
- When data come from full-length (non-UMI) platforms, where P-value calibration and
  the nonlinear covariates underpinning the tests were not validated
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Normalisr
  role: Computes exact linear-model P-values without permutation and converts them
    to Q-values for co-expression edge calling (coex), outlier removal (qc_outlier),
    and DE
  repo: https://github.com/lingfeiwang/normalisr
- name: GOATOOLS
  role: Tests over-abundance of GO pathway gene pairs among network edges with Bonferroni-corrected
    P values
- name: BioServices
  role: Retrieves KEGG pathway annotations for Bonferroni-corrected enrichment testing
    of pathway gene pairs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:30d3a5c25ece04b937882d5e5b5b2a2bc5628d6f4cc5550585271b2ba2f5ca70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - BioServices
  - GOATOOLS
  - Normalisr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fdr-multiple-testing-correction

## Summary

Applies multiple-testing correction (FDR Q-values via linear-model association tests and Bonferroni correction for pathway enrichment) at every decision point of a Normalisr single-cell analysis, so that co-expression edges, outlier cells, and GO/KEGG pathway over-abundance calls remain statistically calibrated at genome scale.

## When to use

Use when performing large-scale single-cell hypothesis testing (co-expression edges, DE, CRISPR screen P values, or pathway enrichment over network edges) where thousands to millions of simultaneous tests require explicit FDR or Bonferroni correction before calling significance.

## When NOT to use

- When hypothesis counts are small (e.g. a handful of candidate genes), where Bonferroni/Q-value correction wastes power without controlling anything meaningful
- When analyzing arbitrary nonlinear associations — the article states Normalisr is not designed for hypothesis testing of these, so its exact P-values and FDR framework do not apply
- When data come from full-length (non-UMI) platforms, where P-value calibration and the nonlinear covariates underpinning the tests were not validated

## Inputs

- Matrix of raw association P-values or Normalisr Q-values from coex (all pairwise gene co-expression) or de runs, in tsv format
- Outlier-test P-values from compute_var/qc_outlier for cell removal
- Pathway-gene membership lists (GO terms via GOATOOLS, KEGG pathways via BioServices) and the co-expression edge list for enrichment testing

## Outputs

- FDR-thresholded co-expression edge list (significant pairs at Q ≤ 1e-15) feeding major_component_coex_network.tsv
- Retained/outlier cell labels from Bonferroni P ≤ 1e-10 outlier filtering
- Bonferroni-corrected GO/KEGG pathway enrichment table saved to pathway_enrichment.tsv (33 GO and 8 KEGG pathways at P ≤ 0.05)

## How to apply

Because Normalisr's linear models yield exact P-values without permutation, every association result is converted to Q-values (storey FDR) and thresholded by the analysis goal: for the dysfunctional CD8+ T-cell co-expression network on the MARS-seq GSE123139 melanoma data, use the strong cutoff Q ≤ 1e-15 to prioritize more direct gene interactions. During normalization, apply iterative outlier removal with a prior outlier proportion bound of 2% and remove low-variance outlier cells at Bonferroni P ≤ 1e-10, using Normalisr's qc_outlier. When testing whether GO or KEGG pathway gene pairs are over-abundant among network edges (via GOATOOLS and BioServices), correct with Bonferroni and retain pathways at P ≤ 0.05. Note the decision point that the co-expression Q-value cutoff feeds back into the iterative GO pathway removal ('cytosolic part' and 'chromosome condensation' programs via gotop), so cutoff choice affects how many iterations are needed before extracting the major connected component.

## Related tools

- **Normalisr** (Computes exact linear-model P-values without permutation and converts them to Q-values for co-expression edge calling (coex), outlier removal (qc_outlier), and DE) — https://github.com/lingfeiwang/normalisr
- **GOATOOLS** (Tests over-abundance of GO pathway gene pairs among network edges with Bonferroni-corrected P values)
- **BioServices** (Retrieves KEGG pathway annotations for Bonferroni-corrected enrichment testing of pathway gene pairs)

## Examples

```
python3 -m normalisr coex --input normalized_counts.tsv --q_cutoff 1e-15
```

## Evaluation signals

- Co-expression output retains only pairs with Q ≤ 1e-15; no edge in major_component_coex_network.tsv exceeds this Q-value
- Outlier-removal step uses prior outlier proportion bound 2% and Bonferroni P ≤ 1e-10; removed cells should be a small minority (~2% or less)
- pathway_enrichment.tsv reports Bonferroni-corrected P values and contains significant pathways only at P ≤ 0.05 (expected 33 GO and 8 KEGG pathways)
- Q-values should be uniformly calibrated under null (exact P-value estimation without permutation); compare against known gene functions — networks should recapitulate GO/KEGG functional similarity
- Enrichment count should be reported per co-expression cutoff since GO pathway removal is iterative given the Q-value cutoff

## Limitations

- Only UMI-based datasets were analyzed; P-value/FDR calibration on full-length platforms is unvalidated and may need revalidation of the nonlinear covariates
- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations, so exact P-values and FDR correction do not cover such tests
- Enrichment results depend on the chosen co-expression Q-value cutoff, and GO pathway removal itself is iterative — a different cutoff changes the pathway set

## Evidence

- [discussion] For co-expression network in dysfunctional T cells, we used a strong cutoff (Q ≤ 10−15) to prioritize more direct gene interactions: "For co-expression network in dysfunctional T cells, we used a strong cutoff (Q ≤ 10−15) to prioritize more direct gene interactions"
- [discussion] With the prior bound for outlier proportion as 2%, low-variance outlier cells with Bonferroni P ≤ 10−10 were removed: "With the prior bound for outlier proportion as 2%, low-variance outlier cells with Bonferroni P ≤ 10−10 were removed"
- [discussion] The GO pathway removal took an iterative process given the significance cutoff for co-expression Q-value: "The GO pathway removal took an iterative process given the significance cutoff for co-expression Q-value"
- [readme] exact P-value estimation without permutation: "exact P-value estimation without permutation"
- [discussion] We used GOATOOLS63 and BioServices64 for GO and KEGG pathways, respectively: "We used GOATOOLS63 and BioServices64 for GO and KEGG pathways, respectively"
