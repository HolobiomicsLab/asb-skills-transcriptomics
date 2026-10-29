---
name: isoform-assignment-parsing-from-alignment-tools
description: Use when comparing de novo transcriptome assemblers by counting uniquely
  assigned reference isoforms from IsoQuant assignment output files against a GENCODE
  annotation for a specific gene locus.
when_to_use_negative:
- No IsoQuant assignment output files exist and only raw read alignments are available
  — counts must come from IsoQuant assignments, not from re-annotating BAMs.
- The assembler failed mid-pipeline (e.g. RNA-Bloom at the k-mer counting step) —
  record it as a failure status rather than running the counter on empty outputs and
  reporting zero isoforms.
- The reference annotation is not GENCODE v32 (or the locus's isoform set differs)
  — the counts are defined against the 34 annotated ATP6AP2 isoforms and are not comparable
  across annotation versions.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: IsoQuant
  role: Produces the transcript-assignment output files against GENCODE v32 that are
    parsed and filtered per assembler
- name: cloudrnaSPAdes
  role: Assembler whose assembly yields the highest count (7 uniquely assigned ATP6AP2
    isoforms) in the comparison
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: rnaSPAdes
  role: Standard short-read assembler included in the comparison (0 uniquely assigned
    ATP6AP2 isoforms)
- name: Trinity
  role: Short-read assembler included in the comparison (2 incomplete isoforms with
    unique assignments)
- name: RNA-Bloom
  role: Assembler whose run failed at the k-mer counting step; recorded as pipeline
    failure in the status column
- name: pandas
  role: Used to parse each IsoQuant assignment output and filter assignments to the
    ATP6AP2 gene locus
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:da105a30267c834deccbd2aa143d9dc5727fee5830acc25478f058cef5ab39b8
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# isoform-assignment-parsing-from-alignment-tools

## Summary

Parse IsoQuant transcript-assignment output files generated for multiple de novo transcriptome assemblies and count how many annotated GENCODE isoforms of a target gene locus (e.g. ATP6AP2 on chromosome X, 34 GENCODE v32 isoforms) received a unique, unambiguous assignment per assembler.

## When to use

Use when comparing de novo transcriptome assemblers by counting uniquely assigned reference isoforms from IsoQuant assignment output files against a GENCODE annotation for a specific gene locus.

## When NOT to use

- No IsoQuant assignment output files exist and only raw read alignments are available — counts must come from IsoQuant assignments, not from re-annotating BAMs.
- The assembler failed mid-pipeline (e.g. RNA-Bloom at the k-mer counting step) — record it as a failure status rather than running the counter on empty outputs and reporting zero isoforms.
- The reference annotation is not GENCODE v32 (or the locus's isoform set differs) — the counts are defined against the 34 annotated ATP6AP2 isoforms and are not comparable across annotation versions.

## Inputs

- IsoQuant transcript-assignment output files generated for each assembler assembly (cloudrnaSPAdes, Trinity, rnaSPAdes, RNA-Bloom) against GENCODE v32
- GENCODE v32 GTF annotation for chromosome X (defining the 34 ATP6AP2 isoforms)

## Outputs

- atp6ap2_isoform_assignments.csv — tidy table with columns assembler, uniquely_assigned_isoforms, status (cloudrnaSPAdes 7, Trinity 2, rnaSPAdes 0, RNA-Bloom failure)

## How to apply

Load the IsoQuant assignment output files produced for each assembler assembly (cloudrnaSPAdes, Trinity, rnaSPAdes, RNA-Bloom) against the GENCODE v32 annotation, together with the GENCODE v32 GTF for chromosome X. Parse each IsoQuant output with pandas and filter transcript assignments to the ATP6AP2 gene locus, restricting matches to the 34 annotated ATP6AP2 isoforms. For each assembler run, count the GENCODE ATP6AP2 isoforms that received a unique (unambiguous single-isoform) assignment. Treat a failed run — e.g. RNA-Bloom failing at the k-mer counting step — as a pipeline failure recorded in the status column, never as zero recovered isoforms. Compile per-assembler counts (cloudrnaSPAdes 7, Trinity 2, rnaSPAdes 0, RNA-Bloom failure) into a tidy table with columns assembler, uniquely_assigned_isoforms, status, and save as atp6ap2_isoform_assignments.csv.

## Related tools

- **IsoQuant** (Produces the transcript-assignment output files against GENCODE v32 that are parsed and filtered per assembler)
- **cloudrnaSPAdes** (Assembler whose assembly yields the highest count (7 uniquely assigned ATP6AP2 isoforms) in the comparison) — https://github.com/1dayac/cloudrnaSPAdes-release
- **rnaSPAdes** (Standard short-read assembler included in the comparison (0 uniquely assigned ATP6AP2 isoforms))
- **Trinity** (Short-read assembler included in the comparison (2 incomplete isoforms with unique assignments))
- **RNA-Bloom** (Assembler whose run failed at the k-mer counting step; recorded as pipeline failure in the status column)
- **pandas** (Used to parse each IsoQuant assignment output and filter assignments to the ATP6AP2 gene locus)

## Examples

```
import pandas as pd; df = pd.read_csv('isoquant_assignments_cloudrnaSPAdes.tsv', sep='\t'); atp6ap2 = df[df['gene'].str.contains('ATP6AP2')]
```

## Evaluation signals

- Per-assembler unique-isoform counts must equal the reported values of 7 (cloudrnaSPAdes), 2 (Trinity), and 0 (rnaSPAdes), with RNA-Bloom recorded as failure — otherwise the output is rejected.
- Output CSV must contain exactly the columns assembler, uniquely_assigned_isoforms, status and one row per assembler run.
- Only GENCODE v32 isoforms of the ATP6AP2 locus (34 annotated isoforms) may be counted; ambiguous multi-isoform assignments are excluded.
- A zero count (rnaSPAdes) must be distinguished from a pipeline failure (RNA-Bloom) in the status column.
- All four assembler runs appear in the table, including the failed run.

## Limitations

- RNA-Bloom failed at the k-mer counting step, so no IsoQuant assignments exist for it — it must be recorded as a pipeline failure rather than as zero recovered isoforms.
- Counts are contingent on the GENCODE v32 annotation and IsoQuant's assignment stringency; different annotation versions or assignment tools will change the isoform universe and counts.
- The case study is gene-specific (ATP6AP2, chromosome X); results do not generalize to other loci without re-running the parsing workflow on those genes.

## Evidence

- [abstract] we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [abstract] cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
- [methods] BayesHammer recommendation: "We recommend to run SPAdes with BayesHammer/IonHammer to obtain high-quality assemblies."
- [readme] rnaSPAdes description: "rnaSPAdes &ndash; a *de novo* transcriptome assembler from RNA-Seq data"
- [readme] SPAdes toolkit description: "SPAdes &ndash; St. Petersburg genome assembler &ndash; is an assembly toolkit containing various assembly pipelines"
