---
name: microrna-exploratory-analysis
description: Use when you have a manually curated/selected list of plant miRNA precursors
  (table + GFF3/GTF + FASTA) and need exploratory characterization — HTML summary
  reports, per-subgenome counts, or homologous-group/triad structure.
when_to_use_negative:
- The sRNA-Seq data are from a species without a reference genome — iwa-miRNA cannot
  handle such data, so precursor coordinates needed for this analysis do not exist
- The candidates are still raw/unfiltered predictions (before MiRNA Selection / Manual
  Curation) — exploratory analysis should operate on the curated selected set, not
  the aggregate candidate list that certainly contains false positive hits
- The goal is enrichment analysis (e.g., GO annotation or miRNA set enrichment) or
  cross-species comparative analysis — the article explicitly notes iwa-miRNA has
  limited ability for these downstream analyses
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0780
tools:
- name: iwa-miRNA
  role: Galaxy-based framework whose Manual Curation module (manualCuration) performs
    exploratory analysis of selected miRNAs and summarizes results into HTML pages
  repo: https://github.com/cma2015/iwa-miRNA
- name: Rmarkdown
  role: Used to render the manualCuration summary results into HTML report pages for
    visualization, interpretation, and sharing
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Platform in which the iwa-miRNA modules (including manualCuration) are organized
    with user-friendly web interfaces
  repo: https://github.com/cma2015/iwa-miRNA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:a091282256a3500fc901ba6cb79241c88f4a04863dfe9a20802b262eeba17a59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - Rmarkdown
  - iwa-miRNA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# microrna-exploratory-analysis

## Summary

Exploratory downstream analysis of curated plant miRNA precursor candidates, as implemented in the iwa-miRNA Manual Curation module: summarizing selected miRNAs into Rmarkdown HTML reports and characterizing their genomic organization (e.g., subgenome assignment and homologous-group/triad structure in polyploids such as hexaploid wheat).

## When to use

Use when you have a manually curated/selected list of plant miRNA precursors (table + GFF3/GTF + FASTA) and need exploratory characterization — HTML summary reports, per-subgenome counts, or homologous-group/triad structure.

## When NOT to use

- The sRNA-Seq data are from a species without a reference genome — iwa-miRNA cannot handle such data, so precursor coordinates needed for this analysis do not exist
- The candidates are still raw/unfiltered predictions (before MiRNA Selection / Manual Curation) — exploratory analysis should operate on the curated selected set, not the aggregate candidate list that certainly contains false positive hits
- The goal is enrichment analysis (e.g., GO annotation or miRNA set enrichment) or cross-species comparative analysis — the article explicitly notes iwa-miRNA has limited ability for these downstream analyses

## Inputs

- Selected miRNA precursor list exported from the iwa-miRNA Manual Curation module in table format
- Selected miRNA precursor list in GFF3/GTF format with genomic coordinates
- Selected miRNA precursor sequences in FASTA format
- Reference genome annotation (for subgenome assignment from coordinates)

## Outputs

- Group-level summary table (homologous_groups_summary.tsv: total groups, grouped precursors, triad count/fraction, per-subgenome counts)
- Rmarkdown-rendered HTML summary/report pages from the manualCuration module
- Per-precursor subgenome (A/B/D) assignments
- Homologous-group clustering of precursors across subgenomes, flagged for complete triads

## How to apply

Export the selected precursor list from the iwa-miRNA Manual Curation module in table, GFF3/GTF, and FASTA formats. Use the manualCuration module to summarize results into HTML pages with Rmarkdown for visualization and sharing. For polyploid genomes (e.g., hexaploid bread wheat), assign each precursor to its subgenome (A, B, or D) from the GFF3/GTF genomic coordinates, cluster precursors into homologous groups by sequence similarity across subgenomes, flag groups containing a complete A:B:D triad, and compute per-subgenome counts plus total/triad statistics. Write the group-level summary (total groups, grouped precursors, triad count and fraction, per-subgenome counts) to a TSV such as homologous_groups_summary.tsv. In the wheat case study this yielded 1919 grouped precursors in 1565 homologous groups with a 7.54% (118/1565) complete-triad fraction and the fewest precursors (571) on the D subgenome.

## Related tools

- **iwa-miRNA** (Galaxy-based framework whose Manual Curation module (manualCuration) performs exploratory analysis of selected miRNAs and summarizes results into HTML pages) — https://github.com/cma2015/iwa-miRNA
- **Rmarkdown** (Used to render the manualCuration summary results into HTML report pages for visualization, interpretation, and sharing) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Platform in which the iwa-miRNA modules (including manualCuration) are organized with user-friendly web interfaces) — https://github.com/cma2015/iwa-miRNA

## Examples

```
Rscript -e 'rmarkdown::render("manualCuration_summary.Rmd")'  # render iwa-miRNA selected-miRNA summary to HTML
```

## Evaluation signals

- Reproducing the wheat case study yields 1565 homologous groups from 1919 grouped precursors, matching the published values
- Complete-triad fraction equals 118/1565 ≈ 7.54% of homologous groups
- Per-subgenome counts report the D subgenome with the fewest precursors (571; A: 611, B: 744)
- Every precursor with clear genomic location is assigned to exactly one subgenome (A, B, or D) from the GFF3/GTF coordinates
- manualCuration produces a non-empty HTML report page rendered via Rmarkdown summarizing the selected miRNAs

## Limitations

- iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences, so this exploratory analysis requires a reference genome/annotation
- Limited downstream analysis of selected miRNAs: no enrichment analyses such as GO annotation or miRNA set enrichment, and no cross-species comparative analysis
- Aggregated candidate lists contain false positives; exploratory results should be interpreted on curated selections, and wet experiments are still needed to validate candidates of particular interest

## Evidence

- [methods] Manual Curation module report rendering: "summarizes the results into HTML pages, using Rmarkdown for easy visualization, interpretation, and sharing."
- [readme] README downstream tools statement: "It also provides users with some useful tools for downstream exploratory analysis."
- [other] Wheat homologous-group case study result: [evidence span withheld — anchor status: not_found]
- [other] Complete triad fraction: [evidence span withheld — anchor status: not_found]
- [methods] Module structure grounding the exploratory analysis step: "The iwa-miRNA comprises three modules: MiRNA Compilation, MiRNA Selection, and Manual Curation"
