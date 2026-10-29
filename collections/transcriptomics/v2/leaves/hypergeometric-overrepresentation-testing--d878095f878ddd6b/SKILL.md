---
name: hypergeometric-overrepresentation-testing
description: Use when a clustering or biclustering step has produced groups of genes
  (e.g. UniBic trend-preserving biclusters from GDS expression matrices) and you must
  quantify what proportion is significantly GO-enriched.
when_to_use_negative:
- When no bicluster/clustering gene groups exist yet — run UniBic first to obtain
  bicluster gene members before testing enrichment.
- When the goal is a differential-expression analysis rather than asking whether bicluster
  gene members are over-represented in GO terms against the matrix's gene universe.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: GOstats
  role: R package used to run the hypergeometric GO over-representation test for each
    bicluster's gene members against the background universe
- name: R
  role: Language environment in which the GOstats enrichment tests and Benjamini–Hochberg
    adjustment are executed
- name: UniBic
  role: Upstream biclustering method whose trend-preserving bicluster gene members
    are the tested gene sets (run at o = 100 default, significance p < 0.05)
  repo: http://sourceforge.net/projects/unibic/files/?source=navbar
- name: runibic
  role: R/Bioconductor parallel implementation of UniBic used to produce biclusters
    on gene expression data prior to enrichment testing
  repo: github.com/athril/runibic
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:69b893f1ed551204830096d35e1123e355a1bab7e811e3948029de2d5898461d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GOstats
  - R
  - UniBic
  - runibic
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# hypergeometric-overrepresentation-testing

## Summary

Assess the biological relevance of gene sets (e.g. UniBic bicluster members) by running a hypergeometric GO over-representation test with the GOstats package against the background gene universe of the input expression matrix, then call a set enriched if any GO term reaches Benjamini–Hochberg-adjusted p < 0.05.

## When to use

Use when a clustering or biclustering step has produced groups of genes (e.g. UniBic trend-preserving biclusters from GDS expression matrices) and you must quantify what proportion is significantly GO-enriched.

## When NOT to use

- When no bicluster/clustering gene groups exist yet — run UniBic first to obtain bicluster gene members before testing enrichment.
- When the goal is a differential-expression analysis rather than asking whether bicluster gene members are over-represented in GO terms against the matrix's gene universe.

## Inputs

- Biclusters from a UniBic run (gene members per bicluster; UniBic run with output o = 100 default and significance p < 0.05)
- Input gene expression matrix (its gene set serves as the background gene universe for the hypergeometric test)

## Outputs

- go_enrichment_summary.csv containing per-bicluster GO-enrichment calls, Benjamini–Hochberg-adjusted p-values, and summary proportions of enriched biclusters
- Per-bicluster enriched GO term list with adjusted p < 0.05 calls

## How to apply

Extract the gene members of each bicluster discovered in the previous UniBic run (output o = 100 default, significance p < 0.05). For each bicluster, run a hypergeometric GO over-representation test with the GOstats package in R, using the full set of genes in the input expression matrix as the background gene universe so enrichment reflects over-representation rather than absolute annotation counts. Adjust the enrichment p-values across all GO terms with the Benjamini–Hochberg procedure, and call a bicluster GO-enriched if any GO term reaches adjusted p < 0.05. Tally the proportion of enriched biclusters and the average enrichment level, and write per-bicluster enrichment calls, adjusted p-values, and summary proportions to go_enrichment_summary.csv.

## Related tools

- **GOstats** (R package used to run the hypergeometric GO over-representation test for each bicluster's gene members against the background universe)
- **R** (Language environment in which the GOstats enrichment tests and Benjamini–Hochberg adjustment are executed)
- **UniBic** (Upstream biclustering method whose trend-preserving bicluster gene members are the tested gene sets (run at o = 100 default, significance p < 0.05)) — http://sourceforge.net/projects/unibic/files/?source=navbar
- **runibic** (R/Bioconductor parallel implementation of UniBic used to produce biclusters on gene expression data prior to enrichment testing) — github.com/athril/runibic

## Examples

```
Rscript -e 'library(GOstats); hg <- new("GOHyperGParams", geneIds=bicluster_genes, universeGeneIds=universe, ontology="BP", annotation="org.Hs.eg.db", test="over", pvalueCutoff=0.05); res <- hyperGTest(hg); padj <- p.adjust(pvalues(res), method="BH")'
```

## Evaluation signals

- The computed number of GO-enriched biclusters must reproduce the reported 62 of 151 UniBic biclusters on the eight GDS datasets.
- The computed average enrichment level must reproduce the reported 41.1% for UniBic, the highest among the compared tools.
- Every bicluster's enrichment p-values are adjusted with the Benjamini–Hochberg procedure before the adjusted p < 0.05 call; unadjusted p-values must not be used for the call.
- The background gene universe for each hypergeometric test equals the set of genes in the input expression matrix, not the whole GO annotation space.
- go_enrichment_summary.csv is produced and contains one enrichment call and adjusted p-values per bicluster.

## Limitations

- Enrichment depends on the quality of upstream biclustering; the identification of general trend-preserving biclusters remains a challenging problem (article limitation).
- Enrichment calls are threshold-sensitive: Benjamini–Hochberg-adjusted p < 0.05 on any GO term defines 'enriched', so biclusters with borderline terms may flip under different adjustment contexts.
- The result is relative to the background universe used (the input matrix's genes); changing the universe changes the hypergeometric test and the enriched-bicluster proportion.

## Evidence

- [other] UniBic discovered 151 biclusters on the eight GDS datasets, of which 62 were GO-enriched, giving the highest average enrichment level of 41.1% among the compared tools.: [evidence span withheld — anchor status: not_found]
- [other] For each bicluster, run a hypergeometric GO over-representation test with the GOstats package against the background gene universe of the input expression matrix.: [evidence span withheld — anchor status: not_found]
- [other] Adjust enrichment p-values with the Benjamini–Hochberg procedure and call a bicluster GO-enriched if any GO term reaches adjusted p < 0.05.: [evidence span withheld — anchor status: not_found]
- [methods] The algorithm locates trend-preserving biclusters within complex and noisy data and is considered one of the most accurate among biclustering methods.: "The algorithm locates trend-preserving biclusters within complex and noisy data and is considered one of the most accurate among biclustering methods."
- [methods] We calculate the significance value for those trend-preserving biclusters obtained in step 5. Those with p-values less than 0.05 are decreasingly ordered in their significance.: "We calculate the significance value for those trend-preserving biclusters obtained in step 5. Those with p-values less than 0.05 are decreasingly ordered in their significance."
