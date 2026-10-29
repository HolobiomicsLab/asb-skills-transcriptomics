---
name: time-resolved-gene-module-annotation
description: Use when you have subject- and time-point-specific gene modules from
  multi-subject dynamic network inference (e.g. 79 donors x 4 time points) and need
  to identify which GO biological processes dominate at each stimulation time point.
when_to_use_negative:
- Networks/communities have not yet been inferred — this skill annotates existing
  MuDCoD community memberships, it does not build Dozer networks or run community
  detection itself.
- The analysis needs ontology enrichment of a single static gene list rather than
  comparing term frequencies across multiple time points and subjects.
- Only community detection accuracy (e.g. ARI against true labels on MuSDynamicDCBM
  simulations) is of interest, not biological interpretation of modules.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MuDCoD
  role: Produces the time-varying personalized communities whose membership gene sets
    are annotated; fit_predict with max_K=50, n_iter=30, alpha=0.05, beta=0.05
  repo: https://github.com/bo1929/MuDCoD
- name: Dozer
  role: Filters genes and constructs the robust personalized gene co-expression networks
    from scRNA-seq that feed MuDCoD
  repo: https://github.com/shanlu01/Dozer
- name: clusterProfiler
  role: Performs enrichGO biological-process over-representation analysis on each
    CRank-prioritized gene set
- name: CRank
  role: Scores and ranks each donor's inferred communities so the top-ranked community
    per donor per time point can be selected for enrichment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:3f5d92a95e837e8a8f268750a317a79e54176a8a3217c17083ff60a4e17a918d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - CRank
  - Dozer
  - MuDCoD
  - clusterProfiler
  - numpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# time-resolved-gene-module-annotation

## Summary

Annotates time-varying personalized gene modules (inferred with MuDCoD on Dozer-constructed networks) by running per-module GO biological-process over-representation with clusterProfiler enrichGO, then tabulating how often each enriched term recurs across subjects at each time point.

## When to use

Use when you have subject- and time-point-specific gene modules from multi-subject dynamic network inference (e.g. 79 donors x 4 time points) and need to identify which GO biological processes dominate at each stimulation time point.

## When NOT to use

- Networks/communities have not yet been inferred — this skill annotates existing MuDCoD community memberships, it does not build Dozer networks or run community detection itself.
- The analysis needs ontology enrichment of a single static gene list rather than comparing term frequencies across multiple time points and subjects.
- Only community detection accuracy (e.g. ARI against true labels on MuSDynamicDCBM simulations) is of interest, not biological interpretation of modules.

## Inputs

- MuDCoD community memberships per donor per time point (from MuDCoD.fit_predict on Dozer-filtered, Dozer-constructed gene co-expression networks)
- CRank-prioritized gene sets (top-ranked community per donor per time point)
- GO biological-process ontology annotation backing for clusterProfiler enrichGO

## Outputs

- Per-gene-set significant GO biological-process terms (BH adjusted FDR < 0.05)
- Term-frequency-ranked table of top-15 most frequent enriched GO terms per time point (go_enrichment_top15_table.csv)

## How to apply

For every donor and time point, select the top-ranked community (e.g. via CRank prioritization of MuDCoD fit_predict communities with max_K=50, n_iter=30, alpha=0.05, beta=0.05) to yield one gene set per subject-time combination (79x4=316 gene sets in the Soskic-2022 CD4+ T cell activation dataset). Run clusterProfiler enrichGO over-representation analysis on the biological-process ontology for each gene set, calling terms significant at Benjamini-Hochberg adjusted FDR 0.05. For each time point, count how frequently each significantly enriched term appears across donors and rank terms by frequency to build a Fig. 6b-style top-15 table. The rationale is that term-frequency trajectories across time points reveal when biological processes (e.g. 'positive regulation of T cell activation' at 16-h and 40-h, 'positive regulation of cytokine production' emerging at 40-h) become active, exploiting MuDCoD's robust identification of time-varying personalized modules.

## Related tools

- **MuDCoD** (Produces the time-varying personalized communities whose membership gene sets are annotated; fit_predict with max_K=50, n_iter=30, alpha=0.05, beta=0.05) — https://github.com/bo1929/MuDCoD
- **Dozer** (Filters genes and constructs the robust personalized gene co-expression networks from scRNA-seq that feed MuDCoD) — https://github.com/shanlu01/Dozer
- **clusterProfiler** (Performs enrichGO biological-process over-representation analysis on each CRank-prioritized gene set)
- **CRank** (Scores and ranks each donor's inferred communities so the top-ranked community per donor per time point can be selected for enrichment)

## Examples

```
MuDCoD.fit_predict(max_K=50, n_iter=30, alpha=0.05, beta=0.05)  # then enrichGO(OrgDb=org.Hs.eg.db, ont='BP', pAdjustMethod='BH', pvalueCutoff=0.05) per donor-time community
```

## Evaluation signals

- Table contains exactly 15 GO terms per time point (Fig. 6b-style structure check).
- 'positive regulation of T cell activation' appears among the top-15 terms only at the 16-h and 40-h time points, matching Fig. 6b.
- 'positive regulation of cytokine production' begins to appear more frequently at 40-h than at earlier time points.
- Every gene set yielded exactly one community per donor per time point (79x4=316 gene sets), and enrichment significance threshold is BH adjusted FDR 0.05.

## Limitations

- The article reports no concrete numeric ARI results in text (values are computed and printed but not shown), so community-level accuracy benchmarks must be recomputed rather than cited.
- Enrichment depends on CRank prioritization and the alpha/beta hyper-parameters; the README recommends cross-validation (examples/cross_validation.py) for alpha and beta rather than fixed values.
- Dozer-based network construction requires following the Dozer vignette only through Section 4: Gene Centrality Analysis (exclusive) to output networks in an .rda file; downstream steps of the vignette are not needed and may alter the network format MuDCoD expects.

## Evidence

- [methods] MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities: "Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities."
- [methods] Dozer filter genes construct robust networks: "We suggest to use Dozer [3] to filter genes and construct robust networks"
- [abstract] MuDCoD robust inference time-varying personalized gene modules: [evidence span withheld — anchor status: not_found]
- [abstract] personalized dynamic community detection aid exploration subject-specific biological processes vary across time: "Our results illustrate how personalized dynamic community detection can aid in the exploration of subject-specific biological processes that vary across time"
- [methods] cross-validation for alpha and beta hyper-parameters: "You can use cross-validation (see `examples/cross_validation.py`) for `alpha` and `beta`."
- [methods] Dozer vignette Section 4 .rda output: "It is sufficient to follow until "Section 4: Gene Centrality Analysis" (not included) for our purposes."
