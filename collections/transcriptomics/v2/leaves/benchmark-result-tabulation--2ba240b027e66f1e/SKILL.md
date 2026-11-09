---
name: benchmark-result-tabulation
description: Use when comparing isoform-assembly tools on the same dataset and you
  need a machine-readable, validated summary table (e.g. assembler vs. uniquely_assigned_isoforms)
  derived from IsoQuant assignment files.
when_to_use_negative:
- When IsoQuant assignment outputs do not yet exist for all assemblers — this skill
  tabulates post-assignment results, it does not run the assemblers or the assignment
  step itself
- When ambiguous multi-isoform IsoQuant assignments should be included in the count
  — this skill counts only unambiguous single-isoform assignments
- When a failed run should legitimately be scored as zero (e.g. a completed run that
  recovered no isoforms) — here a pipeline failure (RNA-Bloom, k-mer counting step)
  must be recorded as 'failure', not 0
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3308
tools:
- name: IsoQuant
  role: produces the per-assembler transcript-assignment outputs against GENCODE v32
    that are parsed and filtered to the ATP6AP2 locus
- name: pandas
  role: parses IsoQuant assignment tables and computes per-assembler counts of uniquely
    assigned ATP6AP2 isoforms
- name: cloudrnaSPAdes
  role: benchmark subject assembler whose seven unique ATP6AP2 isoform assignments
    top the tabulated results
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: rnaSPAdes
  role: benchmark subject assembler; short-read mode that restored zero uniquely assigned
    ATP6AP2 isoforms
- name: Trinity
  role: benchmark subject assembler; only two incomplete ATP6AP2 isoforms with unique
    assignments
- name: RNA-Bloom
  role: benchmark subject assembler that failed at the k-mer counting step and is
    tabulated with status 'failure'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:82637c7986043f0768905e6aea13fc145d80607b410268b6e9822e18fbc6b308
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - IsoQuant
  - RNA-Bloom
  - Trinity
  - cloudrnaSPAdes
  - pandas
  - rnaSPAdes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# benchmark-result-tabulation

## Summary

Tabulates a multi-assembler benchmark into a tidy per-tool result table by parsing IsoQuant transcript-assignment outputs, filtering to a single gene locus (ATP6AP2 on chromosome X, 34 GENCODE v32 isoforms), and counting uniquely assigned isoforms while distinguishing pipeline failures from zero recovery.

## When to use

Use when comparing isoform-assembly tools on the same dataset and you need a machine-readable, validated summary table (e.g. assembler vs. uniquely_assigned_isoforms) derived from IsoQuant assignment files.

## When NOT to use

- When IsoQuant assignment outputs do not yet exist for all assemblers — this skill tabulates post-assignment results, it does not run the assemblers or the assignment step itself
- When ambiguous multi-isoform IsoQuant assignments should be included in the count — this skill counts only unambiguous single-isoform assignments
- When a failed run should legitimately be scored as zero (e.g. a completed run that recovered no isoforms) — here a pipeline failure (RNA-Bloom, k-mer counting step) must be recorded as 'failure', not 0

## Inputs

- IsoQuant transcript-assignment output files, one per assembler (cloudrnaSPAdes, Trinity, rnaSPAdes, RNA-Bloom)
- GENCODE v32 GTF annotation for chromosome X (34 annotated ATP6AP2 isoforms)
- Assembly FASTA/contig outputs from each assembler, as submitted to IsoQuant

## Outputs

- atp6ap2_isoform_assignments.csv with columns assembler, uniquely_assigned_isoforms, status
- Per-assembler unique-isoform counts (cloudrnaSPAdes 7, Trinity 2, rnaSPAdes 0, RNA-Bloom failure)

## How to apply

Load the IsoQuant assignment output files generated for each assembler assembly (cloudrnaSPAdes, Trinity, rnaSPAdes, RNA-Bloom) against the GENCODE v32 annotation, together with the GENCODE v32 GTF for chromosome X. Parse each IsoQuant output with pandas and filter transcript assignments to the ATP6AP2 gene locus, restricting matches to the 34 annotated ATP6AP2 isoforms. For each assembler run, count the GENCODE ATP6AP2 isoforms that received a unique (unambiguous single-isoform) assignment — do not count ambiguous multi-isoform assignments. Handle a failed run (e.g. RNA-Bloom, which failed at the k-mer counting step and produced no assignments) as a pipeline failure status rather than as zero recovered isoforms, so that 'no output' is never conflated with 'assembled nothing'. Compile the per-assembler counts (cloudrnaSPAdes 7, Trinity 2, rnaSPAdes 0, RNA-Bloom failure) into a tidy table with columns assembler, uniquely_assigned_isoforms, status and save it as atp6ap2_isoform_assignments.csv, then validate the counts against the expected values.

## Related tools

- **IsoQuant** (produces the per-assembler transcript-assignment outputs against GENCODE v32 that are parsed and filtered to the ATP6AP2 locus)
- **pandas** (parses IsoQuant assignment tables and computes per-assembler counts of uniquely assigned ATP6AP2 isoforms)
- **cloudrnaSPAdes** (benchmark subject assembler whose seven unique ATP6AP2 isoform assignments top the tabulated results) — https://github.com/1dayac/cloudrnaSPAdes-release
- **rnaSPAdes** (benchmark subject assembler; short-read mode that restored zero uniquely assigned ATP6AP2 isoforms)
- **Trinity** (benchmark subject assembler; only two incomplete ATP6AP2 isoforms with unique assignments)
- **RNA-Bloom** (benchmark subject assembler that failed at the k-mer counting step and is tabulated with status 'failure')

## Examples

```
import pandas as pd; df = pd.read_csv('cloudrnaSPAdes_isoquant_assignments.tsv', sep='\t'); atp = df[df.gene_id == 'ATP6AP2']; print(atp[atp.assignment_type == 'unique'].isoform_id.nunique())
```

## Evaluation signals

- Per-assembler unique-isoform counts must equal the reported values of 7 (cloudrnaSPAdes), 2 (Trinity), 0 (rnaSPAdes) and 'failure' (RNA-Bloom), otherwise the output is rejected
- Output CSV must contain exactly the columns assembler, uniquely_assigned_isoforms, status
- RNA-Bloom row must carry a failure status, not the integer 0, since no assignments were available
- Each per-assembler count must be an integer between 0 and 34 (the number of GENCODE v32 ATP6AP2 isoforms on chromosome X)

## Limitations

- RNA-Bloom failed at the k-mer counting step and produced no assignments, so a missing row must be recorded as pipeline failure rather than zero recovered isoforms
- Trinity's two uniquely assigned ATP6AP2 isoforms were incomplete, so the tabulated count does not capture assembly completeness
- Counts are specific to unique IsoQuant assignments against GENCODE v32 for the SPIso-seq human brain dataset; they do not generalize to other annotations, genes, or datasets

## Evidence

- [other] cloudrnaSPAdes assembled seven isoforms of ATP6AP2 with unique IsoQuant assignments, whereas rnaSPAdes restored none and Trinity only two incomplete isoforms with unique assignments (RNA-Bloom failed at the k-mer counting step).: [evidence span withheld — anchor status: not_found]
- [other] Parse each IsoQuant output with pandas and filter transcript assignments to the ATP6AP2 gene locus, restricting matches to the 34 annotated ATP6AP2 isoforms.: [evidence span withheld — anchor status: not_found]
- [abstract] cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity, evaluated on simulated and real human data: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
- [readme] k-mer counting stand-alone binary in SPAdes package, the stage at which the RNA-Bloom benchmark subject failed: [evidence span withheld — anchor status: not_found]
