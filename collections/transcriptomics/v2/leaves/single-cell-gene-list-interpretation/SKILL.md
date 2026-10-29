---
name: single-cell-gene-list-interpretation
description: Use when you have RMT-selected signal and discarded gene lists from a
  single-cell dataset (e.g., Zeisel mouse cortex, Kang PBMC) and need to test whether
  the selected genes correspond to tissue-specific biological processes rather than
  generic ones.
when_to_use_negative:
- You only have raw count matrices and have not yet run Randomly's feature selection
  to obtain selected/discarded gene lists — run the RMT denoising/selection step first.
- The goal is denoising or low-rank representation of the expression matrix itself
  rather than functional interpretation of an already-produced gene list.
- You need de novo pathway discovery rather than over-representation testing against
  fixed GO/Reactome gene sets with a defined background universe.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Randomly
  role: Produces the RMT-selected signal and discarded gene lists that are the input
    to the enrichment analysis
  repo: github.com/RabadanLab/randomly
- name: Python 3
  role: Implementation language for the hypergeometric over-representation test and
    FDR correction
- name: pip
  role: Installation channel for the randomly package used to generate the gene lists
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:aa1820815690339bbc850434324af2725607ff143f06dfbb91939de2a177711c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-gene-list-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets gene lists produced by Randomly's RMT-based feature selection by running hypergeometric over-representation tests (with FDR q values) against GO/Reactome-style gene sets, verifying that selected 'signal' genes enrich for tissue-specific biology (e.g., chemical synaptic transmission, Neural system) while discarded genes enrich for cell-cycle/S-phase terms.

## When to use

Use when you have RMT-selected signal and discarded gene lists from a single-cell dataset (e.g., Zeisel mouse cortex, Kang PBMC) and need to test whether the selected genes correspond to tissue-specific biological processes rather than generic ones.

## When NOT to use

- You only have raw count matrices and have not yet run Randomly's feature selection to obtain selected/discarded gene lists — run the RMT denoising/selection step first.
- The goal is denoising or low-rank representation of the expression matrix itself rather than functional interpretation of an already-produced gene list.
- You need de novo pathway discovery rather than over-representation testing against fixed GO/Reactome gene sets with a defined background universe.

## Inputs

- RMT-selected signal gene list(s) from Randomly feature selection (e.g., mouse cortex/Zeisel, PBMC/Kang)
- Non-selected (discarded) gene list(s) from the same Randomly run
- Gene set annotations (GO/Reactome terms, e.g., chemical synaptic transmission, neural system, immune system, cytokine signaling)
- Full gene list of each dataset to serve as the enrichment background universe

## Outputs

- enrichment_results.csv with term, overlap size, set size, p value, and q value for every tested gene set
- Ranked list of enriched terms per dataset with FDR-corrected q values

## How to apply

Load the RMT-selected signal gene lists and the non-selected (discarded) gene lists produced by Randomly's feature selection for each dataset, and load gene set annotations (GO/Reactome terms such as chemical synaptic transmission, neural system, immune system, cytokine signaling). Define the full gene list of each dataset (e.g., 3,005 mouse cortex cells from Zeisel or 6,573 human PBMCs from Kang) as the enrichment background universe. For each gene set, compute a hypergeometric over-representation test of selected versus non-selected genes against that universe, then convert p values to q values with a multiple-testing FDR correction across all tested gene sets. Compare reproduced q values against the reported anchors — chemical synaptic transmission 1.4e-23 and Neural system 2.8e-23 (mouse cortex); immune system 1.9e-32 and cytokine signaling 2.0e-21 (PBMC) — and confirm that S-phase/cell-cycle terms rank highest among non-selected genes. Write term, overlap size, set size, p value, and q value for every tested gene set to enrichment_results.csv.

## Related tools

- **Randomly** (Produces the RMT-selected signal and discarded gene lists that are the input to the enrichment analysis) — github.com/RabadanLab/randomly
- **Python 3** (Implementation language for the hypergeometric over-representation test and FDR correction)
- **pip** (Installation channel for the randomly package used to generate the gene lists)

## Evaluation signals

- Reproduced q value for 'transmission across chemical synapses' matches 1.4e-23 in mouse cortex within numerical tolerance
- Reproduced q value for 'Neural system' matches 2.8e-23 in mouse cortex within numerical tolerance
- Reproduced q values for 'immune system' (1.9e-32) and 'cytokine signaling' (2.0e-21) match the PBMC reported values within tolerance
- Cell-cycle/S-phase terms rank highest among the non-selected (discarded) genes
- enrichment_results.csv contains term, overlap size, set size, p value, and q value for every tested gene set

## Limitations

- Only the last ~2% of information in single-cell data reflects true biological signal, so enrichment results depend on Randomly's separation of noise, sparsity-induced signal, and biological signal.
- No version or release information is provided for the publicly available algorithm code, complicating exact reproduction.
- No datasets or data accessions are named in the discussion section, so input gene lists must be reconstructed from the cited sources (Zeisel, Kang).

## Evidence

- [other] Hypergeometric enrichment of RMT-selected genes in mouse cortex: [evidence span withheld — anchor status: not_found]
