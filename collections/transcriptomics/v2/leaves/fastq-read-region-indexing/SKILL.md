---
name: fastq-read-region-indexing
description: Use when preprocessing single-cell genomics FASTQs (e.g. dogmaseq-dig
  RNA/tag/atac/protein splits) and you need the per-tool technology string describing
  where barcodes, UMIs, and cDNA lie in R1/R2/R3 before kb count or alevin-fry.
when_to_use_negative:
- The library structure is unknown or no seqspec spec.yaml exists for the assay —
  the geometry cannot be derived without a machine-readable specification.
- Input is already a cell-by-feature count matrix (post-preprocessing); geometry indexing
  applies only to raw FASTQ read structure, not to downstream matrix steps like mx
  filter/normalize.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: seqspec
  role: Parses the assay spec.yaml and emits the per-tool read geometry string via
    `seqspec index` with -t/-m/-r options
  repo: https://github.com/IGVF/seqspec
- name: kb-python
  role: Downstream consumer of the kb-format geometry string for quantification with
    kallisto/bustools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae168
    title: A machine-readable specification for genomics assays
  source_content_hash: sha256:625d8f1b7ae8e0b832bcc37311ffa446dee019f174e415a09c5eba8b3bd542dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IGVF/seqspec
  tools:
  - kb-python
  - seqspec
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# fastq-read-region-indexing

## Summary

Generates tool-compatible read geometry strings (barcode/UMI/cDNA region positions per FASTQ read) from a machine-readable seqspec assay specification using `seqspec index`, as required by preprocessing tools like kb and simpleaf/alevin-fry.

## When to use

Use when preprocessing single-cell genomics FASTQs (e.g. dogmaseq-dig RNA/tag/atac/protein splits) and you need the per-tool technology string describing where barcodes, UMIs, and cDNA lie in R1/R2/R3 before kb count or alevin-fry.

## When NOT to use

- The library structure is unknown or no seqspec spec.yaml exists for the assay — the geometry cannot be derived without a machine-readable specification.
- Input is already a cell-by-feature count matrix (post-preprocessing); geometry indexing applies only to raw FASTQ read structure, not to downstream matrix steps like mx filter/normalize.

## Inputs

- seqspec assay specification YAML (e.g. specs/dogmaseq-dig/spec.yaml from IGVF/seqspec)
- Modality-specific FASTQ files (e.g. rna_R1/R2_SRR18677638.fastq.gz, tag_R1/R2_SRR18677640.fastq.gz, atac_R1/R2/R3_SRR18677642.fastq.gz, protein_R1/R2_SRR18677644.fastq.gz)
- Target tool selector (-t, e.g. kb or simpleaf/alevin-fry) and modality selector (-m)

## Outputs

- Tool-compatible read geometry/technology string per modality and tool (e.g. 0,0,16:0,16,28:1,0,102 for RNA-kb; 1,8,24:-1,-1,-1:0,0,52,2,0,52 for ATAC-kb)
- read_geometry.txt with one labeled line per modality/tool combination

## How to apply

First obtain the assay's seqspec YAML (e.g. clone https://github.com/IGVF/seqspec.git and use specs/dogmaseq-dig/spec.yaml, which names the FASTQ files). For each modality, run `seqspec index` with `-t` selecting the target tool format (kb or simpleaf/alevin-fry), `-m` selecting the modality (rna, tag, protein, atac), and `-r` supplying the comma-separated FASTQ files for that modality (e.g. fastqs/rna_R1_SRR18677638.fastq.gz,rna_R2_SRR18677638.fastq.gz). The command extracts regions annotated as barcode, UMI, or cDNA in the spec, positionally indexes them against the supplied reads, and formats the geometry per tool. Verify expected outputs: RNA kb string 0,0,16:0,16,28:1,0,102; tag and protein kb strings both 0,0,16:0,16,28:1,0,15; atac (3 reads) 1,8,24:-1,-1,-1:0,0,52,2,0,52. Re-run with the simpleaf/alevin-fry target to get the alternative rendering, and write one labeled line per modality/tool to read_geometry.txt.

## Related tools

- **seqspec** (Parses the assay spec.yaml and emits the per-tool read geometry string via `seqspec index` with -t/-m/-r options) — https://github.com/IGVF/seqspec
- **kb-python** (Downstream consumer of the kb-format geometry string for quantification with kallisto/bustools)

## Examples

```
seqspec index -t kb -m rna -r fastqs/rna_R1_SRR18677638.fastq.gz,rna_R2_SRR18677638.fastq.gz spec.yaml
```

## Evaluation signals

- RNA modality with -t kb returns exactly 0,0,16:0,16,28:1,0,102
- Tag and protein modalities with -t kb both return 0,0,16:0,16,28:1,0,15
- ATAC modality (3 reads) with -t kb returns 1,8,24:-1,-1,-1:0,0,52,2,0,52, including the -1,-1,-1 entry for the read lacking barcode/UMI regions
- Re-running with the simpleaf/alevin-fry target produces the per-tool rendering shown in Fig. 2, distinct from the kb format
- read_geometry.txt contains one labeled line for every modality × tool combination

## Limitations

- Geometry strings are assay- and spec-specific: positions (e.g. 16 bp barcode + 12 bp UMI) are valid only for the dogmaseq-dig sample dataset (SRR18677638/40/42/44); other assays need their own spec.yaml.
- The ATAC modality includes a read with no barcode/UMI regions, encoded as -1,-1,-1; tools consuming the geometry must tolerate such placeholders.
- The article's SRA prefetch/fastq-dump section is marked 'todo: prefetch and fastq-dump', so obtaining the FASTQs upstream is not fully documented.

## Evidence

- [methods] seqspec index supports generating tool-compatible technology strings: Regions associated with barcodes, UMIs, and cDNA are extracted, positionally indexed, and formatted on a per-tool basis (including simpleaf/alevin-fry).: [evidence span withheld — anchor status: not_found]
- [methods] download the repo: "git clone https://github.com/IGVF/seqspec.git"
- [methods] dogmaseq-dig sample dataset: "A sample dataset derived from the original data can be found here: https://github.com/IGVF/seqspec/tree/main/specs/dogmaseq-dig"
- [abstract] seqspec purpose: "We present seqspec, a machine-readable specification for libraries produced by genomics assays that facilitates standardization of preprocessing"
- [methods] read structure understanding: "Before we download, QC, and process the data we must first understand the read structure"
