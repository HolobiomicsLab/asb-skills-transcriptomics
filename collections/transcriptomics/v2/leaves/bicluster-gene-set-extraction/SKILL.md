---
name: bicluster-gene-set-extraction
description: Use when a run of UniBic/runibic has produced trend-preserving biclusters
  on expression data (e.g. the eight GDS datasets) and you must determine which biclusters
  are significantly GO-enriched and report their overall proportion.
when_to_use_negative:
- You only have a raw expression matrix and no biclusters yet — run UniBic first (the
  enrichment skill consumes bicluster gene members, not the matrix directly).
- You need enrichment statistics on individual gene lists or DEG sets rather than
  on the gene members of discovered biclusters — a plain GO over-representation workflow
  without the biclustering step is the right tool.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_0203
tools:
- name: UniBic / runibic
  role: Produces the trend-preserving biclusters whose gene members are extracted
    for enrichment testing (prior step in the workflow)
  repo: https://github.com/athril/runibic
- name: GOstats
  role: Runs the hypergeometric GO over-representation test for each bicluster's gene
    set against the input-matrix gene universe
- name: R
  role: Host language for the GOstats enrichment workflow and for reading runibic
    bicluster results via the biclust wrapper
- name: biclust
  role: Biclust package wrapper used to invoke BCUnibic and hold the bicluster result
    object from which gene members are pulled
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:5a0a25481a2dbda6e98d58e625c98780a0f5ff5bfb1e83e3be9d1fc3ab40316a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/athril/runibic
  tools:
  - GOstats
  - R
  - UniBic
  - UniBic / runibic
  - biclust
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# bicluster-gene-set-extraction

## Summary

Extracts the gene members of trend-preserving biclusters (e.g. from a UniBic/runibic run) and tests each bicluster for GO over-representation with a hypergeometric test, Benjamini–Hochberg-adjusted p < 0.05, to quantify what fraction of discovered biclusters are biologically meaningful.

## When to use

Use when a run of UniBic/runibic has produced trend-preserving biclusters on expression data (e.g. the eight GDS datasets) and you must determine which biclusters are significantly GO-enriched and report their overall proportion.

## When NOT to use

- You only have a raw expression matrix and no biclusters yet — run UniBic first (the enrichment skill consumes bicluster gene members, not the matrix directly).
- You need enrichment statistics on individual gene lists or DEG sets rather than on the gene members of discovered biclusters — a plain GO over-representation workflow without the biclustering step is the right tool.

## Inputs

- Trend-preserving biclusters from a UniBic/runibic run (e.g. biclust result object from BCUnibic(), output o = 100 default, significance p < 0.05) on the eight GDS real datasets
- Gene members of each bicluster (rows of the input expression matrix)
- Background gene universe = all genes in the input expression matrix

## Outputs

- go_enrichment_summary.csv with per-bicluster GO-enrichment calls (yes/no), Benjamini–Hochberg-adjusted p-values, enriched count / total count (62/151 expected), and average enrichment level (41.1% expected)

## How to apply

Load the trend-preserving biclusters output by the prior UniBic run (output o = 100 default, significance p < 0.05, per the UniBic step-6 ordering of biclusters by significance) and extract each bicluster's gene members. For each bicluster, run a hypergeometric GO over-representation test with the GOstats package, using the background gene universe of the input expression matrix rather than the whole genome. Apply the Benjamini–Hochberg procedure to the enrichment p-values and call a bicluster GO-enriched if any GO term reaches adjusted p < 0.05. Tally the enriched count against the total bicluster count and compute the average enrichment level, then write per-bicluster calls, adjusted p-values, and summary proportions to go_enrichment_summary.csv. This step is the biological-validation counterpart to UniBic's promise that 'biologically meaningful trend-preserving biclusters can be readily identified from noisy and complex large data'.

## Related tools

- **UniBic / runibic** (Produces the trend-preserving biclusters whose gene members are extracted for enrichment testing (prior step in the workflow)) — https://github.com/athril/runibic
- **GOstats** (Runs the hypergeometric GO over-representation test for each bicluster's gene set against the input-matrix gene universe)
- **R** (Host language for the GOstats enrichment workflow and for reading runibic bicluster results via the biclust wrapper)
- **biclust** (Biclust package wrapper used to invoke BCUnibic and hold the bicluster result object from which gene members are pulled)

## Examples

```
library(GOstats); hg <- hyperGTest(new("HyperGParams", geneIds=bic.genes, universeGeneIds=universe, annotation="org.Hs.eg.db", ontology="BP", pvalueCutoff=0.05, testDirection="over"))
```

## Evaluation signals

- Tally of GO-enriched biclusters over total biclusters reproduces 62/151 for UniBic on the eight GDS datasets.
- Computed average enrichment level reproduces the reported 41.1%.
- Every bicluster receives an explicit enriched/not-enriched call and a BH-adjusted p-value in go_enrichment_summary.csv (no missing rows).
- The background gene universe equals the full gene set of the input expression matrix (not all catalogued GO genes), matching the task card's workflow step 2.
- A bicluster is called enriched only if at least one GO term reaches BH-adjusted p < 0.05 — verify no calls were made on unadjusted p-values.

## Limitations

- UniBic's own workflow only guarantees significance ordering of trend-preserving biclusters at p < 0.05; identification of general (non-trend-preserving) biclusters is stated to remain a challenging problem, so GO coverage of biclusters is inherently partial (62 of 151 here).
- Enrichment calls depend on the chosen background universe; using a different universe than the input expression matrix will change which biclusters pass adjusted p < 0.05.
- The UniBic paper section notes missing implementation details (no versions/parameters reported in the discussion), so exact reproducibility may depend on the runibic defaults documented in the README rather than the paper.

## Evidence

- [other] For each bicluster, run a hypergeometric GO over-representation test with the GOstats package against the background gene universe of the input expression matrix.: [evidence span withheld — anchor status: not_found]
- [methods] UniBic output-bicluster significance threshold grounding.: "Those with p-values less than 0.05 are decreasingly ordered in their significance."
- [readme] runibic package availability in R.: "This package contains implementation of UniBic biclustering algorithm for gene expression data [Wang2016]"
- [abstract] Biological meaningfulness rationale for enrichment testing of biclusters.: "We found an elementary method by which biologically meaningful trend-preserving biclusters can be readily identified from noisy and complex large data."
