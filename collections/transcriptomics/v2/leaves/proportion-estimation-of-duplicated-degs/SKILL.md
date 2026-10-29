---
name: proportion-estimation-of-duplicated-degs
description: Use when auditing ortholog-level DE results (e.g. non-model salamander
  species) to check whether similar ortholog pairs both passing the DE threshold inflate
  the significant-hit count.
when_to_use_negative:
- Input is gene-level DE from a single reference-genome species where ortholog-mapping
  duplication cannot occur (no cross-species KO assignment ambiguity).
- The goal is to perform or interpret the DE analysis itself (ExpressAnalystR PerformDEAnal)
  rather than to audit the significance count for ortholog-level inflation.
- The DE threshold differs from the study's published cutoff (e.g. the generic FDR
  < 0.05 and |log2FC| > 1.5 criterion), so duplicated-pair membership would not be
  comparable to the ~3% benchmark.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0203
tools:
- name: ExpressAnalyst
  role: Web platform whose ortholog-level (KO-based) DE results for the salamander
    case study are the audited input; used via www.expressanalyst.ca or the synchronized
    R package.
  repo: https://github.com/xia-lab/ExpressAnalystR
- name: ExpressAnalystR
  role: R package providing the DE analysis functions (e.g. PerformDEAnal with FDR
    adjustment) whose ortholog-level outputs are filtered and audited for duplicated
    significant pairs.
  repo: https://github.com/xia-lab/ExpressAnalystR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:9005754d286d82f35c7586e415f5916f6b704ebf7389e500d04401f1f2692dfe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/ExpressAnalystR
  tools:
  - ExpressAnalyst
  - ExpressAnalystR
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Proportion estimation of duplicated DEGs

## Summary

Quantifies how much of a significant DE result set from ExpressAnalyst's ortholog-level analysis is inflated by similar ortholog groups that are both differentially expressed (with distinct species-specific patterns). It directly audits the ExpressAnalyst limitation that reads from one gene may map to different orthologs across species.

## When to use

Use when auditing ortholog-level DE results (e.g. non-model salamander species) to check whether similar ortholog pairs both passing the DE threshold inflate the significant-hit count.

## When NOT to use

- Input is gene-level DE from a single reference-genome species where ortholog-mapping duplication cannot occur (no cross-species KO assignment ambiguity).
- The goal is to perform or interpret the DE analysis itself (ExpressAnalystR PerformDEAnal) rather than to audit the significance count for ortholog-level inflation.
- The DE threshold differs from the study's published cutoff (e.g. the generic FDR < 0.05 and |log2FC| > 1.5 criterion), so duplicated-pair membership would not be comparable to the ~3% benchmark.

## Inputs

- Ortholog-level differential expression results table with per-species FDR-adjusted p-values and expression directions (from ExpressAnalyst/Seq2Fun KO-level mapping, e.g. salamander data SRR7499348–SRR7499365)
- DE significance threshold (salamander case: FDR-adjusted p-value < 0.1, following the original publication)

## Outputs

- duplicated_ortholog_pairs.csv — table of similar ortholog pairs where both members are DE at FDR < 0.1 with distinct species-specific expression patterns
- Single-number fraction of significant DE results attributable to duplicated ortholog pairs (~3% in the salamander case study)

## How to apply

Load the ortholog-level differential expression results table containing per-species FDR-adjusted p-values and expression directions. Filter ortholog groups to significant DEGs using the salamander threshold from the original publication: FDR-adjusted p-value < 0.1. Among significant groups, detect pairs of similar ortholog groups where both members pass the DE threshold but show distinct species-specific expression patterns (one member significant in one species, the other in a different species); flag these as duplicated DEGs and write them to duplicated_ortholog_pairs.csv. Compute the fraction of significant DE results accounted for by these duplicated ortholog pairs and report it as a single number. In the ExpressAnalyst salamander case study this fraction was ~3%, showing that cross-species ortholog mapping does not meaningfully inflate the DEG count.

## Related tools

- **ExpressAnalyst** (Web platform whose ortholog-level (KO-based) DE results for the salamander case study are the audited input; used via www.expressanalyst.ca or the synchronized R package.) — https://github.com/xia-lab/ExpressAnalystR
- **ExpressAnalystR** (R package providing the DE analysis functions (e.g. PerformDEAnal with FDR adjustment) whose ortholog-level outputs are filtered and audited for duplicated significant pairs.) — https://github.com/xia-lab/ExpressAnalystR

## Examples

```
Rscript -e 'res <- read.csv("salamander_de_results.csv"); sig <- res[res$FDR < 0.1, ]; pairs <- read.csv("duplicated_ortholog_pairs.csv"); frac <- nrow(pairs)/(2*nrow(sig)); cat(frac)'
```

## Evaluation signals

- Computed duplicated-DEG fraction approximates the reported value of ~3% of significant results in the salamander case study.
- Every row of duplicated_ortholog_pairs.csv contains two ortholog groups that are both significant at FDR-adjusted p-value < 0.1.
- Each flagged pair's members show distinct species-specific expression patterns (one member significant in one species, the other in a different species) — otherwise the pair is not a duplicated DEG.
- The reported fraction is bounded: numerator (duplicated significant pairs) ≤ denominator (all ortholog groups with FDR < 0.1), and both are drawn from the same results table.

## Limitations

- Reads from the same gene may be mapped to similar but different orthologs for different species, so care must be taken when interpreting cross-species ortholog-level results — this skill measures but does not eliminate that ambiguity.
- The ~3% benchmark is specific to the salamander case study and its FDR < 0.1 threshold; other datasets or cutoffs may yield different fractions.
- Detection depends on the definition of 'similar' ortholog groups; pairs whose members share the same species-specific pattern are not flagged, potentially undercounting redundant mapping.

## Evidence

- [discussion] Ortholog cross-mapping concern motivating the audit: "it’s possible for reads from the same gene to be mapped to similar but different orthologs for different species, and care must be taken while interpreting the results"
- [discussion] Salamander DE significance threshold: "genes were considered differentially expressed if their FDR-adjusted p-values were less than 0.1."
- [readme] FDR adjustment underlying the significance filter applied to the ortholog table: "Adjusted P-value using False Discovery Rate (FDR) method."
