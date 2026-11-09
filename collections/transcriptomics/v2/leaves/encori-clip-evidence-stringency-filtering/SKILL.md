---
name: encori-clip-evidence-stringency-filtering
description: Use when constructing an RBP-ncRNA edge list from ENCORI/starBase interactions
  (human or mouse) for downstream RNC module detection and identifiers must be standardized
  and CLIP-evidence filtered.
when_to_use_negative:
- Input interactions are already filtered, identifier-mapped edge lists with node_gene_ID/type/target_gene_ID
  columns — re-running interStringency is unnecessary
- The analysis requires low-stringency or prediction-only (non-CLIP-supported) ENCORI
  interactions, since this skill enforces >= 3 CLIP experimental evidence
- The network source is not ENCORI/CLIP-seq- or PARE-derived (fRNC constructs networks
  from experimentally derived CLIP-seq or PARE data)
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0099
tools:
- name: fRNC
  role: R package providing interStringency to download, stringency-filter and identifier-map
    ENCORI interactions into an RBP-ncRNA edge list
  repo: https://github.com/BioinformaticsSTU/fRNC
- name: R
  role: Runtime environment in which fRNC's interStringency and downstream runmodule
    functions are executed
- name: ENCORI
  role: Source database of CLIP-seq/PARE-supported RBP-ncRNA and miRNA-ncRNA interactions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:6ade801fd8f57953a82dbb67e19c6234bf6bad97165f80744b0153be341b0c73
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
  - fRNC
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# encori-clip-evidence-stringency-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct the fRNC RBP-ncRNA regulatory network by downloading ENCORI (starBase) interaction tables for RBP-miRNA, RBP-lncRNA, miRNA-circRNA and miRNA-lncRNA pairs, filtering to interactions supported by 3 or more CLIP experimental evidence, and standardizing identifiers via Ensembl, miRBase and circBase ID mapping.

## When to use

Use when constructing an RBP-ncRNA edge list from ENCORI/starBase interactions (human or mouse) for downstream RNC module detection and identifiers must be standardized and CLIP-evidence filtered.

## When NOT to use

- Input interactions are already filtered, identifier-mapped edge lists with node_gene_ID/type/target_gene_ID columns — re-running interStringency is unnecessary
- The analysis requires low-stringency or prediction-only (non-CLIP-supported) ENCORI interactions, since this skill enforces >= 3 CLIP experimental evidence
- The network source is not ENCORI/CLIP-seq- or PARE-derived (fRNC constructs networks from experimentally derived CLIP-seq or PARE data)

## Inputs

- ENCORI (starBase) interaction tables for RBP-miRNA, RBP-lncRNA, miRNA-circRNA and miRNA-lncRNA pairs in 'transcriptome' and 'Protein' categories for human (hg) and mouse
- Species specification ('hg' for human) and interaction category/type ('transcription' or 'Protein')
- Stringency level ('high', corresponding to CLIP experimental evidence count >= 3)

## Outputs

- Filtered, identifier-standardized edge list data frame with columns node_gene_ID, type, target_gene_ID (e.g. rbp_ncrna_network_edges.tsv)
- Network object (interac_rbp) suitable as input to fRNC runmodule for global or local RNC detection

## How to apply

Call the fRNC function interStringency with type set to the category of interest (e.g. 'transcription' for transcriptome-level miRNA-lncRNA/miRNA-mRNA pairs, or 'Protein' for RBP-protein-level analysis) and spec set to the species (e.g. 'hg' for human), with stringency='high' so that only ENCORI interactions supported by 3 or more CLIP experimental evidence are retained. During network construction, lncRNA and gene names are mapped to Ensembl gene IDs, miRNA names to miRBase mature-miRNA accession numbers (MIMAT*), and circRNAs to circBase IDs, so seed nodes for local module detection are specified as MIMAT accessions (e.g. MIMAT0000089). Select exactly the three columns node_gene_ID, type and target_gene_ID from the returned data frame to produce the edge list consumed by runmodule. Rationale: the high-stringency CLIP-evidence threshold plus consistent identifier mapping ensures the network is experimentally supported and joinable against differential-expression results from edgeR (DEGs outputs feeding combinp).

## Related tools

- **fRNC** (R package providing interStringency to download, stringency-filter and identifier-map ENCORI interactions into an RBP-ncRNA edge list) — https://github.com/BioinformaticsSTU/fRNC
- **R** (Runtime environment in which fRNC's interStringency and downstream runmodule functions are executed)
- **ENCORI** (Source database of CLIP-seq/PARE-supported RBP-ncRNA and miRNA-ncRNA interactions)

## Examples

```
interac_rbp <- interStringency(type = "transcription", spec = "hg", stringency = "high"); interac_rbp <- interac_rbp[,c("node_gene_ID","type","target_gene_ID")]
```

## Evaluation signals

- Output file rbp_ncrna_network_edges.tsv exists and contains exactly three columns: node_gene_ID, type, target_gene_ID, with one row per retained interaction
- Every retained edge satisfies the high-stringency threshold of >= 3 supporting CLIP experimental evidence in ENCORI
- miRNA nodes are miRBase mature accession numbers (MIMAT*) and lncRNA/gene nodes are Ensembl gene IDs; circRNA nodes use circBase IDs
- The resulting interac_rbp data frame is accepted by runmodule without schema errors for both method='global' and method='local' (e.g. with seed seletN=c('MIMAT0000089'))

## Limitations

- High stringency (CLIP evidence >= 3) discards interactions with weaker support, so rare or tissue-specific interactions may be lost
- Mapping depends on the package's IDsymbol resources for Ensembl, miRBase and circBase identifiers; symbols not present there cannot be standardized
- Species and category coverage is limited to what ENCORI provides (human 'hg' and mouse; 'transcriptome' and 'Protein' categories as used in the examples)

## Evidence
