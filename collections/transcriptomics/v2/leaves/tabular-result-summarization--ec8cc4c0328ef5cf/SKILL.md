---
name: tabular-result-summarization
description: Use when an analysis pipeline (e.g., iwa-miRNA Manual Curation output)
  produces many per-precursor records that must be condensed into a compact summary
  table of totals, counts, and fractions for validation and reporting.
when_to_use_negative:
- Input sRNA-Seq data come from a species without a reference genome (iwa-miRNA cannot
  handle these and ignores unmapped reads, so coordinate-based grouping is impossible)
- The goal is downstream enrichment analysis (e.g., GO annotation or miRNA set enrichment)
  rather than producing a summary table — iwa-miRNA has limited downstream analysis
  ability
- Records are still raw uncurated candidates containing false positives (the aggregated
  candidate list certainly contains false positive hits), so summary counts would
  be premature
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iwa-miRNA
  role: Its Manual Curation (manualCuration) module produces the summary and report
    pages that this summarization step condenses into group-level TSV output
  repo: https://github.com/cma2015/iwa-miRNA
- name: Rmarkdown
  role: Renders the result summaries into HTML pages for easy visualization, interpretation,
    and sharing
- name: Galaxy
  role: Framework hosting the iwa-miRNA modules whose tabular outputs are summarized
  repo: https://github.com/cma2015/iwa-miRNA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:d8018b0431e21ca222be73ecb842237a1c621c13515b8e3812c9a9a059eb5343
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - Rmarkdown
  - iwa-miRNA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# tabular-result-summarization

## Summary

Aggregates analysis results (e.g., wheat miRNA precursor homologous-group assignments) into a concise table-level summary such as homologous_groups_summary.tsv, and renders the summary into HTML pages with Rmarkdown for easy visualization, interpretation, and sharing.

## When to use

Use when an analysis pipeline (e.g., iwa-miRNA Manual Curation output) produces many per-precursor records that must be condensed into a compact summary table of totals, counts, and fractions for validation and reporting.

## When NOT to use

- Input sRNA-Seq data come from a species without a reference genome (iwa-miRNA cannot handle these and ignores unmapped reads, so coordinate-based grouping is impossible)
- The goal is downstream enrichment analysis (e.g., GO annotation or miRNA set enrichment) rather than producing a summary table — iwa-miRNA has limited downstream analysis ability
- Records are still raw uncurated candidates containing false positives (the aggregated candidate list certainly contains false positive hits), so summary counts would be premature

## Inputs

- Selected miRNA precursor list exported from the iwa-miRNA Manual Curation module (table format)
- Same precursor list in GFF3/GTF format with genomic coordinates
- Precursor sequences in FASTA format

## Outputs

- homologous_groups_summary.tsv (total groups, grouped precursors, triad count and fraction, per-subgenome counts)
- HTML summary/report pages rendered with Rmarkdown

## How to apply

Load the selected miRNA precursor list exported from the iwa-miRNA Manual Curation module in table, GFF3/GTF, and FASTA formats. Derive the aggregate metrics relevant to the analysis — in the wheat case, assign precursors to A/B/D subgenomes from GFF3/GTF coordinates, cluster them into homologous groups by sequence similarity, flag groups containing a complete A/B:D triad, and compute per-subgenome counts. Write the group-level summary (total groups, grouped precursors, triad count and fraction, per-subgenome counts) to homologous_groups_summary.tsv as a tab-separated table. Summarize the results into HTML pages using Rmarkdown so the numbers are easy to visualize, interpret, and share. Validate by checking the summary reproduces the published values: 1565 homologous groups from 1919 grouped precursors, a complete-triad fraction of 7.54% (118/1565), and 571 D-subgenome precursors (A: 611; B: 744).

## Related tools

- **iwa-miRNA** (Its Manual Curation (manualCuration) module produces the summary and report pages that this summarization step condenses into group-level TSV output) — https://github.com/cma2015/iwa-miRNA
- **Rmarkdown** (Renders the result summaries into HTML pages for easy visualization, interpretation, and sharing)
- **Galaxy** (Framework hosting the iwa-miRNA modules whose tabular outputs are summarized) — https://github.com/cma2015/iwa-miRNA

## Examples

```
Rscript -e "write.table(homologous_groups_summary, 'homologous_groups_summary.tsv', sep='\t', quote=FALSE, row.names=FALSE)"
```

## Evaluation signals

- Summary reports 1565 homologous groups derived from 1919 grouped precursors, matching the published wheat case study
- Complete-triad fraction equals 7.54% (118/1565)
- Per-subgenome counts are A: 611, B: 744, D: 571, with D the fewest
- Output file homologous_groups_summary.tsv exists and contains the expected columns (total groups, grouped precursors, triad count and fraction, per-subgenome counts)
- Eight A:B:D configuration classes are represented across the homologous groups

## Limitations

- Aggregated candidate lists contain false positive hits, so summary counts inherit upstream annotation noise
- iwa-miRNA ignores sequencing reads that do not map to the reference genome, so unplaced precursors are excluded from subgenome tallies
- Further wet experiments should validate candidates of particular interest; the summary does not establish functional validity

## Evidence

- [methods] Rmarkdown summary rendering: "summarizes the results into HTML pages, using Rmarkdown for easy visualization, interpretation, and sharing."
- [other] Summary TSV output: [evidence span withheld — anchor status: not_found]
- [other] Validation values: [evidence span withheld — anchor status: not_found]
- [methods] Manual curation module output: [evidence span withheld — anchor status: not_found]
