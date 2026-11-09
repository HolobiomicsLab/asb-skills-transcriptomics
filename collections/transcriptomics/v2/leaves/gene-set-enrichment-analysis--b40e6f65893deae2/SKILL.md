---
name: gene-set-enrichment-analysis
description: Use when you have a selected gene list (e.g. RMT signal genes from Zeisel
  mouse cortex or Kang PBMC data) plus a discarded list, and need to test over-representation
  of biological gene sets against the full gene universe.
when_to_use_negative:
- You only have raw single-cell expression matrices and have not yet run the RMT feature-selection
  step that separates selected signal genes from discarded genes
- Your question is about differential expression between cell types rather than over-representation
  of a fixed gene list against a defined background
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0659
tools:
- name: Randomly
  role: Produces the selected signal-gene and discarded gene lists that feed the hypergeometric
    enrichment test
  repo: github.com/RabadanLab/randomly
- name: Python 3
  role: Language in which the hypergeometric tests, FDR correction, and CSV export
    are implemented
- name: pip
  role: Used to install the Randomly package that generates the input gene lists
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:24f9317ff7123e96f458ba85f8106014c9e96b8c5d19da4293f66bb0cb0d5e9c
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
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gene-set-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Hypergeometric over-representation analysis that tests whether genes selected by the Randomly RMT feature-selection step enrich for tissue- or phenotype-specific gene sets (GO/Reactome terms) rather than generic ones, reported as p values converted to FDR q values.

## When to use

Use when you have a selected gene list (e.g. RMT signal genes from Zeisel mouse cortex or Kang PBMC data) plus a discarded list, and need to test over-representation of biological gene sets against the full gene universe.

## When NOT to use

- You only have raw single-cell expression matrices and have not yet run the RMT feature-selection step that separates selected signal genes from discarded genes
- Your question is about differential expression between cell types rather than over-representation of a fixed gene list against a defined background

## Inputs

- RMT-selected signal gene lists from the Randomly feature-selection step (mouse cortex Zeisel and PBMC Kang datasets)
- Non-selected (discarded) gene lists from the same Randomly step
- Gene set annotations (GO/Reactome terms such as chemical synaptic transmission, neural system, immune system, cytokine signaling)
- Full gene list of each dataset (used as the hypergeometric background universe)

## Outputs

- enrichment_results.csv containing term, overlap size, set size, p value, and q value for every tested gene set
- Ranked list of enriched gene sets with FDR-corrected q values (e.g. synapses 1.4e-23, neural system 2.8e-23 for mouse cortex; immune system 1.9e-32, cytokine signaling 2.0e-21 for PBMC)

## How to apply

Load the RMT-selected signal gene lists and the non-selected (discarded) gene lists produced by the Randomly feature-selection step for the mouse cortex (Zeisel, 3,005 cells) and PBMC (Kang, 6,573 cells) datasets. Load gene set annotations (e.g. GO/Reactome terms such as chemical synaptic transmission, neural system, immune system, and cytokine signaling) and define the full gene list of each dataset as the enrichment background. For each gene set, run a hypergeometric over-representation test of selected versus non-selected genes against this background population, then convert p values to q values with a multiple-testing FDR correction across all tested gene sets. Finally, write the term, overlap size, set size, p value, and q value for every tested gene set to enrichment_results.csv, and check that cell-cycle/S-phase terms dominate the non-selected genes rather than the selected ones.

## Related tools

- **Randomly** (Produces the selected signal-gene and discarded gene lists that feed the hypergeometric enrichment test) — github.com/RabadanLab/randomly
- **Python 3** (Language in which the hypergeometric tests, FDR correction, and CSV export are implemented)
- **pip** (Used to install the Randomly package that generates the input gene lists)

## Examples

```
from scipy.stats import hypergeom; q = fdr([hypergeom.sf(k-1, M, n, N) for k,n,M,N in overlaps]); df.to_csv('enrichment_results.csv')
```

## Evaluation signals

- Reproduced q value for transmission across chemical synapses in mouse cortex matches 1.4e-23 within numerical tolerance
- Reproduced q value for Neural system in mouse cortex matches 2.8e-23 within numerical tolerance
- Reproduced q values for PBMC match immune system 1.9e-32 and cytokine signaling 2.0e-21 within tolerance
- Cell-cycle/S-phase terms rank highest among the non-selected (discarded) genes, not among the RMT-selected signal genes
- enrichment_results.csv contains term, overlap size, set size, p value, and q value for every tested gene set

## Limitations

- q values depend on defining the full dataset gene list as the background; a different universe changes the hypergeometric p and q values
- The expected q values are dataset-specific (Zeisel mouse cortex and Kang PBMC); other datasets will not reproduce them
- Randomly (which generates the input gene lists) only works with Python 3, not Python 2

## Evidence

- [results] RMT-selected signal genes enrich for tissue-specific processes: [evidence span withheld — anchor status: not_found]
