---
name: mirbase-accession-mapping
description: Use when building an RBP-ncRNA edge list (e.g. from ENCORI/starBase CLIP
  data for human or mouse) whose nodes mix gene/lncRNA symbols, mature miRNA names,
  and circRNA IDs and must be converted to Ensembl, MIMAT*, and circBase identifiers.
when_to_use_negative:
- Expression or differential-analysis data already keyed by Ensembl/MIMAT identifiers
  where no symbol-to-accession mapping is needed (e.g. the pre-processed brca_miRNA_re_se
  / brca_RNA_re_se internal fRNC datasets)
- Interaction sources other than ENCORI/starBase that use different identifier vocabularies
  incompatible with the package's ID mapping resources
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0080
tools:
- name: fRNC
  role: R package whose interStringency function retrieves ENCORI interactions at
    a chosen stringency and whose identifier mapping resources convert symbols to
    Ensembl IDs, miRNAs to miRBase MIMAT* accessions, and circRNAs to circBase IDs
  repo: https://github.com/leiming8886/fRNC
- name: R
  role: Runtime environment in which the fRNC mapping and network-construction functions
    are executed
- name: ENCORI
  role: Source database of CLIP-seq-derived RBP-ncRNA interactions whose native identifiers
    are the input to the mapping step
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:5db8b520430c7fdd7debb6857bda13e5fa45e344faafe69c1d75ca9f5dc744d3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/fRNC
  repo_url: https://github.com/leiming8886/fRNC
  tools:
  - ENCORI
  - R
  - fRNC
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# mirbase-accession-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Standardizes identifiers when assembling the fRNC RBP-ncRNA network from ENCORI-derived interactions: lncRNA/gene symbols are mapped to Ensembl gene IDs, miRNA names to miRBase accession numbers of mature miRNAs (MIMAT*), and circRNAs to circBase IDs, so network edges use consistent, resolvable node identifiers.

## When to use

Use when building an RBP-ncRNA edge list (e.g. from ENCORI/starBase CLIP data for human or mouse) whose nodes mix gene/lncRNA symbols, mature miRNA names, and circRNA IDs and must be converted to Ensembl, MIMAT*, and circBase identifiers.

## When NOT to use

- Expression or differential-analysis data already keyed by Ensembl/MIMAT identifiers where no symbol-to-accession mapping is needed (e.g. the pre-processed brca_miRNA_re_se / brca_RNA_re_se internal fRNC datasets)
- Interaction sources other than ENCORI/starBase that use different identifier vocabularies incompatible with the package's ID mapping resources

## Inputs

- ENCORI (starBase) interaction tables for RBP-miRNA, RBP-lncRNA, miRNA-circRNA and miRNA-lncRNA pairs ('transcriptome' and 'Protein' categories, human 'hg' and mouse)
- Gene/lncRNA symbols, mature miRNA names, and circRNA identifiers as provided by ENCORI

## Outputs

- Edge list with columns node_gene_ID, type, target_gene_ID where nodes are Ensembl gene IDs (lncRNA/gene), miRBase MIMAT* accessions (mature miRNAs), and circBase IDs (circRNAs), restricted to high-stringency (CLIP evidence >= 3) interactions

## How to apply

Download the ENCORI (starBase) interaction tables for RBP-miRNA, RBP-lncRNA, miRNA-circRNA and miRNA-lncRNA pairs in the 'transcriptome' and 'Protein' categories for the relevant species (e.g. spec='hg' for human), then apply high-stringency filtering so only interactions with CLIP experimental evidence >= 3 are retained via the fRNC interStringency(type, spec, stringency='high') call. Standardize identifiers using the package's ID mapping resources: lncRNA and gene symbols to Ensembl gene IDs, miRNA names to miRBase accession numbers of mature miRNAs (MIMAT* identifiers such as MIMAT0000089), and circRNAs to circBase IDs. Assemble the mapped pairs into an edge list restricted to the columns node_gene_ID, type, and target_gene_ID, matching the column selection applied to interac_rbp in the fRNC workflow. This ensures that downstream runmodule scoring and Cytoscape XGMML export operate on unambiguous node identifiers.

## Related tools

- **fRNC** (R package whose interStringency function retrieves ENCORI interactions at a chosen stringency and whose identifier mapping resources convert symbols to Ensembl IDs, miRNAs to miRBase MIMAT* accessions, and circRNAs to circBase IDs) — https://github.com/leiming8886/fRNC
- **R** (Runtime environment in which the fRNC mapping and network-construction functions are executed)
- **ENCORI** (Source database of CLIP-seq-derived RBP-ncRNA interactions whose native identifiers are the input to the mapping step)

## Examples

```
interac_rbp <- interStringency(type = "transcription", spec = "hg", stringency = "high"); interac_rbp <- interac_rbp[,c("node_gene_ID","type","target_gene_ID")]
```

## Evaluation signals

- Edge list contains exactly the three columns node_gene_ID, type, target_gene_ID with one row per retained interaction
- Every miRNA node identifier matches the miRBase mature-miRNA accession pattern (MIMAT*)
- Every lncRNA/gene node identifier is an Ensembl gene ID and every circRNA node is a circBase ID
- Every retained edge satisfies the high-stringency threshold: at least 3 supporting CLIP experimental evidence records in ENCORI
- Mapped identifiers are consistent with the species argument used (e.g. spec='hg' for human interactions)

## Limitations

- Mapping applies to ENCORI-derived interactions; identifiers from other interaction databases may not resolve through the package's mapping resources
- Only high-stringency (CLIP evidence >= 3) interactions are retained, so low-evidence pairs are excluded by design
- The article's discussion-section content is absent, so caveats about mapping coverage/completeness (e.g. unmapped symbols) are not documented in the provided context

## Evidence

- [methods] During the construction of the regulatory network, lncRNA and gene names are mapped to Ensembl IDs, and miRNA names are mapped to miRBase accession numbers of mature miRNAs.: [evidence span withheld — anchor status: not_found]
