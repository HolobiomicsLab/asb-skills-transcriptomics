---
name: hypergeometric-overrepresentation-testing
description: Use when you have RMT-selected signal gene lists and discarded gene lists
  from a single-cell dataset (e.g. mouse cortex Zeisel or PBMC Kang) and need to confirm
  the retained genes enrich for tissue-specific pathways rather than generic or sparsity-driven
  ones.
when_to_use_negative:
- The gene lists have not been produced by the RMT feature selection step — enrichment
  of arbitrary gene lists does not test the paper's claim that signal genes are tissue-specific
- You need differential expression or ranked GSEA (e.g. Kolmogorov-Smirnov on expression
  ranks) rather than over-representation of a pre-defined selected/disclosed gene
  split
- The dataset's full gene universe cannot be defined (no complete gene list to serve
  as the hypergeometric background population)
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Randomly
  role: Produces the selected signal gene and discarded gene lists whose enrichment
    is tested (RMT denoising/feature selection library)
  repo: github.com/luisaparicio/randomly
- name: Python 3
  role: Implementation language for the hypergeometric test and FDR correction (Randomly
    only works with Python 3)
- name: pip
  role: Installs the randomly package used upstream of the enrichment analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:601685e906c9084334c78cc8e22054c6ae21433e65e99181f7be0c163df3a944
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
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Hypergeometric overrepresentation testing of RMT-selected genes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that genes selected by the Randomly RMT feature-selection algorithm carry dataset-specific biology by running hypergeometric gene set enrichment (GO/Reactome terms) of selected versus non-selected genes, with FDR-corrected q values checked against the paper's reported enrichments.

## When to use

Use when you have RMT-selected signal gene lists and discarded gene lists from a single-cell dataset (e.g. mouse cortex Zeisel or PBMC Kang) and need to confirm the retained genes enrich for tissue-specific pathways rather than generic or sparsity-driven ones.

## When NOT to use

- The gene lists have not been produced by the RMT feature selection step — enrichment of arbitrary gene lists does not test the paper's claim that signal genes are tissue-specific
- You need differential expression or ranked GSEA (e.g. Kolmogorov-Smirnov on expression ranks) rather than over-representation of a pre-defined selected/disclosed gene split
- The dataset's full gene universe cannot be defined (no complete gene list to serve as the hypergeometric background population)

## Inputs

- RMT-selected signal gene lists (per dataset) from the Randomly feature selection step
- Non-selected (discarded) gene lists (per dataset) from the Randomly feature selection step
- Gene set annotations (GO/Reactome terms, e.g. chemical synaptic transmission, neural system, immune system, cytokine signaling)
- Full gene list of each dataset used as the enrichment background universe

## Outputs

- enrichment_results.csv with term, overlap size, set size, p value, and q value for every tested gene set
- Ranked FDR-corrected q values per gene set for selected and non-selected genes

## How to apply

Load the RMT-selected signal genes and the non-selected (discarded) genes for each dataset (mouse cortex Zeisel, 3,005 cells; PBMC Kang, 6,573 cells) produced by the Randomly feature selection step, which discards genes responsible for localized eigenvectors in the randomized data. Define the full gene list of each dataset as the enrichment background universe, and gather gene set annotations such as chemical synaptic transmission, neural system, immune system, and cytokine signaling. For each gene set, compute a hypergeometric over-representation test of selected versus non-selected genes against this universe, then convert p values to q values with a multiple-testing FDR correction across all tested gene sets. Validate against the paper's reported values — transmission across chemical synapses q = 1.4e-23 and Neural system q = 2.8e-23 for mouse cortex; immune system q = 1.9e-32 and cytokine signaling q = 2.0e-21 for PBMC — and confirm that S-phase/cell-cycle terms dominate the enrichment of the non-selected genes. Write term, overlap size, set size, p value, and q value for every tested gene set to enrichment_results.csv.

## Related tools

- **Randomly** (Produces the selected signal gene and discarded gene lists whose enrichment is tested (RMT denoising/feature selection library)) — github.com/luisaparicio/randomly
- **Python 3** (Implementation language for the hypergeometric test and FDR correction (Randomly only works with Python 3))
- **pip** (Installs the randomly package used upstream of the enrichment analysis)

## Examples

```
pip install randomly; import randomly  # then run hypergeometric ORA of selected vs discarded genes with the dataset's full gene list as background
```

## Evaluation signals

- Reproduced q value for transmission across chemical synapses matches 1.4e-23 (mouse cortex) within reasonable numerical tolerance
- Reproduced q value for Neural system matches 2.8e-23 (mouse cortex) within tolerance
- Reproduced q values for immune system (1.9e-32) and cytokine signaling (2.0e-21) match the PBMC (Kang) reported values
- S-phase/cell-cycle terms rank highest among the non-selected (discarded) genes, not among signal genes
- enrichment_results.csv contains term, overlap size, set size, p value, and q value for every tested gene set

## Limitations

- Randomly only works with Python 3 (not Python 2)
- The paper provides no version or release information for the publicly available algorithm code
- The enrichment validation depends on reproducing the exact RMT-selected and discarded gene lists from the upstream Randomly step; deviations there propagate to the q values

## Evidence

- [results] Hypergeometric gene set enrichment of RMT-selected signal genes in the mouse cortex dataset shows the top pathways correspond to specific brain functions: [evidence span withheld — anchor status: not_found]
