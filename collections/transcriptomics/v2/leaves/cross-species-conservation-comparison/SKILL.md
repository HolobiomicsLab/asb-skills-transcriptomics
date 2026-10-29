---
name: cross-species-conservation-comparison
description: Use when you have predicted core promoter element records (position,
  score, TSS) for multiple species and want to test whether element positioning —
  e.g. TATA, dInr, PB relative to CAGE-annotated TSSs — is conserved across metazoans.
when_to_use_negative:
- Input sequences have not yet been scanned with ElemeNT — run ElemeNT 2023 (web or
  command line) first to generate predicted element records before comparing species
- Goal is de novo motif discovery rather than comparing positions/scores of already-predicted
  core promoter elements across species
- Only one species is available — the skill's value is the cross-species comparison
  (worm to human), which requires multi-species CORE database records
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3295
tools:
- name: ElemeNT 2023
  role: Source of the CORE database of predicted core promoter elements around annotated
    TSSs, expanded to 10 species, used for the cross-species positioning comparison
  repo: https://github.com/OritAdato/ElemeNT
- name: ElemeNT
  role: Interactive Perl-based detection tool underlying the CORE database; its GitHub
    repo also provides the manuscript analysis source code under ElemeNT/ElemeNT2023/src/Manuscript
  repo: https://github.com/OritAdato/ElemeNT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae110
    title: 'ElemeNT 2023: an enhanced tool for detection and curation of core promoter
      elements'
  source_content_hash: sha256:46696068ca29917748e7a6ef169986d71024e22adcc3485ab0db46016ce0a7bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OritAdato/ElemeNT
  tools:
  - ElemeNT
  - ElemeNT 2023
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-species-conservation-comparison

## Summary

Reproduces the ElemeNT 2023 cross-species analysis (Supplementary Fig. S3) in which per-species position and score distributions of predicted core promoter elements from the 10-species CORE database are compared relative to EPDnew CAGE-annotated TSSs to assess evolutionary conservation of TATA, dInr and PB positioning.

## When to use

Use when you have predicted core promoter element records (position, score, TSS) for multiple species and want to test whether element positioning — e.g. TATA, dInr, PB relative to CAGE-annotated TSSs — is conserved across metazoans.

## When NOT to use

- Input sequences have not yet been scanned with ElemeNT — run ElemeNT 2023 (web or command line) first to generate predicted element records before comparing species
- Goal is de novo motif discovery rather than comparing positions/scores of already-predicted core promoter elements across species
- Only one species is available — the skill's value is the cross-species comparison (worm to human), which requires multi-species CORE database records

## Inputs

- Expanded 10-species ElemeNT CORE database of predicted core promoter elements around EPDnew CAGE-annotated TSSs (from github.com/OritAdato/ElemeNT)
- EPDnew CAGE-annotated TSS coordinates per species
- Element type labels (TATA, dInr, PB, other core promoter elements) with position and score fields

## Outputs

- Per-species frequency distributions of core promoter element positions relative to the TSS
- Per-species summary statistics of element scores (e.g. mean score per element type)
- core_element_distributions.csv containing position/score distributions and conservation verification results

## How to apply

Download the expanded 10-species CORE database (worm to human) of predicted core promoter elements from the ElemeNT GitHub repository (github.com/OritAdato/ElemeNT). Parse each record and express every predicted element's position and score relative to its EPDnew CAGE-annotated TSS. Compute per-species frequency distributions of element positions relative to the TSS for each element type (TATA, dInr, PB), plus per-species summary statistics such as the mean score per element type. Compare distributions across the 10 species to verify the reported qualitative patterns: TATA and dInr positions are largely conserved, dInr mean score is elevated in fly and honeybee, and the PB motif is restricted to D. melanogaster with enrichment at +26. Write the per-species position and score distributions together with the verification results to a CSV (core_element_distributions.csv).

## Related tools

- **ElemeNT 2023** (Source of the CORE database of predicted core promoter elements around annotated TSSs, expanded to 10 species, used for the cross-species positioning comparison) — https://github.com/OritAdato/ElemeNT
- **ElemeNT** (Interactive Perl-based detection tool underlying the CORE database; its GitHub repo also provides the manuscript analysis source code under ElemeNT/ElemeNT2023/src/Manuscript) — https://github.com/OritAdato/ElemeNT

## Examples

```
git clone https://github.com/OritAdato/ElemeNT; python src/Manuscript/cross_species_positions.py --core_db CORE_database --out core_element_distributions.csv
```

## Evaluation signals

- Per-species position distributions show TATA box and dInr motif positions are largely conserved across the 10 analyzed species
- Computed dInr mean score is elevated in fly (D. melanogaster) and honeybee relative to other species
- PB motif occurrences are restricted to D. melanogaster with peak enrichment around +26 relative to the TSS
- All element positions are expressed relative to CAGE-annotated EPDnew TSSs (spacing sanity check: element positions fall within a plausible core promoter window)
- Output CSV core_element_distributions.csv contains one distribution set per species and per element type

## Limitations

- Verification is qualitative — acceptance is based on reproducing the reported qualitative patterns (conserved TATA/dInr positioning, elevated fly/honeybee dInr score, fly-restricted PB at +26), not on a quantitative statistical test
- CORE database elements are predicted by ElemeNT rather than experimentally validated, so distributions reflect motif-prediction biases
- Position/score comparisons depend on the quality of the EPDnew CAGE-annotated TSS sets per species

## Evidence

- [supplementary] Cross-species comparison of the CORE database promoter sequences revealed that the positions of both the TATA box and the dInr motif are largely conserved among the analyzed species: [evidence span withheld — anchor status: not_found]
- [abstract] CORE database expanded to 10 species: "ElemeNT's CORE database, which contains predicted core promoter elements around annotated TSSs, is now expanded to cover 10 species"
- [readme] README manuscript analysis source code: "ElemeNT/ElemeNT2023/src/Manuscript - the source code used for the analysis included in the manuscript"
