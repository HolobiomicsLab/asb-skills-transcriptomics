---
name: go-biological-process-enrichment-analysis
description: Use when you have many subject-/time-point-specific gene sets (e.g.,
  316 CRank-prioritized MuDCoD communities from Soskic-2022 CD4+ T cell activation
  networks) and need to identify which GO BP terms are recurrently enriched.
when_to_use_negative:
- The gene sets are not personalized/subject-specific modules but a single bulk gene
  list — a one-off enrichGO run without the per-time-point frequency aggregation is
  sufficient.
- The question concerns a different ontology branch (e.g., KEGG pathways or GO molecular
  function) rather than GO biological process.
- Upstream networks were not built with Dozer/MuDCoD and communities were not prioritized
  with CRank, so the 79×4=316 gene-set structure does not apply.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_0607
tools:
- name: MuDCoD
  role: Produces the personalized dynamic gene modules (via fit_predict with max_K=50,
    n_iter=30, alpha=0.05, beta=0.05) that are the gene sets tested for GO BP enrichment
  repo: https://github.com/bo1929/MuDCoD
- name: Dozer
  role: R package used to filter genes and construct robust subject-specific gene
    co-expression networks before community detection; follow its vignette up to Section
    4 (Gene Centrality Analysis) to output networks as an .rda file
  repo: https://github.com/shanlu01/Dozer
- name: numpy
  role: MuDCoD consumes multi-dimensional numpy arrays as the input network argument
    for community inference
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:59ededef696aa8123e1e3cbeb05772c662bf6d80a65abf2fb7a1843c0f9adbff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - Dozer
  - MuDCoD
  - numpy
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_0607
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# GO biological process enrichment analysis

## Summary

Over-representation analysis of inferred gene modules against the GO biological-process ontology using clusterProfiler enrichGO, with Benjamini-Hochberg FDR 0.05 significance, followed by cross-donor frequency ranking of enriched terms per time point (Fig. 6b-style tables).

## When to use

Use when you have many subject-/time-point-specific gene sets (e.g., 316 CRank-prioritized MuDCoD communities from Soskic-2022 CD4+ T cell activation networks) and need to identify which GO BP terms are recurrently enriched.

## When NOT to use

- The gene sets are not personalized/subject-specific modules but a single bulk gene list — a one-off enrichGO run without the per-time-point frequency aggregation is sufficient.
- The question concerns a different ontology branch (e.g., KEGG pathways or GO molecular function) rather than GO biological process.
- Upstream networks were not built with Dozer/MuDCoD and communities were not prioritized with CRank, so the 79×4=316 gene-set structure does not apply.

## Inputs

- 316 CRank-prioritized gene sets (top-ranked community per donor per time point from Dozer networks + MuDCoD memberships)
- GO biological-process ontology (used by clusterProfiler enrichGO)

## Outputs

- Per-gene-set enrichGO results with Benjamini-Hochberg adjusted p-values (significant at FDR 0.05)
- Per-time-point frequency-ranked table of enriched GO BP terms (go_enrichment_top15_table.csv, top-15 terms per time point)

## How to apply

Take each gene set produced by upstream community detection — here, the top CRank-ranked community per donor per time point (79 donors × 4 time points = 316 gene sets) from MuDCoD.fit_predict (max_K=50, n_iter=30, alpha=0.05, beta=0.05) on Dozer-constructed networks — and run clusterProfiler enrichGO in over-representation mode restricted to the biological-process ontology. Call a term significant at Benjamini-Hochberg adjusted FDR 0.05. For each time point, count how many donors each significant term appears in and rank terms by this frequency. Assemble a Fig. 6b-style table of the top-15 most frequent enriched terms per time point (e.g., written to go_enrichment_top15_table.csv); for the Soskic-2022 data the expected pattern is that 'positive regulation of T cell activation' is in the top 15 only at 16-h and 40-h, while 'positive regulation of cytokine production' rises at 40-h.

## Related tools

- **MuDCoD** (Produces the personalized dynamic gene modules (via fit_predict with max_K=50, n_iter=30, alpha=0.05, beta=0.05) that are the gene sets tested for GO BP enrichment) — https://github.com/bo1929/MuDCoD
- **Dozer** (R package used to filter genes and construct robust subject-specific gene co-expression networks before community detection; follow its vignette up to Section 4 (Gene Centrality Analysis) to output networks as an .rda file) — https://github.com/shanlu01/Dozer
- **numpy** (MuDCoD consumes multi-dimensional numpy arrays as the input network argument for community inference)

## Examples

```
library(clusterProfiler); ego <- enrichGO(gene = top_crank_community_genes, OrgDb = org.Hs.eg.db, ont = "BP", pAdjustMethod = "BH", pvalueCutoff = 0.05)
```

## Evaluation signals

- The top-15 table contains exactly 15 enriched GO BP terms for each of the 4 time points.
- The 316 gene sets count is satisfied: one CRank-top-ranked community per donor (79) per time point (4).
- 'positive regulation of T cell activation' appears among the top-15 terms only at 16-h and 40-h, matching Fig. 6b.
- 'positive regulation of cytokine production' starts to appear more frequently at 40-h, consistent with CD4+ T cell activation mechanisms.
- Every term listed in the table passed Benjamini-Hochberg adjusted FDR 0.05 in the enrichGO run.

## Limitations

- The reported temporal pattern (top-15 membership shifts between 16-h and 40-h) is specific to the Soskic-2022 CD4+ T cell activation dataset and the exact CRank prioritization + enrichGO pipeline; other datasets may need re-derivation.
- Significance depends on the BH FDR 0.05 cutoff and the community-selection rule (top CRank-ranked community per donor/time point); changing either alters which terms reach the frequency table.
- The enriched-index for MuDCoD/Dozer lacks concrete numeric ARI results and a discussion section, so upstream community-quality metrics must be recomputed rather than looked up.

## Evidence

- [methods] Run clusterProfiler enrichGO over-representation analysis on the biological-process ontology for each of the 316 gene sets, calling terms significant at Benjamini-Hochberg adjusted FDR 0.05.: [evidence span withheld — anchor status: not_found]
- [results] Positive regulation of T cell activation appears among the top 15 most frequent enriched GO terms only at 16-h and 40-h after stimulation: [evidence span withheld — anchor status: not_found]
- [readme] We suggest to use Dozer [3] to filter genes and construct robust networks: "We suggest to use Dozer [3] to filter genes and construct robust networks"
- [readme] It is sufficient to follow until "Section 4: Gene Centrality Analysis" (not included) for our purposes.: "It is sufficient to follow until "Section 4: Gene Centrality Analysis" (not included) for our purposes."
- [methods] MuDCoD takes multi-dimensional `numpy` arrays as the input network argument: "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
