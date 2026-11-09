---
name: hotspot-module-display-annotation
description: Use when an analyzed VISION/PhyloVision object contains Hotspot modules
  that must be annotated against signature libraries and surfaced in the interactive
  web-based report for lineage-expression interpretation.
when_to_use_negative:
- Input is a raw expression matrix or unclustered tree with no Hotspot modules computed
  — module identification (runHotspot, compute_local_correlations) must precede annotation
- The goal is joint analysis of lineage and expression de novo rather than annotating
  existing modules against signatures — tools like iTOL or CeLaVi are explicitly noted
  as lacking automated interpretation, but this skill is only the reporting/annotation
  stage
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: PhyloVision
  role: Pipeline that annotates Hotspot modules against signatures and serves the
    interactive report via viewResults
  repo: https://github.com/YosefLab/VISION
- name: VISION
  role: Front-end on which the interactive web-based report is built; supplies signature-scoring
    procedure and createGeneSignature
  repo: https://github.com/YosefLab/VISION
- name: Hotspot
  role: Identifies informative genes and modules from the intersection of phylogeny
    and expression data via compute_local_correlations and compute_autocorrelations
  repo: https://github.com/YosefLab/Hotspot
- name: MSigDB
  role: Source of gene signature sets used for module enrichment annotation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:773dd6e828f1241f878b4e5c8d57b86ce59cafb801ab29eb8d9f471641df6ebd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - Hotspot
  - MSigDB
  - PhyloVision
  - R
  - VISION
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Hotspot module display and signature annotation

## Summary

Within the PhyloVision web report, Hotspot-derived heritable gene modules are annotated against user-provided gene signatures (e.g., MSigDB) via a hypergeometric enrichment test and rendered as collapsible module tables alongside the radial/linear phylogeny and UMAP/tSNE overlays, making phylogeny-associated expression programs interpretable.

## When to use

Use when an analyzed VISION/PhyloVision object contains Hotspot modules that must be annotated against signature libraries and surfaced in the interactive web-based report for lineage-expression interpretation.

## When NOT to use

- Input is a raw expression matrix or unclustered tree with no Hotspot modules computed — module identification (runHotspot, compute_local_correlations) must precede annotation
- The goal is joint analysis of lineage and expression de novo rather than annotating existing modules against signatures — tools like iTOL or CeLaVi are explicitly noted as lacking automated interpretation, but this skill is only the reporting/annotation stage

## Inputs

- Analyzed VISION/PhyloVision object in R with Hotspot modules computed (runHotspot, model='normal', tree=TRUE)
- User-provided gene signatures (createGeneSignature objects or MSigDB downloads)
- Hotspot local-correlation matrix from compute_local_correlations
- Optionally, previously saved report state (differential-expression cache and selections)

## Outputs

- Interactive web-based report (phylovision_report.html plus accompanying JSON state) with collapsible signature/Hotspot module tables annotated by hypergeometric-test enrichment
- Module-signature overlap significance table (FDR < 0.05 entries highlighted)

## How to apply

Run Hotspot with runHotspot (model='normal', tree=TRUE), then cluster the pairwise autocorrelation matrix from Hotspot's compute_local_correlations to define gene modules. Assess overlap between each module's gene set and user-provided signatures (created with createGeneSignature or downloaded from MSigDB) using a hypergeometric test in R, and report significance (entries with FDR < 0.05 are displayed in the report tables). Render via viewResults on the processed VISION object, using host='0.0.0.0' and browser=FALSE when serving remotely, so the collapsible signature/Hotspot module tables appear next to the PhyloPlot.js phylogeny and UMAP/tSNE panels. Optionally restore prior report state with load_de_cache and load_selections before rendering so previous selections are preserved.

## Related tools

- **PhyloVision** (Pipeline that annotates Hotspot modules against signatures and serves the interactive report via viewResults) — https://github.com/YosefLab/VISION
- **VISION** (Front-end on which the interactive web-based report is built; supplies signature-scoring procedure and createGeneSignature) — https://github.com/YosefLab/VISION
- **Hotspot** (Identifies informative genes and modules from the intersection of phylogeny and expression data via compute_local_correlations and compute_autocorrelations) — https://github.com/YosefLab/Hotspot
- **MSigDB** (Source of gene signature sets used for module enrichment annotation)

## Examples

```
sig <- createGeneSignature(name = "Interesting Process", sigData = sigData); viewResults(obj, port = 8080, host = "0.0.0.0", browser = FALSE)
```

## Evaluation signals

- Report files exist (file_exists) and parse as valid HTML (format_is)
- Report contains collapsible signature/Hotspot module tables with the expected FDR < 0.05 entries
- Report includes the radial/linear phylogeny display (PhyloPlot.js) and UMAP/tSNE overlay panels (contains_substring / expert_review)
- Module-signature overlap significance values reproduce from the hypergeometric test computed with R
- Restored state via load_de_cache and load_selections appears in the rendered report without re-running differential expression

## Limitations

- PhyloVision is implemented in R, which may provide challenges for utilizing separate software available in Python, like Cassiopeia
- Module annotation is limited to overlap-based hypergeometric enrichment; relative fitness quantification from phylogenies is not directly supported within the software
- The analysis of the metastasis dataset still lacks experimental validation to support the generated hypotheses

## Evidence

- [methods] hypergeometric module annotation: "Then, we assess significance using a hypergeometric test with R"
- [methods] module definition: "clustering the pairwise autocorrelation matrix computed with Hotspot’s “compute_local_correlations” function"
- [methods] signature creation: "sig <- createGeneSignature(name = "Interesting Process", sigData = sigData)"
- [results] web report: "Our interactive web-based report is built on our VISION front-end (DeTomaso et al., 2019)"
- [discussion] R limitation: "our implementation of PhyloVision in the R programming language may provide challenges for utilizing separate software available in Python"
