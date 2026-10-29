---
name: tss-anchored-motif-coordinate-analysis
description: Use when you have predicted core promoter elements anchored to annotated
  TSSs across multiple species and need to compare element position/score distributions
  to assess conservation or motif positioning quality.
when_to_use_negative:
- Your goal is de novo motif discovery or scanning arbitrary user-supplied sequences
  for motifs rather than analyzing already-predicted elements anchored to annotated
  TSSs.
- Your TSS annotations are not CAGE-based/EPDnew-style, so element positions cannot
  be meaningfully expressed relative to a high-confidence TSS.
- The species or element types of interest are outside the CORE database's 10 metazoan
  species (worm to human).
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: ElemeNT 2023
  role: Source of the expanded 10-species CORE database of predicted core promoter
    elements around annotated TSSs; its manuscript source code in src/Manuscript implements
    the cross-species positioning analysis
  repo: https://github.com/OritAdato/ElemeNT
- name: ElemeNT
  role: Interactive Perl tool underlying CORE database element predictions used for
    the cross-species TSS-anchored coordinate comparison
  repo: https://github.com/OritAdato/ElemeNT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae110
    title: 'ElemeNT 2023: an enhanced tool for detection and curation of core promoter
      elements'
  source_content_hash: sha256:b5e3a579d2f1797dc97df55935f615fc2d11faabc2db081b2c88c58396c07a49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OritAdato/ElemeNT
  tools:
  - ElemeNT
  - ElemeNT 2023
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tss-anchored-motif-coordinate-analysis

## Summary

Workflow for computing per-species frequency distributions of predicted core promoter element positions (TATA, dInr, PB) relative to CAGE-annotated TSSs from EPDnew, using ElemeNT 2023's expanded 10-species CORE database to test cross-species conservation of element positioning.

## When to use

Use when you have predicted core promoter elements anchored to annotated TSSs across multiple species and need to compare element position/score distributions to assess conservation or motif positioning quality.

## When NOT to use

- Your goal is de novo motif discovery or scanning arbitrary user-supplied sequences for motifs rather than analyzing already-predicted elements anchored to annotated TSSs.
- Your TSS annotations are not CAGE-based/EPDnew-style, so element positions cannot be meaningfully expressed relative to a high-confidence TSS.
- The species or element types of interest are outside the CORE database's 10 metazoan species (worm to human).

## Inputs

- Expanded 10-species CORE database of predicted core promoter elements around EPDnew CAGE-annotated TSSs (from github.com/OritAdato/ElemeNT)
- EPDnew CAGE-annotated transcription start site (TSS) coordinates per species
- Core promoter element type labels and ElemeNT prediction scores (TATA, dInr, PB, etc.)

## Outputs

- Per-species frequency distributions of element positions relative to the TSS for each core promoter element type
- Per-species summary statistics of element scores (e.g. mean score per element type)
- core_element_distributions.csv containing position/score distributions and verification results

## How to apply

Download the expanded 10-species CORE database of predicted core promoter elements (worm to human) from the ElemeNT GitHub repository. Parse each record and express every predicted element's position and score relative to its CAGE-annotated TSS from EPDnew, then compute per-species frequency distributions of element positions for each element type (TATA, dInr, PB) and per-species summary statistics such as mean element score. Compare distributions across the 10 species: TATA and dInr positions should be largely conserved, dInr mean score should be higher in fly and honeybee, and the PB motif should be restricted to D. melanogaster with enrichment at +26 relative to the TSS. Write per-species position and score distributions together with verification results (e.g. core_element_distributions.csv) so the qualitative patterns can be checked against the reported ones.

## Related tools

- **ElemeNT 2023** (Source of the expanded 10-species CORE database of predicted core promoter elements around annotated TSSs; its manuscript source code in src/Manuscript implements the cross-species positioning analysis) — https://github.com/OritAdato/ElemeNT
- **ElemeNT** (Interactive Perl tool underlying CORE database element predictions used for the cross-species TSS-anchored coordinate comparison) — https://github.com/OritAdato/ElemeNT

## Examples

```
git clone https://github.com/OritAdato/ElemeNT && python parse_core_db.py --core-db ElemeNT2023/CORE --tss-epdnew epdnew.tsv --out core_element_distributions.csv
```

## Evaluation signals

- TATA box and dInr motif positions are largely conserved across the 10 analyzed species (worm to human) when plotted relative to the TSS
- dInr mean prediction score is higher in fly (D. melanogaster) and honeybee than in other species
- PB motif occurrences are restricted to D. melanogaster and enriched at position +26 relative to the TSS
- Output file (e.g. core_element_distributions.csv) contains one per-species position and score distribution per core promoter element type
- All element positions are expressed relative to the CAGE-annotated TSS from EPDnew, not raw genomic coordinates

## Limitations

- Cross-species conclusions are limited to the 10 metazoan species covered by the expanded CORE database (ranging from worms to human).
- Elements are ElemeNT-predicted rather than experimentally validated, so distributions reflect predicted motif positioning.
- PB motif analysis is only meaningful for D. melanogaster, where the motif is present.

## Evidence

- [results] Cross-species comparison of the CORE database promoter sequences revealed that the positions of both the TATA box and the dInr motif are largely conserved among the analyzed species: [evidence span withheld — anchor status: not_found]
- [abstract] ElemeNT's CORE database, which contains predicted core promoter elements around annotated TSSs, is now expanded to cover 10 species: "ElemeNT's CORE database, which contains predicted core promoter elements around annotated TSSs, is now expanded to cover 10 species, ranging from worms to human"
- [abstract] The source code and command line version of ElemeNT 2023 are available at https://github.com/OritAdato/ElemeNT.: "The source code and command line version of ElemeNT 2023 are available at https://github.com/OritAdato/ElemeNT."
- [readme] ElemeNT2023/src/Manuscript - the source code used for the analysis included in the manuscript: "**> ElemeNT/ElemeNT2023/src/Manuscript** - the source code used for the analysis included in the manuscript"
