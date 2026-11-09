---
name: stratified-fraction-aggregation-by-cell-type-and-length
description: Use when a per-call result table already exists (e.g., classified Ginkgo
  CNA calls in 65 PTA-amplified brain cells) and the analysis goal is summary fractions
  broken down by cell type and call-length bins, as in HiScanner Fig. 4d–j.
when_to_use_negative:
- Input is raw read-depth or BAF data and calls have not yet been segmented, classified,
  and filtered (e.g., bulk-overlapping calls not removed) — stratification is meaningless
  until per-call labels exist
- The comparison involves other single-cell callers such as HiScanner, CHISEL, SCOPE,
  or SCYN whose output formats and concordance definitions differ from the Ginkgo
  per-call table assumed here
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Ginkgo
  role: Source of the read-depth-based CNA calls per single-cell BAM whose discordance
    fractions are aggregated
  repo: https://github.com/robertaboukhalil/ginkgo
- name: HiScanner
  role: Reference allele-specific CNA caller whose BAF-based framework defines the
    concordance criterion and the Fig. 4d–j context
  repo: https://github.com/parklab/HiScanner
- name: SCAN2
  role: Genotyper run before HiScanner on PTA-amplified single cells to support the
    broader brain-cell CNA analysis
  repo: https://github.com/parklab/SCAN2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:c15c46796061f59d6690f62336203985c602e531cef43c69e08b8f9932e38540
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robertaboukhalil/ginkgo
  tools:
  - Ginkgo
  - HiScanner
  - SCAN2
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# stratified-fraction-aggregation-by-cell-type-and-length

## Summary

Aggregates per-call classification results (e.g., BAF-concordant vs BAF-discordant Ginkgo CNA calls) into discordance fractions stratified by cell type (neurons vs oligodendrocytes) and by call-length bins, reproducing the Fig. 4d–j breakdown used to expose Ginkgo's unsupported CNA calls in brain cells.

## When to use

Use when a per-call result table already exists (e.g., classified Ginkgo CNA calls in 65 PTA-amplified brain cells) and the analysis goal is summary fractions broken down by cell type and call-length bins, as in HiScanner Fig. 4d–j.

## When NOT to use

- Input is raw read-depth or BAF data and calls have not yet been segmented, classified, and filtered (e.g., bulk-overlapping calls not removed) — stratification is meaningless until per-call labels exist
- The comparison involves other single-cell callers such as HiScanner, CHISEL, SCOPE, or SCYN whose output formats and concordance definitions differ from the Ginkgo per-call table assumed here

## Inputs

- Per-call TSV of Ginkgo CNA calls with cell type, call length, and BAF-concordance/discordance labels
- Cell-type metadata for the 65 single cells from the 11 neurotypical brains
- Call-length bin definitions matching the Fig. 4d–j stratification

## Outputs

- ginkgo_baf_concordance.tsv containing per-call classification records
- Stratified summary table of discordance fractions per cell type and per call-length bin

## How to apply

Start from a per-call TSV where each Ginkgo-called CNA region in the 65 brain cells is annotated with its cell, cell type (neuron or oligodendrocyte), call length, and BAF-concordance label (concordant when local BAF deviates from 0.5 in the read-depth-implied direction, discordant otherwise, with calls overlapping bulk CNAs removed). Group calls by cell type and by call-length bin, count discordant and total calls per stratum, and compute discordance fractions. Aggregate oligodendrocyte and neuronal calls separately across the 11 neurotypical brains. Write both the per-call table (ginkgo_baf_concordance.tsv) and the stratified summary fractions so the headline numbers are auditable from raw counts. Validate that the fractions reproduce 97% (581/599) discordant in oligodendrocytes and 66% (1410/2121) in neurons.

## Related tools

- **Ginkgo** (Source of the read-depth-based CNA calls per single-cell BAM whose discordance fractions are aggregated) — https://github.com/robertaboukhalil/ginkgo
- **HiScanner** (Reference allele-specific CNA caller whose BAF-based framework defines the concordance criterion and the Fig. 4d–j context) — https://github.com/parklab/HiScanner
- **SCAN2** (Genotyper run before HiScanner on PTA-amplified single cells to support the broader brain-cell CNA analysis) — https://github.com/parklab/SCAN2

## Examples

```
python aggregate_baf_concordance.py --calls ginkgo_baf_concordance.tsv --stratify cell_type,call_length_bin --out discordance_summary.tsv
```

## Evaluation signals

- Oligodendrocyte discordance fraction reproduces 97% (581/599 calls)
- Neuronal discordance fraction reproduces 66% (1410/2121 calls)
- Per-call table row count equals 599 + 2121 = 2720 Ginkgo calls (after bulk-CNA-overlap removal)
- Every aggregated fraction can be recomputed exactly from counts in the per-call TSV strata
- All retained calls are autosomal and derived from phased gHETs with bulk VAF between 0.1 and 0.9

## Limitations

- Calls overlapping bulk CNA calls are removed before aggregation, so fractions do not describe all raw Ginkgo calls
- Discordance is only assessed on autosomes and on SNPs with bulk VAF in 0.1–0.9, limiting genomic coverage of the concordance test
- Only 65 cells from 11 brains were analyzed, so stratified fractions carry sampling uncertainty and are specific to this PTA-amplified scWGS cohort

## Evidence

- [discussion] Oligodendrocyte and neuronal discordance fractions: [evidence span withheld — anchor status: not_found]
- [discussion] Filter of overlapping calls and gHET VAF window: "Single-cell CNA calls that overlapped with bulk CNA calls were removed"
- [discussion] gHET retention criterion: "Only the gHETs that can be phased by SHAPEIT2 and have a variant allele frequency between 0.1 and 0.9 in bulk data were retained"
- [discussion] Autosome restriction for cross-sex consistency: "To maintain consistency across samples from both sexes, our analysis was restricted to autosomal chromosomes"
- [readme] Ginkgo command-line source used in benchmark: "Ginkgo’s command line version was downloaded from https://github.com/robertaboukhalil/ginkgo"
