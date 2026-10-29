---
name: cytokine-activity-inference-with-mlm
description: Use when you need to benchmark or validate cell-cell communication (CCC)
  method predictions against independent cytokine activity signatures, e.g. scoring
  cytokines on pseudobulk cell-type profiles from scRNA-seq and testing overlap with
  prioritised ligand-receptor interactions.
when_to_use_negative:
- When no cytokine target gene signatures are available for the species or the gene
  sets have restricted coverage of the cytokines of interest
- When the question concerns long-distance endocrine signalling, since scRNA-seq gene
  expression does not capture endocrine events
- When you need intracellular signalling inference (e.g. NicheNet-style ligand activity
  from downstream targets) rather than cytokine activity scoring from expression
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0085
tools:
- name: decoupleR
  role: Scores cytokine activities on pseudobulk profiles using its multivariate linear
    model (mlm), returning an activity estimate and p-value per cytokine per cell
    type
  repo: https://github.com/saezlab/decoupleR
- name: OmnipathR
  role: Retrieves cytokine-receptor interactions and cytokine target gene sets from
    OmniPath to build the decoupleR input network
  repo: https://github.com/saezlab/OmnipathR
- name: LIANA
  role: Ranks cytokine-receptor interactions per cell-type pair, applying CellChat/CellPhoneDB
    post-filtering before the overlap analysis
  repo: https://github.com/saezlab/liana
- name: CellChat
  role: Source of the post-filtering behaviour applied to ranked interactions (removing
    interactions with insufficiently expressed transmitter/receiver genes)
- name: CellPhoneDB
  role: Source of the post-filtering behaviour on minimum expression of heteromeric
    receptor-complex subunits
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:a8111c6635c62e22e52e638b93cb4db98e81fe34aeb843e992b3d0954cb0c450
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/decoupleR
  tools:
  - CellChat
  - CellPhoneDB
  - LIANA
  - OmnipathR
  - decoupleR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cytokine-activity-inference-with-mlm

## Summary

Infers cytokine activities from single-cell RNA-Seq-derived pseudobulk profiles using decoupleR's multivariate linear model (mlm) with OmniPath cytokine target gene sets, and validates CCC ligand-receptor predictions against these activity estimates via Fisher's exact tests over ranked interaction intervals.

## When to use

Use when you need to benchmark or validate cell-cell communication (CCC) method predictions against independent cytokine activity signatures, e.g. scoring cytokines on pseudobulk cell-type profiles from scRNA-seq and testing overlap with prioritised ligand-receptor interactions.

## When NOT to use

- When no cytokine target gene signatures are available for the species or the gene sets have restricted coverage of the cytokines of interest
- When the question concerns long-distance endocrine signalling, since scRNA-seq gene expression does not capture endocrine events
- When you need intracellular signalling inference (e.g. NicheNet-style ligand activity from downstream targets) rather than cytokine activity scoring from expression

## Inputs

- HER2+ breast cancer scRNA-seq count matrix with cell-type labels
- triple-negative breast cancer scRNA-seq count matrix with cell-type labels
- cytokine-receptor interactions and cytokine target gene sets retrieved from OmniPath via OmnipathR

## Outputs

- cytokine activity estimates and p-values per cytokine per cell type (decoupleR mlm scores)
- odds-ratio table over cumulative rank intervals (fig6a_odds_ratios.csv)
- Fig 6A odds-ratio panel visualisation

## How to apply

Aggregate HER2+ and triple-negative breast cancer scRNA-seq count matrices into pseudobulk expression profiles per cell type. Retrieve cytokine-receptor interactions and cytokine target gene sets from OmniPath via OmnipathR to serve as decoupleR input. Score cytokine activities on the pseudobulk profiles using decoupleR's multivariate linear model (mlm), which yields an activity estimate and p-value per cytokine per cell type. Rank cytokine-receptor interactions with LIANA per cell-type pair, applying CellChat/CellPhoneDB post-filtering behaviour (removing interactions with insufficiently expressed receptor-complex subunits or transmitter/receiver genes). Bin ranked interactions into cumulative rank intervals spanning 100-10000, tabulate overlap between highly ranked interactions and cytokines with significant mlm activity, and compute Fisher's exact test odds ratios per rank interval per dataset to assemble the Fig 6A odds-ratio matrix (saved as fig6a_odds_ratios.csv).

## Related tools

- **decoupleR** (Scores cytokine activities on pseudobulk profiles using its multivariate linear model (mlm), returning an activity estimate and p-value per cytokine per cell type) — https://github.com/saezlab/decoupleR
- **OmnipathR** (Retrieves cytokine-receptor interactions and cytokine target gene sets from OmniPath to build the decoupleR input network) — https://github.com/saezlab/OmnipathR
- **LIANA** (Ranks cytokine-receptor interactions per cell-type pair, applying CellChat/CellPhoneDB post-filtering before the overlap analysis) — https://github.com/saezlab/liana
- **CellChat** (Source of the post-filtering behaviour applied to ranked interactions (removing interactions with insufficiently expressed transmitter/receiver genes))
- **CellPhoneDB** (Source of the post-filtering behaviour on minimum expression of heteromeric receptor-complex subunits)

## Examples

```
cytokine_scores <- decoupleR::run_mlm(mat = pseudobulk_expr, net = omnipath_cytokine_targets, .source = "source", .target = "target")
```

## Evaluation signals

- decoupleR mlm output contains an activity estimate and p-value for each cytokine in each cell type's pseudobulk profile
- Odds ratios per rank interval are positive at the most prioritised ranks for all CCC methods, consistent with the reported generally positive trends
- Odds-ratio trends converge toward the random baseline as more interactions (up to 10000 cumulative rank) are considered
- Ranked interactions have passed CellChat/CellPhoneDB-style post-filtering (receptor-complex subunits and transmitter/receiver genes sufficiently expressed)
- fig6a_odds_ratios.csv contains one odds ratio per rank interval per dataset, reproducible as the Fig 6A matrix

## Limitations

- The agreement analysis is limited by restricted coverage of the cytokine activity signatures and receptor proteins
- Agreement trends converge toward the random baseline as more interactions are considered, so validation is strongest only for the most prioritised interactions
- Benchmarks cannot distinguish simple co-expression from actual CCC events, so cytokine-activity agreement is not definitive proof of communication

## Evidence

- [results] Cytokine activity validation workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract finding: "we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance and find that predictions are generally coherent with those"
- [methods] mlm scoring step: [evidence span withheld — anchor status: not_found]
- [other] OmniPath role: [evidence span withheld — anchor status: not_found]
- [discussion] Discussion limitation: "The limitations include the restricted coverage of the cytokine activity signatures and receptor proteins, and the technical shortcomings of current spatial transcriptomics technologies."
