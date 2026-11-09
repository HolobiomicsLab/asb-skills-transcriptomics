---
name: cross-species-expression-pattern-comparison
description: Use when interpreting differential expression results from ExpressAnalyst
  across taxonomically diverse species lacking reference transcriptomes, where KO-based
  mapping could double-count homologous genes as separate DE ortholog groups.
when_to_use_negative:
- The analysis is within a single species with a reference genome, so gene IDs are
  directly comparable and ortholog duplication cannot inflate DEG counts
- Input is functional enrichment (pathway) results rather than a per-ortholog DE table
  with per-species FDR values
- The comparison uses different tissues, treatments, or timepoints within one species
  rather than the same gene compared across species via ortholog mapping
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ExpressAnalyst
  role: Web platform producing the ortholog-level differential expression results
    from raw RNA-seq reads that are the input to this duplication audit
  repo: https://www.nature.com/articles/s41467-023-38785-y.pdf
- name: ExpressAnalystR
  role: R package for reproducing the differential expression analysis (annotation,
    normalization, DE) whose output tables are screened for duplicated DE ortholog
    pairs
  repo: github.com/xia-lab/ExpressAnalystR
- name: Seq2Fun 2.0.2
  role: Maps translated RNA-seq reads to KEGG ortholog (KO) databases, generating
    the per-species KO-level counts that may contain the cross-species duplication
    being quantified
- name: EcoOmicsDB
  role: Ortholog database providing the KO group structure used to define which ortholog
    pairs are 'similar' candidates for duplication
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:05782600dc9fbd6ec96566efe7070a885eab3d76bbd7a1441a2858da61654f0f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EcoOmicsDB
  - ExpressAnalyst
  - ExpressAnalystR
  - Seq2Fun 2.0.2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# cross-species-expression-pattern-comparison

## Summary

Quantifies how much of the significant differential expression count in a multi-species analysis is inflated by reads from the same gene mapping to similar but distinct ortholog IDs in different species. In the ExpressAnalyst salamander case study this duplication was rare (~3% of significant results), validating the KO/EcoOmicsDB ortholog-level quantification strategy.

## When to use

Use when interpreting differential expression results from ExpressAnalyst across taxonomically diverse species lacking reference transcriptomes, where KO-based mapping could double-count homologous genes as separate DE ortholog groups.

## When NOT to use

- The analysis is within a single species with a reference genome, so gene IDs are directly comparable and ortholog duplication cannot inflate DEG counts
- Input is functional enrichment (pathway) results rather than a per-ortholog DE table with per-species FDR values
- The comparison uses different tissues, treatments, or timepoints within one species rather than the same gene compared across species via ortholog mapping

## Inputs

- Ortholog-level differential expression results table with per-species FDR-adjusted p-values and expression direction (up/down) per species
- Species-to-sample mapping for the multi-species RNA-seq study (e.g., salamander SRA samples SRR7499348–SRR7499365)
- Ortholog group definitions from the KEGG/EcoOmicsDB-based database used by Seq2Fun/ExpressAnalyst

## Outputs

- duplicated_ortholog_pairs.csv — table of similar ortholog pairs where both members are significant DEGs (FDR < 0.1) with distinct species-specific expression patterns
- Single numeric value: fraction of significant DE results accounted for by duplicated ortholog pairs (expected ~3% in the salamander case study)

## How to apply

Load the ortholog-level differential expression table containing per-species FDR-adjusted p-values and expression directions. Restrict to significant DEGs using the salamander threshold from the original publication (FDR-adjusted p-value < 0.1, which the article notes as 'genes were considered differentially expressed if their FDR-adjusted p-values were less than 0.1'). Among significant ortholog groups, detect pairs of similar orthologs where both members pass the DE threshold but show distinct species-specific patterns — one member significant in one species, the other in a different species — and write these to duplicated_ortholog_pairs.csv. Compute the fraction of significant DE results accounted for by duplicated pairs and report it as a single number; the article's concern stems from the limitation that 'it's possible for reads from the same gene to be mapped to similar but different orthologs for different species'. A low fraction (~3%) supports interpreting the full significant DEG set as non-redundant.

## Related tools

- **ExpressAnalyst** (Web platform producing the ortholog-level differential expression results from raw RNA-seq reads that are the input to this duplication audit) — https://www.nature.com/articles/s41467-023-38785-y.pdf
- **ExpressAnalystR** (R package for reproducing the differential expression analysis (annotation, normalization, DE) whose output tables are screened for duplicated DE ortholog pairs) — github.com/xia-lab/ExpressAnalystR
- **Seq2Fun 2.0.2** (Maps translated RNA-seq reads to KEGG ortholog (KO) databases, generating the per-species KO-level counts that may contain the cross-species duplication being quantified)
- **EcoOmicsDB** (Ortholog database providing the KO group structure used to define which ortholog pairs are 'similar' candidates for duplication)

## Examples

```
devtools::install_github("xia-lab/ExpressAnalystR", build = TRUE, build_opts = c("--no-resave-data", "--no-manual")) ; dataSet <- readDataset("salamander_degs.txt") ; sig <- subset(dataSet$comp.res, FDR < 0.1) ; frac_dup <- nrow(duplicated_pairs) / nrow(sig)
```

## Evaluation signals

- Computed duplicated-DEG fraction should approximate the reported value of ~3% for the salamander case study
- duplicated_ortholog_pairs.csv must contain only pairs whose members are both differentially expressed at FDR-adjusted p-value < 0.1
- Each flagged pair must show species-specific expression patterns — one member significant in one species, the other in a different species — not co-significance in the same species
- The denominator (number of significant ortholog results at FDR < 0.1) must match the full salamander DEG count, including duplicated entries
- Pairs flagged should correspond to similar (not identical) ortholog groups, consistent with reads from one gene mapping to different but related KOs

## Limitations

- The article explicitly flags this as an interpretation risk: 'it's possible for reads from the same gene to be mapped to similar but different orthologs for different species, and care must be taken while interpreting the results', especially for taxonomically diverse comparisons
- The estimate is dataset- and threshold-dependent: it relies on the salamander publication's looser threshold (FDR < 0.1) rather than the standard FDR < 0.05 plus |logFC| > 1.5 used elsewhere in the paper
- Details on sample sex were not included in the original salamander manuscript, limiting full metadata-driven stratification of expression patterns
- Improper sub-group database selection (e.g., mapping protist reads to the 'fishes' database) can miss key functional groups and distort ortholog assignment altogether

## Evidence

- [discussion] Ortholog duplication limitation: "it’s possible for reads from the same gene to be mapped to similar but different orthologs for different species, and care must be taken while interpreting the results"
- [discussion] Salamander DEG threshold: "genes were considered differentially expressed if their FDR-adjusted p-values were less than 0.1."
- [abstract] Salamander case study scope: "demonstrate its utility with a case study of RNA-sequencing data from multiple non-model salamander species, including two that do not have a reference transcriptome."
- [readme] ExpressAnalystR install (README): "devtools::install_github("xia-lab/ExpressAnalystR", build = TRUE, build_opts = c("--no-resave-data", "--no-manual"))"
