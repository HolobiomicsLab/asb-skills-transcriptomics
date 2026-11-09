---
name: gold-standard-interaction-mapping
description: Use when a ranked GRN prediction (e.g., SINGE_Ranked_Edge_List.txt) must
  be audited per regulator against a gold standard, to check whether aggregate precision-recall
  conceals poor performance for individual regulators.
when_to_use_negative:
- You only need an overall precision-recall curve for a GRN method and are not investigating
  regulator-level caveats
- No gold standard with documented regulator-target edges is available for the dataset
  (e.g., the ESCAPE evaluated regulators do not cover your system)
- The prediction is a binary adjacency matrix rather than a ranked edge list — edge
  ranks for boxplots cannot be formed
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: SINGE
  role: Produces the Borda-aggregated ranked edge list whose gold-standard edge ranks
    are decomposed per regulator
  repo: https://github.com/gitter-lab/SINGE
- name: MATLAB R2018a
  role: Tested execution environment for running the SINGE GLG + SINGE Aggregate workflow
    that generates the ranked edge list
  repo: https://github.com/gitter-lab/SINGE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:c1d8a073e246ff19c02b99b8017abc65183c1227e0da2ac3a6681bf5c877d173
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB R2018a
  - SINGE
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gold-standard-interaction-mapping

## Summary

A regulator-specific evaluation skill for GRN inference: map each evaluated regulator's gold-standard edges (e.g., the 12 regulators in the ESCAPE mESC standard such as Dab2, Esrrb) onto SINGE's Borda-aggregated ranked edge list, and render per-regulator boxplots of edge ranks sorted by gold-standard outgoing-edge count to expose regulators whose predictions are poor despite acceptable overall precision-recall performance.

## When to use

Use when a ranked GRN prediction (e.g., SINGE_Ranked_Edge_List.txt) must be audited per regulator against a gold standard, to check whether aggregate precision-recall conceals poor performance for individual regulators.

## When NOT to use

- You only need an overall precision-recall curve for a GRN method and are not investigating regulator-level caveats
- No gold standard with documented regulator-target edges is available for the dataset (e.g., the ESCAPE evaluated regulators do not cover your system)
- The prediction is a binary adjacency matrix rather than a ranked edge list — edge ranks for boxplots cannot be formed

## Inputs

- SINGE aggregated (Borda-ranked) edge-rank predictions (SINGE_Ranked_Edge_List.txt) for the retinoic acid mESC dataset from the SINGE GLG + SINGE Aggregate workflow
- ESCAPE gold-standard regulatory network with 12 evaluated regulators (e.g., Dab2, Esrrb) and their documented target genes
- MATLAB R2018a environment (article-tested version) for the SINGE workflow, with matplotlib (Python) or MATLAB plotting for the boxplots

## Outputs

- regulator_edge_rank_boxplots.png: one boxplot per regulator (12 total), ordered by gold-standard outgoing-edge count, sharing a common edge-rank axis
- Per-regulator gold-standard outgoing-edge counts used for the sort order

## How to apply

Load the SINGE aggregated Borda-ranked edge-rank predictions (SINGE_Ranked_Edge_List.txt produced by the SINGE GLG + SINGE Aggregate workflow) for the retinoic acid mESC dataset. Load the ESCAPE gold-standard regulatory network and extract the 12 evaluated regulators (e.g., Dab2, Esrrb) with their documented target genes. Count each regulator's outgoing gold-standard edges and sort the regulators by this degree. For each regulator, collect the SINGE-predicted ranks of its gold-standard edges and draw per-regulator boxplots of the edge-rank distributions sharing a common rank axis. The rationale, per the Cell Reports SINGE paper, is that overall precision-recall evaluation is partially explained by — and can be somewhat misleading about — individual regulators, motivating this regulator-level decomposition.

## Related tools

- **SINGE** (Produces the Borda-aggregated ranked edge list whose gold-standard edge ranks are decomposed per regulator) — https://github.com/gitter-lab/SINGE
- **MATLAB R2018a** (Tested execution environment for running the SINGE GLG + SINGE Aggregate workflow that generates the ranked edge list) — https://github.com/gitter-lab/SINGE

## Examples

```
edges = pd.read_csv("SINGE_Ranked_Edge_List.txt", sep="\t"); edges[edges.regulator.isin(evaluated_regulators)].boxplot(column="rank", by="regulator")
```

## Evaluation signals

- regulator_edge_rank_boxplots.png exists
- The figure contains exactly one boxplot per regulator (12 total)
- Boxplots are ordered by each regulator's gold-standard outgoing-edge count (ascending or decreasing per the specified sort)
- The figure visually exposes regulators whose edge-rank distributions are poor despite acceptable overall precision-recall performance
- All 12 regulators in the plot match the ESCAPE-evaluated regulator set (e.g., Dab2, Esrrb) and their ranks come from the SINGE aggregated output

## Limitations

- Regulator-specific evaluation only 'partially explains' overall precision-recall performance — it does not fully decompose it
- The article's detailed examination also flags caveats about uninformative pseudotimes, which this skill does not address
- The SINGE README warns that the master branch may be unstable; a versioned release should be used so the ranked edge list being audited is reproducible

## Evidence

- [abstract] Caveats about poor performance for individual regulators: "However, a more detailed examination reveals caveats about poor performance for individual regulators and uninformative pseudotimes."
- [methods] Regulator subset support in SINGE: "SINGE version 0.3.0 introduced functionality where a subset of the gene list is earmarked as candidate regulators."
- [readme] Ranked edge list output format: [evidence span withheld — anchor status: not_found]
