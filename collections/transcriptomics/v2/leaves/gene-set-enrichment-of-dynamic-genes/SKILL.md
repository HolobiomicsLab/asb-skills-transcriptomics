---
name: gene-set-enrichment-of-dynamic-genes
description: Use when scLANE (or a similar trajectory DE tool) has called genes significantly
  dynamic along a Slingshot/Monocle3 trajectory at FDR-adjusted p < 0.01, and you
  need pathway-level interpretation of lineage-specific dynamics such as osteoblastic
  vs. apoptotic fates.
when_to_use_negative:
- The gene list is not derived from trajectory differential testing — without scLANE
  dynamic-gene calls (FDR < 0.01) and the slope > 5 / < -5 filter, the enrichment
  targets are not defined
- The organism/gene identifiers are incompatible with the mouse-oriented collections
  used in this workflow (BioPlanet 2019, MSigDB mouse GO biological processes)
- The goal is to evaluate trajectory or embedding quality itself — that is Escort's
  role, not downstream gene set enrichment
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Enrichr
  role: Web service used to perform gene set enrichment on dynamic genes against BioPlanet
    2019 and MSigDB mouse GO biological processes
- name: scLANE
  role: Generates the dynamic gene lists (FDR < 0.01; last-segment slope > 5 or <
    -5) that serve as enrichment input
- name: clusterProfiler
  role: Bioconductor enrichment package bundled as an Escort dependency, with org.Mm.eg.db
    for mouse gene mapping
  repo: https://github.com/xiaorudong/Escort
- name: Escort
  role: R package whose dependencies include clusterProfiler and organism annotation
    DBs supporting the enrichment step
  repo: https://github.com/xiaorudong/Escort
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:bc235a00523ab062a147c151a77814ee6def90a1faf5467a4af6492197a31b3c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaorudong/Escort
  repo_url: https://github.com/xiaorudong/Escort
  tools:
  - Enrichr
  - Escort
  - R
  - clusterProfiler
  - scLANE
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-set-enrichment-of-dynamic-genes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interpret trajectory differential-expression results by submitting scLANE dynamic genes (FDR < 0.01, extreme last-segment slopes) to Enrichr using the BioPlanet 2019 collection and the GSEA Molecular Signatures Database for mouse (GO biological processes), as done in the Escort paper's Col10a1 hypertrophic chondrocyte case study.

## When to use

Use when scLANE (or a similar trajectory DE tool) has called genes significantly dynamic along a Slingshot/Monocle3 trajectory at FDR-adjusted p < 0.01, and you need pathway-level interpretation of lineage-specific dynamics such as osteoblastic vs. apoptotic fates.

## When NOT to use

- The gene list is not derived from trajectory differential testing — without scLANE dynamic-gene calls (FDR < 0.01) and the slope > 5 / < -5 filter, the enrichment targets are not defined
- The organism/gene identifiers are incompatible with the mouse-oriented collections used in this workflow (BioPlanet 2019, MSigDB mouse GO biological processes)
- The goal is to evaluate trajectory or embedding quality itself — that is Escort's role, not downstream gene set enrichment

## Inputs

- scLANE per-gene trajectory differential expression results (test statistics, FDR-adjusted p-values, segment slopes)
- Dynamic gene lists filtered at FDR-adjusted p < 0.01 and last-segment slope > 5 or < -5, per lineage
- Mouse gene symbols mapped via org.Mm.eg.db
- Enrichr gene set collections: BioPlanet 2019 and GSEA Molecular Signatures Database for mouse (GO biological processes)

## Outputs

- Ranked enriched pathway terms with enrichment statistics per lineage (e.g. osteoblastic fate vs. terminal apoptosis)
- Lineage-level pathway interpretation of hypertrophic chondrocyte transdifferentiation
- Comparison of dynamic-gene enrichment between Escort-recommended and original embeddings (figure6_embedding_comparison.png)

## How to apply

First run scLANE with default settings on each trajectory and call genes significantly dynamic at FDR-adjusted p-value < 0.01, then filter for genes enriched at the trajectory ends using a last-segment slope > 5 or < -5 cutoff. Submit the resulting dynamic gene lists — ideally split per lineage, e.g. the osteoblastic fate lineage versus the terminal apoptosis lineage in the Long et al. 2022 GSE190616 Col10a1 dataset — to Enrichr against the BioPlanet 2019 collection and the GSEA Molecular Signatures Database for mouse (GO biological processes). The Escort package ships clusterProfiler plus org.Hs.eg.db and org.Mm.eg.db as Bioconductor dependencies, so enrichment can also be run locally with mouse gene-symbol mapping. Compare enrichment results between the Escort-recommended embedding and the original Monocle3-default embedding to reproduce the paper's Figure 6 contrast, where the recommended embedding revealed additional insight into hypertrophic chondrocyte transdifferentiation.

## Related tools

- **Enrichr** (Web service used to perform gene set enrichment on dynamic genes against BioPlanet 2019 and MSigDB mouse GO biological processes)
- **scLANE** (Generates the dynamic gene lists (FDR < 0.01; last-segment slope > 5 or < -5) that serve as enrichment input)
- **clusterProfiler** (Bioconductor enrichment package bundled as an Escort dependency, with org.Mm.eg.db for mouse gene mapping) — https://github.com/xiaorudong/Escort
- **Escort** (R package whose dependencies include clusterProfiler and organism annotation DBs supporting the enrichment step) — https://github.com/xiaorudong/Escort

## Examples

```
enrichr(dynamic_genes, databases = c("BioPlanet_2019"))
```

## Evaluation signals

- Dynamic genes pass the FDR-adjusted p-value < 0.01 criterion before enrichment
- Genes retained for enrichment meet the last-segment slope > 5 or < -5 filter (endpoint-enriched dynamics)
- Enrichment is run separately on lineage-specific gene sets (osteoblastic fate vs. terminal apoptosis) rather than a single pooled list
- Enrichment contrast between embeddings reproduces the paper's Figure 6 comparison — recommended embedding yields additional transdifferentiation insight
- Returned terms come from the specified collections: BioPlanet 2019 and GSEA MSigDB mouse GO biological processes

## Limitations

- Gene set collections specified are mouse-oriented (GSEA MSigDB for mouse GO biological processes); other organisms need adjusted collections
- Enrichment quality depends entirely on upstream dynamic-gene calling thresholds (FDR < 0.01, slope > 5 or < -5); arbitrary slope cutoffs can shift results
- Escort is not guaranteed to identify a recommended embedding among the choices evaluated, so the enrichment comparison step may lack a recommended-embedding arm

## Evidence

- [other] Enrichr with BioPlanet 2019 and MSigDB: [evidence span withheld — anchor status: not_found]
- [other] scLANE dynamic gene criteria: [evidence span withheld — anchor status: not_found]
