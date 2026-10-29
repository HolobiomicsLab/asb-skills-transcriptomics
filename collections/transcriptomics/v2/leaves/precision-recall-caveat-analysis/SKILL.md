---
name: precision-recall-caveat-analysis
description: Use when a GRN method's aggregate precision-recall curve looks acceptable
  and you need to check whether that summary hides poor edge-rank recovery for individual
  regulators in the gold standard.
when_to_use_negative:
- The input is raw single-cell expression data rather than a completed, aggregated
  ranked edge list — run SINGE GLG and SINGE Aggregate first
- The gold standard does not decompose into individual regulators with known targets,
  so regulator-specific edge ranks cannot be computed
- You need a single overall precision-recall number rather than a diagnosis of regulator-level
  performance caveats
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: SINGE
  role: Produces the aggregated (Borda-ranked) edge-rank list whose per-regulator
    gold-standard edge ranks are evaluated
  repo: https://github.com/gitter-lab/SINGE
- name: MATLAB R2018a
  role: Environment for the per-regulator edge-rank boxplot rendering (or equivalent
    matplotlib plotting)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:b7eb8f0c1f68e5ce036cef0bd0a80f47074eaeb17888e0c09cebab090256e8a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB R2018a
  - SINGE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# precision-recall-caveat-analysis

## Summary

A regulator-specific evaluation that sorts gold-standard regulators by their outgoing edge count in the ESCAPE regulatory network and plots per-regulator distributions of predicted edge ranks, exposing regulators whose edge recovery is poor despite acceptable overall precision-recall performance of a GRN inference method.

## When to use

Use when a GRN method's aggregate precision-recall curve looks acceptable and you need to check whether that summary hides poor edge-rank recovery for individual regulators in the gold standard.

## When NOT to use

- The input is raw single-cell expression data rather than a completed, aggregated ranked edge list — run SINGE GLG and SINGE Aggregate first
- The gold standard does not decompose into individual regulators with known targets, so regulator-specific edge ranks cannot be computed
- You need a single overall precision-recall number rather than a diagnosis of regulator-level performance caveats

## Inputs

- SINGE aggregated Borda-ranked edge-rank predictions (SINGE_Ranked_Edge_List.txt) for the retinoic acid mESC dataset
- ESCAPE gold-standard regulatory network with 12 evaluated regulators (e.g., Dab2, Esrrb) and their documented target genes

## Outputs

- regulator_edge_rank_boxplots.png containing one boxplot per regulator (12 total), ordered by gold-standard outgoing-edge count on a shared rank axis
- Per-regulator gold-standard outgoing-edge degree counts

## How to apply

Load the SINGE aggregated (Borda-ranked) SINGE_Ranked_Edge_List.txt output for the retinoic acid mESC dataset, which comes from the SINGE GLG + SINGE Aggregate workflow. Load the ESCAPE gold-standard regulatory network and extract the 12 evaluated regulators (e.g., Dab2, Esrrb) with their documented target genes. Count each regulator's outgoing gold-standard edges and sort the 12 regulators by this degree. For each regulator, collect the SINGE-predicted ranks of its gold-standard edges and render per-regulator boxplots of the edge-rank distributions sharing a common rank axis, so regulators with many versus few gold-standard targets can be compared directly. The rationale is that ensemble aggregation over hyperparameters and subsamples yields a single ranked edge list whose global PR curve can mask regulator-level failures; the boxplots partially explain, and qualify, the overall precision-recall performance.

## Related tools

- **SINGE** (Produces the aggregated (Borda-ranked) edge-rank list whose per-regulator gold-standard edge ranks are evaluated) — https://github.com/gitter-lab/SINGE
- **MATLAB R2018a** (Environment for the per-regulator edge-rank boxplot rendering (or equivalent matplotlib plotting))

## Evaluation signals

- regulator_edge_rank_boxplots.png exists and contains exactly 12 boxplots, one per evaluated regulator
- Regulators on the x-axis are ordered by their number of outgoing ESCAPE gold-standard edges
- All boxplots share a common edge-rank axis so distributions are directly comparable across regulators
- At least one regulator with many gold-standard outgoing edges shows a poor (high) edge-rank distribution despite acceptable overall precision-recall performance of the method
- Each plotted edge rank corresponds to a gold-standard regulator-target pair present in both the SINGE ranked list and the ESCAPE network

## Limitations

- The regulator-specific evaluation only partially explains overall precision-recall performance; it demonstrates the aggregate metric is somewhat misleading but does not fully account for it
- The analysis is restricted to the 12 regulators evaluated in the ESCAPE gold standard, so conclusions do not extend to regulators absent from the gold standard
- Uninformative pseudotimes are a separate caveat noted for detailed examinations of SINGE performance

## Evidence

- [abstract] caveats finding: "However, a more detailed examination reveals caveats about poor performance for individual regulators and uninformative pseudotimes."
- [abstract] aggregate performance finding: "In two mouse embryonic stem cell differentiation datasets, SINGE outperforms other contemporary algorithms."
- [abstract] aggregation step: "It aggregates predictions from an ensemble of regression analyses to compile a ranked list of candidate interactions between transcriptional regulators and target genes."
- [readme] ranked edge list output: [evidence span withheld — anchor status: not_found]
