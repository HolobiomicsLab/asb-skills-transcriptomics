---
name: gene-ontology-enrichment-analysis-via-toppfun
description: Use when a subclone-associated gene list (e.g. GATA2R361C subclone genes)
  overlaps a co-expression module (e.g. TCGA VIM-correlated genes, 2191 genes at q
  < 0.001) and the intersecting genes need biological interpretation via GO terms.
when_to_use_negative:
- The gene lists have not yet been reduced to a significant overlap — ToppFun should
  interpret the ~198-gene intersection, not the raw 2191-gene VIM-correlated set or
  the full subclone list.
- The upstream hypergeometric overlap test does not reproduce significance (p = 3.5e-95),
  in which case the VIM regulon interpretation is unsupported.
- The analysis targets variants or barcodes rather than gene-set biology — use cb_sniffer
  or VarTrix instead.
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ToppFun
  role: Web tool used to perform GO enrichment analysis on the ~198-gene VIM/GATA2R361C
    intersection, recovering immune response, cytoskeletal organization, and focal
    adhesion terms
- name: python3 (scipy.stats)
  role: Scripted hypergeometric testing of the overlap between the VIM-correlated
    gene set and the subclone gene list prior to ToppFun submission
- name: pysam
  role: Listed among the python3-based tooling available in the sridnona/python3:180925.v1
    Docker image used for the analysis environment
- name: Docker (sridnona/python3:180925.v1)
  role: Container image providing the python3 environment in which the overlap computation
    is run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:32b92c55be2f9ff372718e9f7c39a62aaafd79408970f8b4f072fa5a8994e877
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - Docker (sridnona/python3:180925.v1)
  - ToppFun
  - pysam
  - python3
  - python3 (scipy.stats)
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Gene ontology enrichment analysis via ToppFun

## Summary

Interpret a statistically significant gene-set overlap — the ~198 genes shared between TCGA VIM-correlated genes (q < 0.001) and GATA2R361C subclone-associated genes (hypergeometric p = 3.5e-95) — by submitting the intersection to ToppFun for GO enrichment across immune response, cytoskeletal organization, and focal adhesion categories.

## When to use

Use when a subclone-associated gene list (e.g. GATA2R361C subclone genes) overlaps a co-expression module (e.g. TCGA VIM-correlated genes, 2191 genes at q < 0.001) and the intersecting genes need biological interpretation via GO terms.

## When NOT to use

- The gene lists have not yet been reduced to a significant overlap — ToppFun should interpret the ~198-gene intersection, not the raw 2191-gene VIM-correlated set or the full subclone list.
- The upstream hypergeometric overlap test does not reproduce significance (p = 3.5e-95), in which case the VIM regulon interpretation is unsupported.
- The analysis targets variants or barcodes rather than gene-set biology — use cb_sniffer or VarTrix instead.

## Inputs

- TCGA AML expression matrix
- GATA2R361C subclone-associated gene list (from task_006)
- VIM-correlated gene set (2191 genes, q < 0.001 after Benjamini-Hochberg correction)

## Outputs

- vim_gata2_overlap_results.tsv containing the intersection gene list, hypergeometric p-value, and ToppFun GO terms
- List of ~198 overlapping genes
- GO enrichment terms for immune response, cytoskeletal organization, and focal adhesion

## How to apply

First compute the Pearson correlation of every gene with VIM across all TCGA AML samples, apply Benjamini-Hochberg correction, and retain the 2191 genes with q < 0.001. In a python3 script, use scipy.stats hypergeometric testing to test overlap between this VIM-correlated set and the GATA2R361C subclone gene list from task_006; accept the analysis if the intersection is ~198 genes with p = 3.5e-95. Then submit the ~198 overlapping genes to ToppFun for GO enrichment analysis, checking that terms covering immune response, cytoskeletal organization, and focal adhesion are recovered. Write the intersection gene list, the hypergeometric p-value, and the ToppFun GO terms to vim_gata2_overlap_results.tsv.

## Related tools

- **ToppFun** (Web tool used to perform GO enrichment analysis on the ~198-gene VIM/GATA2R361C intersection, recovering immune response, cytoskeletal organization, and focal adhesion terms)
- **python3 (scipy.stats)** (Scripted hypergeometric testing of the overlap between the VIM-correlated gene set and the subclone gene list prior to ToppFun submission)
- **pysam** (Listed among the python3-based tooling available in the sridnona/python3:180925.v1 Docker image used for the analysis environment)
- **Docker (sridnona/python3:180925.v1)** (Container image providing the python3 environment in which the overlap computation is run)

## Evaluation signals

- The reported intersection between TCGA VIM-correlated genes and GATA2R361C subclone genes contains ~198 genes.
- The scipy.stats hypergeometric overlap test reproduces p = 3.5e-95.
- ToppFun recovers GO terms for immune response, cytoskeletal organization, and focal adhesion.
- The VIM-correlated gene set contains 2191 genes at q < 0.001 after Benjamini-Hochberg correction.
- vim_gata2_overlap_results.tsv is written with the intersection gene list, p-value, and GO terms.

## Limitations

- The overlap p-value depends on the upstream gene-set definitions (the 2191-gene q < 0.001 VIM set and the task_006 subclone gene list); changes to those sets change the statistics.
- ToppFun is an external web service, so results depend on its current GO annotation database and cannot be fully reproduced offline from the article alone.
- Interpretation of the VIM regulon rests on correlation with a single gene (VIM) in bulk TCGA AML data, not on direct single-cell evidence.

## Evidence

- [other] yielding an intersection of ~198 genes with p = 3.5e-95: [evidence span withheld — anchor status: not_found]
- [other] ToppFun recovering GO terms for immune response, cytoskeletal organization, and focal adhesion: [evidence span withheld — anchor status: not_found]
- [abstract] identify expression signatures associated with subclonal mutations: "identify expression signatures associated with subclonal mutations, and find cell surface markers that could be used to purify subclones for further study"
- [methods] read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv: "read counts for Chromium molecular barcode(UB_tag): _counts_UB.tsv"
