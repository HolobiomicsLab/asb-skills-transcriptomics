---
name: single-cell-barcode-umi-position-indexing
description: Use when FASTQ reads must be routed into a single-cell preprocessing
  tool (STARsolo, kb/kallisto bustools) that needs explicit barcode/UMI/cDNA start,end,strand
  positions, and an assay spec.yaml exists or can be authored.
when_to_use_negative:
- Read structure is already known and hardcoded for a standard 10x Genomics chemistry
  supported natively by the downstream tool — a hand-written whitelist/positions config
  suffices
- Only an unannotated assay is available and no seqspec YAML can be built or validated
  — index output would be meaningless until the spec is authored
- Data are not single-cell genomics reads (e.g. bulk RNA-seq or ATAC fragment files
  with no barcode/UMI structure)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: seqspec
  role: Reads the assay spec.yaml, positionally indexes barcode/UMI/cDNA regions,
    and emits the per-tool technology string via `seqspec index`
  repo: https://github.com/IGVF/seqspec
- name: kb-python
  role: Consumer of the kb-format technology string; couples kallisto and bustools
    to perform quantification
- name: cellatlas
  role: Universal preprocessing framework that uses seqspec with kallisto bustools
    for single-cell preprocessing
  repo: https://github.com/cellatlas/cellatlas.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae168
    title: A machine-readable specification for genomics assays
  source_content_hash: sha256:f33d6af78d307c748dbbe823d3e174425e5b5cc5d200d1ac791f82587a658efa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IGVF/seqspec
  tools:
  - cellatlas
  - kb-python
  - seqspec
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# single-cell-barcode-umi-position-indexing

## Summary

Generates tool-specific technology strings (positional annotations of barcode, UMI, and cDNA regions per FASTQ read) from a machine-readable seqspec assay file, enabling STARsolo- or kb-compatible preprocessing of single-cell RNA-seq data without hand-crafted parsing scripts.

## When to use

Use when FASTQ reads must be routed into a single-cell preprocessing tool (STARsolo, kb/kallisto bustools) that needs explicit barcode/UMI/cDNA start,end,strand positions, and an assay spec.yaml exists or can be authored.

## When NOT to use

- Read structure is already known and hardcoded for a standard 10x Genomics chemistry supported natively by the downstream tool — a hand-written whitelist/positions config suffices
- Only an unannotated assay is available and no seqspec YAML can be built or validated — index output would be meaningless until the spec is authored
- Data are not single-cell genomics reads (e.g. bulk RNA-seq or ATAC fragment files with no barcode/UMI structure)

## Inputs

- seqspec assay specification YAML (spec.yaml) declaring barcode, UMI, and cDNA Regions per modality
- FASTQ read pairs per modality (e.g. fastqs/rna_R1_SRR18677638.fastq.gz, rna_R2_SRR18677638.fastq.gz)
- target tool name for the technology string format (-t kb or STARsolo)

## Outputs

- tool-specific technology string of colon-separated start,end,strand triplets (e.g. 0,0,16:0,16,28:1,0,102)
- text file capturing the string (e.g. rna_technology_string.txt) for downstream STARsolo/kb consumption

## How to apply

Author or obtain a validated seqspec YAML (e.g. specs/dogmaseq-dig/spec.yaml from github.com/IGVF/seqspec) that declares the library structure per modality (rna, tag, atac, protein). Install seqspec with `pip install git+https://github.com/IGVF/seqspec.git`, then run `seqspec index` with the target tool (`-t kb` for kallisto|bustools, STARsolo equivalents also supported), the modality (`-m rna`), and the modality's FASTQ reads supplied positionally (e.g. rna_R1_SRR18677638.fastq.gz,rna_R2_SRR18677638.fastq.gz). The command extracts the barcode, UMI, and cDNA Regions from the spec, positionally indexes them against the supplied reads, and formats the colon-separated start,end,strand triplets on a per-tool basis; for the dogmaseq-dig RNA modality the expected output is 0,0,16:0,16,28:1,0,102. Write the emitted string to a file (e.g. rna_technology_string.txt) and pass it to the downstream tool so barcode extraction, correction, and UMI counting (performed with bustools) parse the reads correctly. Repeat per modality since the positional layout differs for tag/atac/protein feature-barcode reads.

## Related tools

- **seqspec** (Reads the assay spec.yaml, positionally indexes barcode/UMI/cDNA regions, and emits the per-tool technology string via `seqspec index`) — https://github.com/IGVF/seqspec
- **kb-python** (Consumer of the kb-format technology string; couples kallisto and bustools to perform quantification)
- **cellatlas** (Universal preprocessing framework that uses seqspec with kallisto bustools for single-cell preprocessing) — https://github.com/cellatlas/cellatlas.git

## Examples

```
seqspec index -t kb -m rna -r fastqs/rna_R1_SRR18677638.fastq.gz,rna_R2_SRR18677638.fastq.gz spec.yaml
```

## Evaluation signals

- Output string for the dogmaseq-dig RNA modality exactly equals 0,0,16:0,16,28:1,0,102 (colon-separated start,end,strand triplets per read)
- Number of comma-separated triplets equals the number of FASTQ reads supplied via -r for the modality
- Each triplet's end position equals that region's length in the spec (barcode 0–16, UMI 16–28, cDNA read spanning the 102 bp read)
- The generated string is accepted by the target tool (-t kb) without technology-string parse errors during kb count / cellatlas preprocessing
- Running the same command for tag/atac/protein modalities (e.g. tag_R1_SRR18677640.fastq.gz) yields modality-specific strings, confirming per-modality indexing rather than a reused RNA layout

## Limitations

- seqspec index requires a correctly authored and validated spec.yaml — an incorrect library structure in the specification produces silently wrong positional annotations
- The SRA data-acquisition section of the source notebook is incomplete ('todo: prefetch and fastq-dump'), so obtaining raw reads may require extra steps not covered here
- Formats are tool-specific: a string generated for `-t kb` must be regenerated if switching to STARsolo rather than reused

## Evidence

- [methods] seqspec index workflow grounding: "pip install git+https://github.com/IGVF/seqspec.git"
- [methods] spec file provenance: "A sample dataset derived from the original data can be found here: https://github.com/IGVF/seqspec/tree/main/specs/dogmaseq-dig"
- [intro] downstream consumption of seqspec technology strings: "The cellatlas tool uses seqspec and kallisto bustools to facilitate universal preprocessing of single-cell genomics data."
- [methods] purpose of the technology string in preprocessing: "1. Sequencing reads are aligned to a reference
2. Barcodes errors are corrected
3. UMIs/reads are counted"
- [readme] seqspec repository layout: [evidence span withheld — anchor status: external_doc]
