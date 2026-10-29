---
name: gene-locus-isoform-filtering
description: Use when comparing de novo transcriptome assemblies against a reference
  annotation and you need per-locus counts of uniquely assigned isoforms, especially
  for high-isoform-diversity genes like ATP6AP2.
when_to_use_negative:
- The run produced no assignment output at all — record it as a pipeline failure (e.g.
  RNA-Bloom failing at the k-mer counting step), not as zero uniquely assigned isoforms
- Assignments are multi-isoform/ambiguous — do not count them toward the unique-isoform
  total
- The reference annotation is not the GENCODE version used for the IsoQuant run —
  restrict matches to isoforms present in that annotation (34 GENCODE v32 ATP6AP2
  isoforms)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: IsoQuant
  role: Produces transcript-to-annotation assignment outputs that are parsed and filtered
    per gene locus
- name: cloudrnaSPAdes
  role: Assembler whose assembly's unique isoform assignments at the ATP6AP2 locus
    are counted (7 uniquely assigned isoforms)
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: rnaSPAdes
  role: Comparison assembler whose ATP6AP2 unique-isoform count (0) is tabulated
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: Trinity
  role: Comparison assembler whose ATP6AP2 unique-isoform count (2 incomplete isoforms)
    is tabulated
- name: RNA-Bloom
  role: Comparison assembler; its k-mer counting failure is recorded as a pipeline
    failure status
- name: pandas
  role: Parses IsoQuant output files and filters/count assignments to the ATP6AP2
    locus
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:a64da372de8e5cb867e9bf998719b8d9fc068b6af97b693d62badcf49ce61dfa
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gene-locus-isoform-filtering

## Summary

Parses IsoQuant transcript-to-annotation assignment outputs and filters them to a single gene locus (e.g. ATP6AP2 on chromosome X), counting only unambiguous single-isoform assignments against GENCODE v32 annotations to quantify how many annotated isoforms each assembler recovered.

## When to use

Use when comparing de novo transcriptome assemblies against a reference annotation and you need per-locus counts of uniquely assigned isoforms, especially for high-isoform-diversity genes like ATP6AP2.

## When NOT to use

- The run produced no assignment output at all — record it as a pipeline failure (e.g. RNA-Bloom failing at the k-mer counting step), not as zero uniquely assigned isoforms
- Assignments are multi-isoform/ambiguous — do not count them toward the unique-isoform total
- The reference annotation is not the GENCODE version used for the IsoQuant run — restrict matches to isoforms present in that annotation (34 GENCODE v32 ATP6AP2 isoforms)

## Inputs

- IsoQuant transcript assignment output files (one per assembler: cloudrnaSPAdes, Trinity, rnaSPAdes, RNA-Bloom)
- GENCODE v32 GTF annotation for chromosome X
- Assembled transcript FASTA files from each assembler (context for the assignments)

## Outputs

- Tidy CSV table (e.g. atp6ap2_isoform_assignments.csv) with columns assembler, uniquely_assigned_isoforms, status

## How to apply

Load the IsoQuant assignment output files generated for each assembler assembly (cloudrnaSPAdes, Trinity, rnaSPAdes, RNA-Bloom) against the GENCODE v32 annotation, together with the GENCODE v32 GTF for the relevant chromosome. Parse each IsoQuant output with pandas and filter transcript assignments to the target gene locus (ATP6AP2, chromosome X), restricting matches to the 34 annotated GENCODE v32 ATP6AP2 isoforms. For each assembler run, count the GENCODE isoforms that received a unique (unambiguous single-isoform) assignment rather than ambiguous multi-isoform matches. Treat pipeline failures (e.g. RNA-Bloom failing at the k-mer counting step) as an explicit 'failure' status rather than zero recovered isoforms, so assembler failure is not conflated with poor recovery. Compile the per-assembler counts into a tidy table (columns: assembler, uniquely_assigned_isoforms, status) and save as CSV.

## Related tools

- **IsoQuant** (Produces transcript-to-annotation assignment outputs that are parsed and filtered per gene locus)
- **cloudrnaSPAdes** (Assembler whose assembly's unique isoform assignments at the ATP6AP2 locus are counted (7 uniquely assigned isoforms)) — https://github.com/1dayac/cloudrnaSPAdes-release
- **rnaSPAdes** (Comparison assembler whose ATP6AP2 unique-isoform count (0) is tabulated) — https://github.com/1dayac/cloudrnaSPAdes-release
- **Trinity** (Comparison assembler whose ATP6AP2 unique-isoform count (2 incomplete isoforms) is tabulated)
- **RNA-Bloom** (Comparison assembler; its k-mer counting failure is recorded as a pipeline failure status)
- **pandas** (Parses IsoQuant output files and filters/count assignments to the ATP6AP2 locus)

## Examples

```
df = pd.read_csv('isoquant_atp6ap2_assignments.tsv', sep='\t'); uniq = df[df.isoform_id.isin(gencode_atp6ap2_isoforms)].groupby('assembler')['isoform_id'].nunique(); uniq.to_csv('atp6ap2_isoform_assignments.csv')
```

## Evaluation signals

- Per-assembler unique-isoform counts must equal the reported values: cloudrnaSPAdes 7, Trinity 2, rnaSPAdes 0 — otherwise the output is rejected
- RNA-Bloom must appear with status 'failure' (not 0) because it failed at the k-mer counting step and produced no assignments
- Every counted assignment must map to one of the 34 annotated GENCODE v32 ATP6AP2 isoforms on chromosome X
- Output CSV must be tidy with exactly the columns assembler, uniquely_assigned_isoforms, status

## Limitations

- Trinity's two uniquely assigned ATP6AP2 isoforms were incomplete, so unique-assignment counts alone do not guarantee full-length isoform recovery
- Assembler-level pipeline failures (e.g. RNA-Bloom at the k-mer counting step) cannot be scored as isoform recovery and must be flagged separately
- Counts depend on the GENCODE v32 annotation and IsoQuant assignment stringency; switching annotation versions changes the locus isoform universe

## Evidence

- [abstract] cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
- [other] Per-assembler counts and failure handling: [evidence span withheld — anchor status: not_found]
- [readme] K-mer counting is a pipeline stage: [evidence span withheld — anchor status: not_found]
- [other] IsoQuant assignment parsing method: [evidence span withheld — anchor status: not_found]
