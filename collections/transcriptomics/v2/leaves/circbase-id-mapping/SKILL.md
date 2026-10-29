---
name: circbase-id-mapping
description: Use when building RBP-ncRNA edge lists from ENCORI/starBase interaction
  tables and node identifiers must be normalized to Ensembl gene IDs, miRBase MIMAT*
  accessions, or circBase IDs before module detection.
when_to_use_negative:
- Expression matrices are already indexed by Ensembl gene IDs or MIMAT* accessions
  and no interaction table needs remapping
- The interaction source is not ENCORI/starBase-derived CLIP-seq or PARE data, so
  the package's IDsymbol mappings and high-stringency filtering do not apply
- The goal is circRNA discovery/annotation rather than standardizing node IDs for
  RBP-ncRNA network construction
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: fRNC
  role: R package providing interStringency and the IDsymbol resource used to filter
    ENCORI interactions and map symbols to Ensembl IDs, miRBase MIMAT* accessions
    and circBase IDs
  repo: https://github.com/BioinformaticsSTU/fRNC
- name: ENCORI
  role: Source database of experimentally derived (CLIP-seq/PARE) RBP-ncRNA and miRNA-ncRNA
    interaction tables whose identifiers are standardized
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:113b6f4c5dcea5b0280e5cbb2ff708e88a104fdbb7c270110c8df25d46897a26
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# circbase-id-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Standardizes node identifiers when constructing the fRNC RBP-ncRNA network from ENCORI interactions: lncRNA/gene symbols are mapped to Ensembl gene IDs, mature miRNAs to miRBase accessions (MIMAT*), and circRNAs to circBase IDs.

## When to use

Use when building RBP-ncRNA edge lists from ENCORI/starBase interaction tables and node identifiers must be normalized to Ensembl gene IDs, miRBase MIMAT* accessions, or circBase IDs before module detection.

## When NOT to use

- Expression matrices are already indexed by Ensembl gene IDs or MIMAT* accessions and no interaction table needs remapping
- The interaction source is not ENCORI/starBase-derived CLIP-seq or PARE data, so the package's IDsymbol mappings and high-stringency filtering do not apply
- The goal is circRNA discovery/annotation rather than standardizing node IDs for RBP-ncRNA network construction

## Inputs

- ENCORI/starBase interaction tables (RBP-miRNA, RBP-lncRNA, miRNA-circRNA, miRNA-lncRNA) in 'transcriptome' and 'Protein' categories for human (hg) and mouse
- lncRNA/gene symbols, mature miRNA names, and circRNA identifiers to be standardized
- fRNC package IDsymbol ID-mapping resource

## Outputs

- Edge list with exactly three columns: node_gene_ID, type, target_gene_ID (e.g. rbp_ncrna_network_edges.tsv), with miRNAs as MIMAT* accessions, genes/lncRNAs as Ensembl gene IDs, and circRNAs as circBase IDs

## How to apply

Download the ENCORI (starBase) interaction tables for RBP-miRNA, RBP-lncRNA, miRNA-circRNA and miRNA-lncRNA pairs in the 'transcription'/'transcriptome' and 'Protein' categories for human ('hg') and mouse, then apply the fRNC interStringency call with stringency='high' so only CLIP-supported interactions with evidence count >= 3 are retained. Map identifiers using the package's IDsymbol resource: lncRNA/gene symbols to Ensembl gene IDs, miRNAs to mature-miRNA miRBase accession numbers (MIMAT*, e.g. the seed node MIMAT0000089), and circRNAs to circBase IDs. Subset the mapped table to the three columns node_gene_ID, type and target_gene_ID to produce the edge list consumed by runmodule. The rationale is that scoring and module search operate on a single consistent identifier space across miRNA, mRNA, lncRNA and protein node types.

## Related tools

- **fRNC** (R package providing interStringency and the IDsymbol resource used to filter ENCORI interactions and map symbols to Ensembl IDs, miRBase MIMAT* accessions and circBase IDs) — https://github.com/BioinformaticsSTU/fRNC
- **ENCORI** (Source database of experimentally derived (CLIP-seq/PARE) RBP-ncRNA and miRNA-ncRNA interaction tables whose identifiers are standardized)

## Examples

```
interac_rbp <- interStringency(type = "transcription", spec = "hg", stringency = "high"); interac_rbp <- interac_rbp[,c("node_gene_ID","type","target_gene_ID")]
```

## Evaluation signals

- Edge file exists and contains exactly the three columns node_gene_ID, type and target_gene_ID with one row per retained interaction
- Every retained edge satisfies the high-stringency threshold: number of supported CLIP experimental evidence >= 3
- All miRNA nodes match the miRBase mature accession pattern (MIMAT*, e.g. MIMAT0000089 used as a runmodule seed node)
- lncRNA/gene nodes are Ensembl gene IDs and circRNA nodes are circBase IDs, with no residual unmapped symbols
- Mapped, filtered edge list loads directly into runmodule as the 'network' argument without further identifier manipulation

## Limitations

- The high-stringency setting (CLIP evidence >= 3) discards interactions with weaker support, reducing network coverage
- Identifier mapping depends on the fRNC package's bundled IDsymbol resource; symbols absent from that resource cannot be standardized
- Interactions and mappings are species-specific and must be requested with the correct spec (e.g. 'hg') and category ('transcription'/'Protein')

## Evidence

- [other] Map the identifiers using the package's IDsymbol resource: [evidence span withheld — anchor status: not_found]
- [other] During the construction of the regulatory network, lncRNA and gene names are mapped to Ensembl IDs: [evidence span withheld — anchor status: not_found]
- [readme] ENCORI high stringency filtering: "were extracted from ENCORI with a high stringency, where the number of the supported CLIP experimental evidence is 3 or greater."
