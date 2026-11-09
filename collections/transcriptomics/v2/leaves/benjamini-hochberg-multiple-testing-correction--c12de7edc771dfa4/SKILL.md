---
name: benjamini-hochberg-multiple-testing-correction
description: Use when many parallel significance tests have been run (e.g. one hypergeometric
  GO over-representation test per bicluster across 151 UniBic biclusters) and enrichment
  calls must control the false discovery rate before tallying proportions.
when_to_use_negative:
- The input is a set of bicluster p-values from a single test per hypothesis already
  corrected upstream — a second BH pass would over-correct.
- The analysis goal is bicluster discovery itself (the UniBic LCS/index-matrix steps),
  not post-hoc enrichment significance calling.
- Rank-based or resampling FDR methods are specifically required and the BH assumption
  of valid raw p-values across tests cannot be met.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GOstats
  role: Performs the hypergeometric GO over-representation test whose p-values are
    subsequently BH-adjusted to call biclusters enriched at p < 0.05
- name: runibic
  role: Supplies the UniBic biclusters (upstream input) whose gene members are tested
    for GO enrichment before BH correction
  repo: https://github.com/athril/runibic
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:23dfff845452031e1134477d2e41386951c0fd0b21a948c94dc16d2137db0967
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/athril/runibic
  tools:
  - GOstats
  - R
  - UniBic
  - runibic
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Applies Benjamini–Hochberg (BH) false-discovery-rate adjustment to the per-bicluster GO enrichment p-values computed with the GOstats hypergeometric test, so that UniBic biclusters on the GDS datasets can be called GO-enriched at adjusted p < 0.05.

## When to use

Use when many parallel significance tests have been run (e.g. one hypergeometric GO over-representation test per bicluster across 151 UniBic biclusters) and enrichment calls must control the false discovery rate before tallying proportions.

## When NOT to use

- The input is a set of bicluster p-values from a single test per hypothesis already corrected upstream — a second BH pass would over-correct.
- The analysis goal is bicluster discovery itself (the UniBic LCS/index-matrix steps), not post-hoc enrichment significance calling.
- Rank-based or resampling FDR methods are specifically required and the BH assumption of valid raw p-values across tests cannot be met.

## Inputs

- Per-bicluster raw GO enrichment p-values from the GOstats hypergeometric over-representation test
- Gene member lists of the 151 trend-preserving biclusters discovered by UniBic on the eight GDS datasets
- Background gene universe of the input expression matrix

## Outputs

- BH-adjusted p-values per bicluster / GO term
- Per-bicluster enrichment calls at adjusted p < 0.05
- go_enrichment_summary.csv with adjusted p-values and summary proportions (62/151 enriched, 41.1% average enrichment)

## How to apply

For each bicluster extracted from the UniBic run (trend-preserving biclusters, output o = 100 default, significance p < 0.05), run a hypergeometric GO over-representation test with the GOstats package against the background gene universe of the input expression matrix. Collect the resulting per-bicluster enrichment p-values and adjust them with the Benjamini–Hochberg procedure (R's p.adjust with method = 'BH'). Call a bicluster GO-enriched if any GO term reaches adjusted p < 0.05, then tally the enriched proportion and average enrichment level — here 62 of 151 biclusters enriched, an average enrichment level of 41.1% for UniBic, the highest among the compared tools. The BH step is essential because testing many GO terms across many biclusters inflates raw p-value significance; the same p < 0.05 significance convention is used throughout the UniBic pipeline.

## Related tools

- **GOstats** (Performs the hypergeometric GO over-representation test whose p-values are subsequently BH-adjusted to call biclusters enriched at p < 0.05)
- **runibic** (Supplies the UniBic biclusters (upstream input) whose gene members are tested for GO enrichment before BH correction) — https://github.com/athril/runibic

## Examples

```
adj_p <- p.adjust(pvals_per_term, method = "BH"); bicluster_enriched <- any(adj_p < 0.05)
```

## Evaluation signals

- The tally of enriched biclusters reproduces the reported 62 of 151 UniBic biclusters on the eight GDS datasets.
- The computed average enrichment level matches the reported 41.1%, the highest among the compared tools.
- Every enrichment call corresponds to at least one GO term with BH-adjusted p < 0.05 in go_enrichment_summary.csv.
- Adjusted p-values are monotone-compatible with the raw p-values and never smaller than them (BH property check).
- The background gene universe used in the hypergeometric test equals the gene set of the input expression matrix, not the GO annotation database.

## Limitations

- Bicluster genes not represented in the GO annotation or outside the defined universe are silently excluded, biasing enrichment tests.
- Calling a bicluster enriched if ANY GO term passes p < 0.05 is a liberal rule; BH controls term-level FDR but the bicluster-level tally is not itself FDR-corrected.
- UniBic's identification of general trend-preserving biclusters remains a challenging problem, so the set of tested biclusters (and hence enrichment proportions) depends on the biclustering run's parameters (o = 100, p < 0.05).

## Evidence

- [other] Adjust enrichment p-values with the Benjamini–Hochberg procedure and call a bicluster GO-enriched if any GO term reaches adjusted p < 0.05.: [evidence span withheld — anchor status: not_found]
- [other] run a hypergeometric GO over-representation test with the GOstats package against the background gene universe: [evidence span withheld — anchor status: not_found]
- [other] UniBic discovered 151 biclusters on the eight GDS datasets, of which 62 were GO-enriched: [evidence span withheld — anchor status: not_found]
- [methods] Those with p-values less than 0.05 are decreasingly ordered in their significance.: "Those with p-values less than 0.05 are decreasingly ordered in their significance."
