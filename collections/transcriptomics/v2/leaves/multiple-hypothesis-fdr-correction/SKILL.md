---
name: multiple-hypothesis-fdr-correction
description: Use when you have two disjoint gene lists (algorithm-selected vs discarded
  genes) plus gene set annotations and need over-representation p values converted
  to FDR q values to validate that selection enriches specific biology.
when_to_use_negative:
- You need enrichment of a single pre-specified hypothesis gene set without multiple-testing
  burden — an uncorrected hypergeometric p value suffices and FDR correction is unnecessary
- You lack a defined background universe (full gene list of the dataset), making hypergeometric
  over-representation testing invalid
- Your goal is denoising or feature selection itself (Randomly's RMT eigenvalue/eigenvector
  analysis) rather than downstream validation of selected genes
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: Randomly
  role: Produces the RMT-selected signal gene lists and discarded gene lists that
    are the tested and background inputs to the hypergeometric enrichment
  repo: github.com/RabadanLab/randomly
- name: Python 3
  role: Implementation language for the hypergeometric test, FDR correction, and writing
    enrichment_results.csv
- name: pip
  role: Installs the Randomly library and Python dependencies used to run the enrichment
    workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:e5b2c144a8038234bcba89fb74a6884e5b12ee18e4f419cf80e3ca316d1b8b91
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/RabadanLab/randomly
  tools:
  - Python 3
  - Randomly
  - pip
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# multiple-hypothesis-fdr-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Hypergeometric over-representation testing of RMT-selected signal genes against gene set annotations (GO/Reactome-style terms), with Benjamini-Hochberg-style FDR q-value correction across all tested gene sets, to show that Randomly-selected genes enrich for tissue-specific processes (e.g. chemical synaptic transmission, Neural system in mouse cortex) rather than generic ones.

## When to use

Use when you have two disjoint gene lists (algorithm-selected vs discarded genes) plus gene set annotations and need over-representation p values converted to FDR q values to validate that selection enriches specific biology.

## When NOT to use

- You need enrichment of a single pre-specified hypothesis gene set without multiple-testing burden — an uncorrected hypergeometric p value suffices and FDR correction is unnecessary
- You lack a defined background universe (full gene list of the dataset), making hypergeometric over-representation testing invalid
- Your goal is denoising or feature selection itself (Randomly's RMT eigenvalue/eigenvector analysis) rather than downstream validation of selected genes

## Inputs

- RMT-selected signal gene list per dataset (mouse cortex Zeisel; PBMC Kang) from the Randomly feature selection step
- Non-selected (discarded) gene list per dataset from the same step
- Gene set annotations (GO/Reactome terms such as chemical synaptic transmission, Neural system, immune system, cytokine signaling)
- Full gene list of each dataset, used as the enrichment background universe

## Outputs

- enrichment_results.csv with term, overlap size, set size, p value, and q value for every tested gene set
- FDR-corrected q values for named terms (synapses 1.4e-23, Neural system 2.8e-23, immune system 1.9e-32, cytokine signaling 2.0e-21)

## How to apply

Load the RMT-selected signal gene lists and the non-selected (discarded) gene lists produced by the Randomly feature selection step for each dataset (mouse cortex/Zeisel, 3,005 cells; PBMC/Kang, 6,573 cells). Load gene set annotations (e.g. chemical synaptic transmission, Neural system, immune system, cytokine signaling) and define the full gene list of each dataset as the enrichment background/universe. For each gene set, compute a hypergeometric over-representation test of selected versus non-selected genes against that universe. Convert all enrichment p values to q values with a multiple-testing FDR correction across all tested gene sets, then compare against reported q values: transmission across chemical synapses 1.4e-23 and Neural system 2.8e-23 for mouse cortex; immune system 1.9e-32 and cytokine signaling 2.0e-21 for PBMC. Also confirm that S-phase/cell-cycle terms rank highest among the non-selected (discarded) genes, and write term, overlap size, set size, p value, and q value for every tested gene set to enrichment_results.csv.

## Related tools

- **Randomly** (Produces the RMT-selected signal gene lists and discarded gene lists that are the tested and background inputs to the hypergeometric enrichment) — github.com/RabadanLab/randomly
- **Python 3** (Implementation language for the hypergeometric test, FDR correction, and writing enrichment_results.csv)
- **pip** (Installs the Randomly library and Python dependencies used to run the enrichment workflow)

## Examples

```
from scipy.stats import hypergeom; from statsmodels.stats.multitest import multipletests; p = hypergeom.sf(k-1, M, n, N); q = multipletests(p, method='fdr_bh')[1]
```

## Evaluation signals

- Reproduced q values match reported values within numerical tolerance: transmission across chemical synapses 1.4e-23 and Neural system 2.8e-23 (mouse cortex); immune system 1.9e-32 and cytokine signaling 2.0e-21 (PBMC)
- Top enriched terms for RMT-selected signal genes correspond to tissue-specific brain functions, not generic terms
- S-phase/cell-cycle terms rank highest among the non-selected (discarded) genes
- enrichment_results.csv contains term, overlap size, set size, p value, and q value for every tested gene set
- All q values are monotonically consistent with p values and bounded in [0, 1] after FDR correction

## Limitations

- Results depend on the correctness of the upstream Randomly feature selection; if selected/discarded lists are wrong, enrichment conclusions do not transfer
- The enrichment background must be the full gene list of each dataset; using a different universe changes q values
- No version or release information is provided for the publicly available algorithm code, complicating exact reproduction

## Evidence

- [results] Hypergeometric gene set enrichment of RMT-selected signal genes in the mouse cortex dataset shows the top pathways correspond to specific brain functions: [evidence span withheld — anchor status: not_found]
- [methods] Convert enrichment p values to q values with a multiple-testing FDR correction across all tested gene sets.: [evidence span withheld — anchor status: not_found]
- [methods] compute a hypergeometric over-representation test of selected versus non-selected genes using the full gene universe as the background population: [evidence span withheld — anchor status: not_found]
