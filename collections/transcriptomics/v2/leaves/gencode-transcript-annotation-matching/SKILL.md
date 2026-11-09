---
name: gencode-transcript-annotation-matching
description: Use when comparing de novo isoform assemblies (e.g. cloudrnaSPAdes, rnaSPAdes,
  Trinity, RNA-Bloom) by counting uniquely assigned GENCODE annotated isoforms per
  gene from IsoQuant assignment output files.
when_to_use_negative:
- The assembler never produced an assembly (e.g. RNA-Bloom failed at the k-mer counting
  step) — record it as a pipeline failure, not as zero recovered isoforms.
- The gene of interest has few or no annotated GENCODE isoforms, so 'unique isoform
  recovery' is not a discriminative benchmark between assemblers.
- Assemblies should be assessed on full-length transcript quality metrics rather than
  annotated-isoform recall — use a dedicated assembly evaluation tool instead.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: IsoQuant
  role: Assigns assembled transcripts to GENCODE v32 annotated isoforms; its assignment
    output files are parsed and filtered per gene
- name: pandas
  role: Parses IsoQuant assignment outputs and filters/count unique single-isoform
    assignments at the ATP6AP2 locus
- name: cloudrnaSPAdes
  role: Reference-free assembler of full-length isoforms from barcoded RNA-seq linked
    reads whose assignments are benchmarked
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: rnaSPAdes
  role: Standard short-read transcriptome assembler used as the comparison baseline
    (restored 0 ATP6AP2 isoforms)
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: Trinity
  role: Short-read transcriptome assembler used as a comparison baseline (2 incomplete
    isoforms with unique assignments)
- name: RNA-Bloom
  role: Short-read transcriptome assembler whose run failed at the k-mer counting
    step and is recorded as a failure
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:bdfc387b17aaac01cecafe1f2c70fc576b1dedc75e7aae6a2eef1efd3a3df9ab
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gencode-transcript-annotation-matching

## Summary

Counts how many annotated GENCODE isoforms of a target gene are uniquely recovered by each transcriptome assembler by parsing IsoQuant assignment outputs against the GENCODE v32 GTF, as demonstrated on the ATP6AP2 gene (34 isoforms, chromosome X) from the real SPIso-seq human brain dataset.

## When to use

Use when comparing de novo isoform assemblies (e.g. cloudrnaSPAdes, rnaSPAdes, Trinity, RNA-Bloom) by counting uniquely assigned GENCODE annotated isoforms per gene from IsoQuant assignment output files.

## When NOT to use

- The assembler never produced an assembly (e.g. RNA-Bloom failed at the k-mer counting step) — record it as a pipeline failure, not as zero recovered isoforms.
- The gene of interest has few or no annotated GENCODE isoforms, so 'unique isoform recovery' is not a discriminative benchmark between assemblers.
- Assemblies should be assessed on full-length transcript quality metrics rather than annotated-isoform recall — use a dedicated assembly evaluation tool instead.

## Inputs

- IsoQuant assignment output files for each assembler's transcript assembly, produced against the GENCODE v32 annotation
- GENCODE v32 GTF annotation for chromosome X (containing the 34 annotated ATP6AP2 isoforms)
- Transcript assemblies from cloudrnaSPAdes, rnaSPAdes, Trinity, and RNA-Bloom runs on the SPIso-seq human brain dataset

## Outputs

- atp6ap2_isoform_assignments.csv: tidy table with columns assembler, uniquely_assigned_isoforms, status (cloudrnaSPAdes 7, Trinity 2, rnaSPAdes 0, RNA-Bloom failure)

## How to apply

Load the IsoQuant assignment output files generated for each assembler's assembly (here cloudrnaSPAdes, rnaSPAdes, Trinity, and RNA-Bloom runs) together with the GENCODE v32 GTF for the locus of interest. Parse each IsoQuant output with pandas and filter transcript assignments to the target gene locus, restricting matches to the annotated isoform set (34 ATP6AP2 isoforms in GENCODE v32). For each assembler run, count the annotated isoforms that received a unique (unambiguous single-isoform) assignment; multi-isoform assignments must not be counted. Treat assembler pipeline failures explicitly — the RNA-Bloom run died at the k-mer counting step, so it is recorded as a pipeline failure rather than as zero recovered isoforms. Compile per-assembler counts (cloudrnaSPAdes 7, Trinity 2 incomplete isoforms, rnaSPAdes 0, RNA-Bloom failure) into a tidy table with columns assembler, uniquely_assigned_isoforms, status and save as atp6ap2_isoform_assignments.csv.

## Related tools

- **IsoQuant** (Assigns assembled transcripts to GENCODE v32 annotated isoforms; its assignment output files are parsed and filtered per gene)
- **pandas** (Parses IsoQuant assignment outputs and filters/count unique single-isoform assignments at the ATP6AP2 locus)
- **cloudrnaSPAdes** (Reference-free assembler of full-length isoforms from barcoded RNA-seq linked reads whose assignments are benchmarked) — https://github.com/1dayac/cloudrnaSPAdes-release
- **rnaSPAdes** (Standard short-read transcriptome assembler used as the comparison baseline (restored 0 ATP6AP2 isoforms)) — https://github.com/1dayac/cloudrnaSPAdes-release
- **Trinity** (Short-read transcriptome assembler used as a comparison baseline (2 incomplete isoforms with unique assignments))
- **RNA-Bloom** (Short-read transcriptome assembler whose run failed at the k-mer counting step and is recorded as a failure)

## Examples

```
assignments = pd.read_csv("cloudrnaSPAdes_isoquant_assignments.tsv", sep="\t"); atp = assignments[assignments["gene"] == "ATP6AP2"]; unique = atp[atp["gencode_isoform"].isin(annotated_34_isoforms)].groupby("assembled_transcript").filter(lambda g: g["gencode_isoform"].nunique() == 1)
```

## Evaluation signals

- Per-assembler unique-isoform counts must equal the reported values: cloudrnaSPAdes 7, Trinity 2, rnaSPAdes 0, RNA-Bloom failure — otherwise the output is rejected.
- Only assignments restricted to the 34 annotated ATP6AP2 isoforms in GENCODE v32 are counted; multi-isoform (ambiguous) assignments are excluded.
- The output CSV has exactly the columns assembler, uniquely_assigned_isoforms, status.
- RNA-Bloom must appear with a failure/pipeline-failure status, not as 0 uniquely assigned isoforms.

## Limitations

- RNA-Bloom failed at the k-mer counting step on this dataset, so its result is a pipeline failure rather than a measure of isoform recovery.
- Trinity's two unique assignments correspond to incomplete isoforms, so unique-assignment counts alone do not indicate full-length recovery.
- Unique-assignment counting depends on IsoQuant's assignment stringency; ambiguous multi-isoform assignments are discarded and thus understate total isoform overlap.

## Evidence

- [results] cloudrnaSPAdes assembled seven isoforms of ATP6AP2 with unique IsoQuant assignments, whereas rnaSPAdes restored none and Trinity only two incomplete isoforms with unique assignments (RNA-Bloom failed at the k-mer counting step).: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract claim that isoform assembly is accurate for diverse genes.: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
- [readme] SPAdes toolkit description grounding the assembler ecosystem used in the benchmark.: "SPAdes &ndash; St. Petersburg genome assembler &ndash; is an assembly toolkit containing various assembly pipelines"
