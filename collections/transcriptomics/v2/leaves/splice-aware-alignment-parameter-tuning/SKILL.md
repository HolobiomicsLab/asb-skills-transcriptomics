---
name: splice-aware-alignment-parameter-tuning
description: Use when building or adapting a Snakemake neoantigen/variant pipeline
  that must align RNA-seq reads splice- and chimeric-aware (for fusions, exitrons,
  indels) and DNA-seq reads genome-wide, with defined STAR chimera parameters and
  per-branch BAM indexing.
when_to_use_negative:
- Input is raw, unpreprocessed FASTQ with adapters/low-quality tails — run fastp pre-processing
  first.
- The downstream goal is HLA genotyping — ScanNeo2 instead filters reads with yara
  and aligns against HLA alleles with Optitype/HLA-HD, not the genome-alignment branch.
- Only short-read SNV/indel calling on DNA-seq is needed with no RNA-seq branch —
  a plain BWA-MEM alignment without the STAR chimeric configuration suffices.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: STAR
  role: Chimeric-aware spliced aligner for RNA-seq reads, producing BAMs with chimera
    parameters tuned for fusion/exitron detection (v2.7.10b settings)
  repo: https://github.com/alexdobin/STAR
- name: BWA
  role: Aligns DNA-seq reads with BWA-MEM (mapq 30) and re-aligns STAR-produced RNA
    BAMs to emit BWA-style CIGARs for transIndel and ScanExitron
- name: Snakemake
  role: Workflow engine that fans per-(sample, seqtype, group) alignment branches
    across cores/nodes and enforces the realign and postproc_bam_index rules
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: Arriba
  role: Downstream consumer of the chimeric-aware STAR alignments for gene fusion
    detection
  repo: https://github.com/suhrig/arriba
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:e4dcec29ad2d0737c8bc56ae565cbfd24d845f389faeba7c3dea7dfce3303b24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexdobin/STAR
  tools:
  - Arriba
  - BWA
  - STAR
  - ScanNeo2
  - Snakemake
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# splice-aware-alignment-parameter-tuning

## Summary

Configure chimeric-aware STAR alignment for RNA-seq alongside BWA-MEM alignment for DNA-seq in the ScanNeo2 workflow, including a BWA realignment step for STAR-produced BAMs so downstream variant callers (transIndel, ScanExitron) see a BWA-style CIGAR.

## When to use

Use when building or adapting a Snakemake neoantigen/variant pipeline that must align RNA-seq reads splice- and chimeric-aware (for fusions, exitrons, indels) and DNA-seq reads genome-wide, with defined STAR chimera parameters and per-branch BAM indexing.

## When NOT to use

- Input is raw, unpreprocessed FASTQ with adapters/low-quality tails — run fastp pre-processing first.
- The downstream goal is HLA genotyping — ScanNeo2 instead filters reads with yara and aligns against HLA alleles with Optitype/HLA-HD, not the genome-alignment branch.
- Only short-read SNV/indel calling on DNA-seq is needed with no RNA-seq branch — a plain BWA-MEM alignment without the STAR chimeric configuration suffices.

## Inputs

- Preprocessed (fastp-trimmed) RNA-seq FASTQ read groups
- Preprocessed DNA-seq (WES) FASTQ read groups
- Reference genome (ENSEMBL release, configured via the 'release' option)
- Sample sheet with (sample, seqtype, group) rows

## Outputs

- Aligned DNA-seq BAMs in results/<sample>/dnaseq/align/
- STAR chimeric-aware RNA-seq BAMs plus BWA-realigned RNA-seq BAMs in results/<sample>/rnaseq/align/
- .bai index files alongside every aligned BAM
- Intermediate STAR BAMs retained alongside final realigned BAMs

## How to apply

Launch the Snakemake workflow so independent per-group alignment branches fan out across cores/nodes for every (sample, seqtype, group) row in the sample sheet. Align DNA-seq read groups directly with BWA-MEM using a global mapping-quality threshold of mapq: 30, and align RNA-seq read groups with STAR in chimeric-aware mode using chimSegmentMin: 20, chimScoreMin: 10, chimJunctionOverhangMin: 10, chimScoreDropMax: 30, and chimScoreSeparation: 10 (parameters identical to STAR v2.7.10b). Because downstream RNA variant callers (transIndel, ScanExitron) require a BWA-style CIGAR, run a 'realign' rule that re-aligns the STAR-produced BAM with BWA, keeping both intermediates and the final BAM. Finish by writing .bai index files next to every aligned BAM via the postproc_bam_index rule. The rationale is that chimeric detection in STAR captures gene fusions and exitron splicing needed for neoantigen discovery, while BWA realignment preserves caller compatibility.

## Related tools

- **STAR** (Chimeric-aware spliced aligner for RNA-seq reads, producing BAMs with chimera parameters tuned for fusion/exitron detection (v2.7.10b settings)) — https://github.com/alexdobin/STAR
- **BWA** (Aligns DNA-seq reads with BWA-MEM (mapq 30) and re-aligns STAR-produced RNA BAMs to emit BWA-style CIGARs for transIndel and ScanExitron)
- **Snakemake** (Workflow engine that fans per-(sample, seqtype, group) alignment branches across cores/nodes and enforces the realign and postproc_bam_index rules) — https://github.com/ylab-hi/ScanNeo2/
- **Arriba** (Downstream consumer of the chimeric-aware STAR alignments for gene fusion detection) — https://github.com/suhrig/arriba

## Examples

```
snakemake --use-conda --cores 16 results/<sample>/rnaseq/align/<sample>.bam results/<sample>/dnaseq/align/<sample>.bam
```

## Evaluation signals

- results/<sample>/dnaseq/align/ and results/<sample>/rnaseq/align/ contain BAMs each paired with a .bai index (postproc_bam_index rule completed).
- STAR command line reflects all five chimera parameters: chimSegmentMin 20, chimScoreMin 10, chimJunctionOverhangMin 10, chimScoreDropMax 30, chimScoreSeparation 10.
- DNA-seq BAMs were filtered at mapq 30.
- For each RNA-seq sample, both the STAR intermediate BAM and the BWA-realigned final BAM are present ('realign' rule kept intermediates).
- Realigned RNA BAM CIGARs are BWA-style (as required by transIndel and ScanExitron); those callers run without CIGAR-format errors.

## Limitations

- The STAR chimeric parameters match STAR v2.7.10b specifically; other STAR versions may behave differently.
- Keeping both STAR and BWA-realigned RNA BAM intermediates increases storage footprint.
- The article text notes no specific Snakemake version is pinned in the manuscript; the repository README indicates snakemake ≥9.0.0.
- Some README-derived documentation (e.g. the unrelated transcriptase-atom repo) surfaced alongside the skill and should not be treated as ScanNeo2 evidence.

## Evidence

- [methods] Chimeric-aware STAR parameters for RNA-seq alignment: [evidence span withheld — anchor status: not_found]
- [methods] BWA realignment rationale for RNA variant callers: [evidence span withheld — anchor status: not_found]
- [methods] DNA-seq mapping quality threshold: [evidence span withheld — anchor status: not_found]
- [methods] STAR reference in article methods: "Please refer to the [STAR manual](https://github.com/alexdobin/STAR/blob/STAR_2.7.10b_alpha_230301/doc/STARmanual.pdf)"
- [readme] Arriba builds on STAR alignments: [evidence span withheld — anchor status: external_doc]
