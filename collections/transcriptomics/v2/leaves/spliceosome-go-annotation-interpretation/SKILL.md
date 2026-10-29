---
name: spliceosome-go-annotation-interpretation
description: Use when building the CRIT feature matrix for candidate circRNA-regulating
  RBPs and you need to quantify each gene's functional proximity to mRNA splicing
  via GO annotation and semantic similarity.
when_to_use_negative:
- You already have a merged CRIT feature matrix (MATRIX.txt) and only need to run
  NMF clustering or validate candidates — use NMF.R / RBP_analysis.R instead
- You need gene-alteration, survival, or circRNA-binding features (Zp, Zs, ρm, C value)
  rather than GO-splicing similarity
- The studied biological system is unrelated to splicing-dependent circRNA biogenesis
  (e.g., untested non-cancer, non-spliceosome contexts), since Zg is defined specifically
  against GO:0000398
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0203
tools:
- name: GoSemSim
  role: R package used to compute GO semantic similarity scores between each gene's
    annotated GO terms and the selected term GO:0000398, which are then Z-scored to
    produce the Zg feature
- name: CRIT
  role: NMF-based pipeline that consumes the Zg feature together with ρm, Zp, Zs,
    and C value in the merged MATRIX.txt to cluster candidate circRNA-regulating RBPs
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: GOSim
  role: Companion R package installed alongside GoSemSim for GO similarity computation
    in the CRIT dependency set
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:c6c69b470c4a88d1642042afbfb761cc2909e4670c75f78f1f1619749f24fc9d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  repo_url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CRIT
  - GOSim
  - GoSemSim
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spliceosome-go-annotation-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the Zg feature of the CRIT pipeline: a Z-score of GO semantic similarity between each gene's annotations and the GO:0000398 term ('mRNA splicing, via spliceosome'), used to capture how closely an RBP's biological role matches spliceosomal circRNA formation.

## When to use

Use when building the CRIT feature matrix for candidate circRNA-regulating RBPs and you need to quantify each gene's functional proximity to mRNA splicing via GO annotation and semantic similarity.

## When NOT to use

- You already have a merged CRIT feature matrix (MATRIX.txt) and only need to run NMF clustering or validate candidates — use NMF.R / RBP_analysis.R instead
- You need gene-alteration, survival, or circRNA-binding features (Zp, Zs, ρm, C value) rather than GO-splicing similarity
- The studied biological system is unrelated to splicing-dependent circRNA biogenesis (e.g., untested non-cancer, non-spliceosome contexts), since Zg is defined specifically against GO:0000398

## Inputs

- List of candidate RBPs with identifiers (Ensembl gene id, HGNC symbol, Entrez gene id, UniProt SwissProt), e.g. RBP_list.RData with 1344 RBPs
- GO annotations for each gene (via org.Hs.eg.db / clusterProfiler)
- Selected functional GO term: GO:0000398, mRNA splicing, via spliceosome

## Outputs

- GO_sim_Z_Score.txt: feature Zg of each gene
- Zg column incorporated into MATRIX.txt (merged candidate regulator feature matrix)

## How to apply

Annotate each candidate RBP (from the RBP_list.RData set of 1344 RBPs, identified by Ensembl gene id, HGNC symbol, Entrez id, and UniProt SwissProt) with its GO terms, then compute the semantic similarity between those terms and the single selected term GO:0000398 (mRNA splicing, via spliceosome) using the R package 'GoSemSim'. Convert the per-gene similarity scores into a Z-score so they are comparable with the other CRIT features (ρm, Zp, Zs, C value) before merging them in GET_feauters_MATRIX.R. The rationale is that a true circRNA regulator must participate in circRNA formation, so high Zg marks core splicing proteins and peripheral factors; downstream Welch t tests across the 51 CRIT feature columns confirm Zg is among the top features separating authentic regulators from other RBPs (adjusted p < 0.05).

## Related tools

- **GoSemSim** (R package used to compute GO semantic similarity scores between each gene's annotated GO terms and the selected term GO:0000398, which are then Z-scored to produce the Zg feature)
- **CRIT** (NMF-based pipeline that consumes the Zg feature together with ρm, Zp, Zs, and C value in the merged MATRIX.txt to cluster candidate circRNA-regulating RBPs) — https://github.com/BioinformaticsSTU/CRIT
- **GOSim** (Companion R package installed alongside GoSemSim for GO similarity computation in the CRIT dependency set)

## Examples

```
BiocManager::install("biomaRt","survival","org.Hs.eg.db","clusterProfiler","GOSemSim","GOSim","readr","stringr","NMF","ggplot2","ComplexHeatmap")
```

## Evaluation signals

- GO_sim_Z_Score.txt is produced containing a Zg value for every gene in the candidate RBP list
- Zg values are Z-scored (mean ~0) and comparable in scale to the other CRIT features (ρm, Zp, Zs, C value) merged into MATRIX.txt
- In the downstream per-feature two-sample Welch t test between predicted-regulator and non-predicted NMF clusters, Zg shows a significant group difference (Benjamini-Hochberg adjusted p < 0.05) and ranks among the top distinguishing features alongside Zp
- Predicted regulators enriched for high Zg are consistent with the reported result that authentic circRNA regulators are core splicing proteins and peripheral factors

## Limitations

- Zg is defined against a single functional term (GO:0000398, mRNA splicing, via spliceosome), so genes' splicing-relatedness is judged only through this one term
- The skill relies on curated GO annotations; RBPs with sparse or absent GO annotation will receive uninformative Zg scores
- CRIT was developed and validated for cancers; generalization to other diseases such as neurodegenerative disease is untested per the article's discussion

## Evidence
