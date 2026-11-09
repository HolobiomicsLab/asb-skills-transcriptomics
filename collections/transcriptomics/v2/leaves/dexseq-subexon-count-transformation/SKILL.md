---
name: dexseq-subexon-count-transformation
description: Use when you already hold a fortuna signature-counts table (cnt.tsv with
  subexon-ID signatures and counts) and need DEXSeq-compatible per-subexon counts
  for alternative splicing analysis.
when_to_use_negative:
- Input is raw reads or alignments rather than a fortuna signature-counts table —
  signature counts must first be produced by fortuna's --index/--run pipeline (pseudoalignment
  step D)
- The goal is annotation of novel splicing events (exon skipping, intron retention,
  etc.) — that is fortuna's separate E2 output (alt.tsv), not the count aggregation
- Transcript- or gene-level abundance quantification is needed (kallisto output) rather
  than subexon-level splicing counts
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: fortuna
  role: Provides the '--trans' command that aggregates signature counts (-incnt) into
    DEXSeq-style per-subexon counts (-outcnt)
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: 'Downstream-implicit: signature counts are derived from kallisto''s equivalence
    classes of pseudoaligned reads, enabling the count conversion'
  repo: https://github.com/pachterlab/kallisto
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:0458aaa28ada1dfc9cba7347a6e1ecc6f790d1c140dc46baa5a7b862b006410c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - fortuna
  - kallisto
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# dexseq-subexon-count-transformation

## Summary

Aggregates fortuna's elementary signature counts into larger DEXSeq-style per-subexon counting units without reprocessing raw reads or alignments: for every subexon s, sum the counts of all '|' -joined mapping signatures that contain that subexon.

## When to use

Use when you already hold a fortuna signature-counts table (cnt.tsv with subexon-ID signatures and counts) and need DEXSeq-compatible per-subexon counts for alternative splicing analysis.

## When NOT to use

- Input is raw reads or alignments rather than a fortuna signature-counts table — signature counts must first be produced by fortuna's --index/--run pipeline (pseudoalignment step D)
- The goal is annotation of novel splicing events (exon skipping, intron retention, etc.) — that is fortuna's separate E2 output (alt.tsv), not the count aggregation
- Transcript- or gene-level abundance quantification is needed (kallisto output) rather than subexon-level splicing counts

## Inputs

- fortuna signature-counts table (cnt.tsv): tab-separated lines of '|' -joined subexon IDs (NodeIds from the segmented GTF) followed by a read count
- Segmented transcript annotation GTF produced by bin/processGTF.sh (defines the subexon ID space)

## Outputs

- Tab-separated per-subexon count table where each subexon's count equals the sum of all signature counts whose signature contains that subexon ID
- DEXSeq-compatible subexon counting units (E1) suitable for alternative splicing analysis

## How to apply

Load the signature-counts table (cnt.tsv) in which each tab-separated line is a mapping signature — a sequence of subexon IDs from the segmented GTF joined by '|' — followed by the read count, e.g. '223832|223833|223836|	2274'. Run fortuna's '--trans' command with '-incnt' pointing to the signature-counts file and '-outcnt' to the output file, so fortuna aggregates signature counts into the per-subexon counting units used by DEXSeq (each subexon count = sum of signatures containing it). This exploits the fact that signature counts are the most elementary splicing units derived from kallisto's equivalence classes, so larger units can be obtained purely by aggregation — no reprocessing of raw reads or alignments is needed. Subexon IDs are taken from the preprocessed GTF (processGTF.sh output, field NodeId), so the same segmented annotation must underpin both the input counts and any downstream DEXSeq analysis.

## Related tools

- **fortuna** (Provides the '--trans' command that aggregates signature counts (-incnt) into DEXSeq-style per-subexon counts (-outcnt)) — https://github.com/canzarlab/fortuna
- **kallisto** (Downstream-implicit: signature counts are derived from kallisto's equivalence classes of pseudoaligned reads, enabling the count conversion) — https://github.com/pachterlab/kallisto

## Examples

```
fortuna --trans -incnt cnt.tsv -outcnt subexon_counts.tsv
```

## Evaluation signals

- Hand-check on a small example (e.g. the paper's Fig. 2 subexon configuration): every output subexon count equals the sum of all signature counts whose '|' -separated signature contains that subexon ID
- Output is a tab-separated table whose subexon IDs match the NodeId field of the segmented input GTF (sample.rg.gtf)
- Aggregation is order- and completeness-consistent: every subexon ID appearing in any input signature appears in the output; no new IDs are introduced
- Sanity check against the README example: signature '223832|223833|223836|' with count 2274 must contribute 2274 to each of subexons 223832, 223833 and 223836 in the output

## Limitations

- The aggregation is one-directional: subexon counts cannot be decomposed back into signature counts, so this step destroys the elementary-unit information
- Signatures spanning multiple subexons contribute their full count to every contained subexon, so overlapping contributions are inherent to DEXSeq-style units (by design of the DEXSeq counting scheme, not an error)
- Subexon IDs only match if the GTF was preprocessed with bin/processGTF.sh; using a differently segmented annotation will break ID correspondence

## Evidence

- [methods] DEXSeq-style counts are derived from signature counts by a simple aggregation: for every subexon s, sum the counts of all mapping signatures that contain that subexon.: [evidence span withheld — anchor status: not_found]
- [readme] Computed counts can be directly used for alternative splicing analysis or added up to larger units such as those used by DEXSeq [3] (E1).: "Computed counts can be directly used for alternative splicing analysis or added up to larger units such as those used by DEXSeq [3] (E1)."
- [readme] File cnt.tsv contains signature counts in the format documented below.: "File cnt.tsv contains signature counts in the format documented below."
