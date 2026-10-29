---
name: custom-geneset-gmt-construction
description: Use when a custom gene signature (e.g., a 22-gene glycolysis or 14-gene
  hypoxia set) must be formatted as a .gmt file so ARMT/GSVA can compute gene-set
  scores from a normalized expression matrix.
when_to_use_negative:
- The gene sets are already available in .gmt format or as one of the 9 MSigDb genesets
  in ARMT's internal datasets — use them directly in the Normalization&GSVA page instead.
- The gene list is in Ensembl ID form and has not yet been converted to symbols (ARMT
  transfers Ensembl IDs to Symbol during normalization, but the geneset .gmt should
  reference symbols).
- The goal is gene-level differential expression (edgeR/limma DEG) rather than gene-set-level
  GSVA scoring, which does not require a .gmt file.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ARMT
  role: Provides the 'Data' page GUI that converts a two-column .csv geneset table
    into a .gmt file usable for GSVA
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: 'Downstream consumer: the constructed .gmt file is the geneset input format
    for GSVA score computation in ARMT'
  repo: https://github.com/Dulab2020/ARMT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:31c8b195d4e6cb29c88c24c78beb8ef2a165b83c8e295cdf0fe1c40bf5d8496f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Dulab2020/ARMT
  repo_url: https://github.com/Dulab2020/ARMT
  tools:
  - ARMT
  - GSVA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# custom-geneset-gmt-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs Gene Matrix Transposed (.gmt) files for arbitrary custom gene signatures (e.g., the paper's 22-gene glycolysis and 14-gene hypoxia signatures) using ARMT's 'Data' page, so the sets can be scored by GSVA. This enables downstream gene-set analysis beyond the 9 built-in MSigDb sets shipped with ARMT.

## When to use

Use when a custom gene signature (e.g., a 22-gene glycolysis or 14-gene hypoxia set) must be formatted as a .gmt file so ARMT/GSVA can compute gene-set scores from a normalized expression matrix.

## When NOT to use

- The gene sets are already available in .gmt format or as one of the 9 MSigDb genesets in ARMT's internal datasets — use them directly in the Normalization&GSVA page instead.
- The gene list is in Ensembl ID form and has not yet been converted to symbols (ARMT transfers Ensembl IDs to Symbol during normalization, but the geneset .gmt should reference symbols).
- The goal is gene-level differential expression (edgeR/limma DEG) rather than gene-set-level GSVA scoring, which does not require a .gmt file.

## Inputs

- Two-column .csv file: column 1 = geneset names, column 2 = gene symbols per geneset
- Custom gene signatures (e.g., 22-gene glycolysis set, 14-gene hypoxia set) as gene symbol lists

## Outputs

- .gmt file containing named genesets with their member genes (main geneset input format for ARMT GSVA)

## How to apply

Assemble the signature as a two-column .csv where the first column holds the geneset names and the second column holds the genes belonging to each geneset, using gene symbols consistent with ARMT's symbol-based TPM output. Load this .csv in ARMT's 'Data' page to generate the .gmt file, which is the main geneset input format for the 'Normalization&GSVA' page. Follow the paper's example structure: build separate named sets such as the 22-gene glycolysis signature (SLC2A1, HK1, HK2, HK3, GPI, PFKL, PFKM, PFKP, ALDOA, ALDOB, ALDOC, TPI1, GAPDH, PGK1, PGAM1, PGAM4, ENO1, ENO2, ENO3, PKLR, PKM, LDHA) and the 14-gene hypoxia signature (ALDOA, MIF, TUBB6, P4HA1, SLC2A1, PGAM1, ENO1, LDHA, CDKN3, TPI1, NDRG1, VEGFA, ACOT7, ADM), noting genes may belong to multiple sets. The resulting .gmt can then be selected in the Normalization&GSVA page instead of one of the 9 internal MSigDb genesets to compute GSVA scores.

## Related tools

- **ARMT** (Provides the 'Data' page GUI that converts a two-column .csv geneset table into a .gmt file usable for GSVA) — https://github.com/Dulab2020/ARMT
- **GSVA** (Downstream consumer: the constructed .gmt file is the geneset input format for GSVA score computation in ARMT) — https://github.com/Dulab2020/ARMT

## Examples

```
devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()  # use the GUI 'Data' page to upload the two-column geneset .csv and create the .gmt
```

## Evaluation signals

- The source .csv has exactly two columns: geneset names in the first, member genes in the second (per README Figure2 format).
- The generated .gmt parses into named sets with correct membership — e.g., the glycolysis set resolves to 22 genes and the hypoxia set to 14 genes.
- The .gmt loads successfully in ARMT's 'Normalization&GSVA' page and yields a GSVA score matrix (samples x genesets) exported as .csv.
- Gene identifiers in the .gmt are symbols matching the normalized TPM matrix's gene symbol rows.
- Downstream GSVA-based analyses (e.g., hypoxia-high vs -low differential analysis of glycolysis score) run without geneset-matching errors.

## Limitations

- Geneset creation requires the specific two-column .csv layout; other table formats will not map correctly to the .gmt builder.
- README truncation omits the full GSVA-score-matrix input specification, so edge cases in geneset/GSVA data exchange are not fully documented.
- No software or ARMT version numbers are stated, so reproducibility of the exact geneset-to-.gmt behavior may depend on the repo state at the time of use.

## Evidence
