---
name: rbp-ncrna-interaction-network-construction
description: Use when building the prior interaction graph needed to search RBP-ncRNA
  circuits (RNCs) from multi-omics differential expression scores, e.g. TCGA ESCA/BRCA
  or APP/PS1 mouse data scored with fRNC.
when_to_use_negative:
- You have no ENCORI/CLIP-seq or PARE-derived interaction data and only a user-supplied
  generic PPI or co-expression graph — fRNC's interStringency pipeline is specific
  to ENCORI RBP-ncRNA interactions.
- You need low-stringency interactions including non-experimental predictions — interStringency
  with stringency='high' restricts to CLIP experimental evidence count >= 3, discarding
  weaker support by design.
- The goal is differential expression itself rather than circuit detection — use the
  fRNC DEGs/edgeR step first; this skill only builds the interaction prior.
edam_operation: http://edamontology.org/operation_3902
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3523
tools:
- name: fRNC
  role: Provides interStringency, IDsymbol mapping, and the three-column edge-list
    format consumed by runmodule
  repo: https://github.com/BioinformaticsSTU/fRNC
- name: ENCORI
  role: Source of CLIP-evidence-supported RBP-ncRNA interaction tables filtered at
    high stringency (CLIP evidence >= 3)
- name: R
  role: Runtime environment for the fRNC package and its dependency installation helper
    install_dependpackages
  repo: https://github.com/BioinformaticsSTU/fRNC
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_008
  - task_002
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:0b7b98e688ea286bc86c9d1632890948eabc05e060e5ab05f4d915513082c924
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/fRNC
  repo_url: https://github.com/BioinformaticsSTU/fRNC
  tools:
  - ENCORI
  - R
  - edgeR
  - fRNC
  edam_operation: http://edamontology.org/operation_3902
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# rbp-ncrna-interaction-network-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct a high-stringency RBP-ncRNA interaction network in the fRNC R package by extracting ENCORI (starBase) CLIP-seq/PARE-derived interactions with interStringency and standardizing node identifiers (Ensembl, miRBase MIMAT, circBase IDs) into a node_gene_ID/type/target_gene_ID edge list.

## When to use

Use when building the prior interaction graph needed to search RBP-ncRNA circuits (RNCs) from multi-omics differential expression scores, e.g. TCGA ESCA/BRCA or APP/PS1 mouse data scored with fRNC.

## When NOT to use

- You have no ENCORI/CLIP-seq or PARE-derived interaction data and only a user-supplied generic PPI or co-expression graph — fRNC's interStringency pipeline is specific to ENCORI RBP-ncRNA interactions.
- You need low-stringency interactions including non-experimental predictions — interStringency with stringency='high' restricts to CLIP experimental evidence count >= 3, discarding weaker support by design.
- The goal is differential expression itself rather than circuit detection — use the fRNC DEGs/edgeR step first; this skill only builds the interaction prior.

## Inputs

- ENCORI (starBase) interaction tables for RBP-miRNA, RBP-lncRNA, miRNA-circRNA and miRNA-lncRNA pairs ('transcriptome'/'Protein' categories, human hg or mouse)
- fRNC IDsymbol identifier-mapping resource (gene symbols to Ensembl, miRNAs to MIMAT* accessions, circRNAs to circBase IDs)
- Differential-expression gene weights (type/logFC/PValue table converted with combinp) that will be joined onto network nodes

## Outputs

- High-stringency RBP-ncRNA edge list with exactly three columns: node_gene_ID, type, target_gene_ID (one row per interaction)
- Identifier-standardized network ready as the 'network' argument of fRNC runmodule (method='global' or 'local')

## How to apply

Install fRNC from GitHub (devtools::install_github('leiming8886/fRNC', ref='main')) after running install_dependpackages for limma, igraph, edgeR and other dependencies. Call interStringency with the appropriate type ('transcription' for transcriptome-level miRNA-lncRNA/miRNA-mRNA(RBP) pairs, 'Protein' for protein-level circuits) and spec ('hg' for human, mouse for APP/PS1 analyses), using stringency='high' so only ENCORI interactions supported by 3 or more CLIP experimental evidences are retained. Standardize identifiers with the package's IDsymbol resource: lncRNA/gene symbols to Ensembl gene IDs, miRNAs to mature miRBase accession numbers (MIMAT*), and circRNAs to circBase IDs. Subset the result to exactly three columns — node_gene_ID, type, target_gene_ID — before passing it to runmodule; this column selection is applied in the README examples (interac_rbp[,c('node_gene_ID','type','target_gene_ID')]). Rationale: the high-stringency CLIP-evidence threshold removes weakly supported edges, and consistent ID mapping ensures differential-expression weights from combinp join correctly onto network nodes.

## Related tools

- **fRNC** (Provides interStringency, IDsymbol mapping, and the three-column edge-list format consumed by runmodule) — https://github.com/BioinformaticsSTU/fRNC
- **ENCORI** (Source of CLIP-evidence-supported RBP-ncRNA interaction tables filtered at high stringency (CLIP evidence >= 3))
- **R** (Runtime environment for the fRNC package and its dependency installation helper install_dependpackages) — https://github.com/BioinformaticsSTU/fRNC

## Examples

```
interac_rbp <- interStringency(type = "transcription", spec = "hg", stringency = "high"); interac_rbp <- interac_rbp[,c("node_gene_ID","type","target_gene_ID")]
```

## Evaluation signals

- The edge list contains exactly three columns named node_gene_ID, type, target_gene_ID with one row per retained interaction.
- Every retained edge is supported by 3 or more CLIP experimental evidences from ENCORI (high stringency).
- miRNA nodes are represented as mature miRBase accession numbers (MIMAT*, e.g. MIMAT0000089) and lncRNA/gene nodes as Ensembl gene IDs; circRNAs as circBase IDs.
- The constructed network is accepted by runmodule without ID-mismatch errors when combined with combinp-derived gene2weight scores.
- For human analyses spec='hg' and the interaction category matches the intended circuit type ('transcription' vs 'Protein' as in the ESCA and BRCA cases).

## Limitations

- Only ENCORI interaction categories and species available in the package ('transcription' and 'Protein' types; hg and mouse) can be used — other data sources are not supported by interStringency.
- High stringency discards interactions with fewer than 3 CLIP experimental evidences, which may remove true but weakly supported edges.
- Network construction is only the prior graph; detected RNCs depend on the quality of downstream differential-expression weights and FDR thresholds (e.g. FDR = 1e-5 for ESCA, 1e-20 for BRCA protein-level).

## Evidence
