---
name: reditools-pileup-table-generation
description: Use when RNAseq BAM files must be converted into position-wise pileup
  tables of candidate substitutions (e.g. A-to-G events) in the tabix-indexed REDItools
  outTable format consumed by REDInet or REDItools-based editing callers.
when_to_use_negative:
- Input is already a bgzip/tabix-indexed REDItools outTable — proceed directly to
  REDInet_Inference.py instead of re-running pileup generation
- The task is variant calling on DNAseq-only data or non-A-to-I substitution discovery
  where low-stringency RNAseq pileups are not the appropriate representation
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0085
tools:
- name: REDItools
  role: Suite of Python scripts (REDItoolDnaRna.py) run at low stringency to generate
    the variant/invariant RNAseq pileup outTables
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: REDItools v1
  role: Stable REDItools version used for the low-stringency pileup step (~9 hours
    vs <60 minutes for v3 on identical jobs)
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: REDItools3
  role: Speed-optimized REDItools reimplementation that can generate the same pileup
    profiles in under 60 minutes
  repo: https://github.com/BioinfoUNIBA/REDItools3
- name: pysam
  role: Python SAM/BAM library wrapping the samtools C-API used underneath the REDItools
    pileup implementation
  repo: https://github.com/pysam-developers/pysam
- name: Python
  role: Runtime environment (Python 3.9.0 conda env) in which REDItools scripts are
    executed
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:0c871728f0e1076e4b207a080f99b4e4ebdebd86b1d20a584fcf8088c9409a68
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/BioinfoUNIBA/REDItools
  tools:
  - Python
  - REDInet
  - REDItools
  - REDItools v1
  - REDItools3
  - pysam
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# reditools-pileup-table-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate variant/invariant RNAseq pileup tables with REDItools (REDItoolDnaRna.py at low stringency) and compress/index them with bgzip+tabix, producing the REDItools outTable format that downstream classifiers like REDInet require as input.

## When to use

Use when RNAseq BAM files must be converted into position-wise pileup tables of candidate substitutions (e.g. A-to-G events) in the tabix-indexed REDItools outTable format consumed by REDInet or REDItools-based editing callers.

## When NOT to use

- Input is already a bgzip/tabix-indexed REDItools outTable — proceed directly to REDInet_Inference.py instead of re-running pileup generation
- The task is variant calling on DNAseq-only data or non-A-to-I substitution discovery where low-stringency RNAseq pileups are not the appropriate representation

## Inputs

- Sorted RNAseq BAM file(s) (optionally DNAseq BAMs for DnaRna mode), e.g. SRR12492027.SRR12492028.Aligned.sortedByCoord.out.chr10.bam
- Reference genome FASTA (GRCh37 or GRCh38 primary assembly) with samtools faidx index

## Outputs

- REDItools outTable (tab-delimited pileup table of variant/invariant positions with base counts and substitution frequencies)
- bgzip-compressed outTable.gz with tabix index (tabix -s 1 -b 2 -e 2 -c R), ready for REDInet_Inference.py

## How to apply

Run REDItools v1 REDItoolDnaRna.py against a reference genome (e.g. GRCh37 primary assembly FASTA) with deliberately low-stringency settings — -t 40 -c 0,1 -m 0,255 -v 1 -q 0,30 -e -n 0.0 -N 0.0 -u -l -p -s 2 -g 2 -S — so that complete 101-nucleotide intervals needed for downstream predictions are retrieved rather than prematurely filtered. The script traverses the reference genome position by position and emits an outTable of variant and invariant sites supported by RNAseq reads; alternatively REDItools3 can be used for faster pileup generation (seconds vs ~9 hours for v1 on the same jobs). Compress each outTable with bgzip and index it with 'tabix -s 1 -b 2 -e 2 -c R', the exact indexing scheme REDInet expects. Rationale: low stringency preserves candidate editing sites and their flanking read coverage; aggressive pre-filtering at the pileup stage would discard positions REDInet could have classified correctly.

## Related tools

- **REDItools** (Suite of Python scripts (REDItoolDnaRna.py) run at low stringency to generate the variant/invariant RNAseq pileup outTables) — https://github.com/BioinfoUNIBA/REDItools
- **REDItools v1** (Stable REDItools version used for the low-stringency pileup step (~9 hours vs <60 minutes for v3 on identical jobs)) — https://github.com/BioinfoUNIBA/REDItools
- **REDItools3** (Speed-optimized REDItools reimplementation that can generate the same pileup profiles in under 60 minutes) — https://github.com/BioinfoUNIBA/REDItools3
- **pysam** (Python SAM/BAM library wrapping the samtools C-API used underneath the REDItools pileup implementation) — https://github.com/pysam-developers/pysam
- **Python** (Runtime environment (Python 3.9.0 conda env) in which REDItools scripts are executed)

## Examples

```
python ../REDItools/main/REDItoolDnaRna.py -o ../REDInet/Package/Data -i input.bam -f GRCh37.primary_assembly.genome.fa -t 40 -c 0,1 -m 0,255 -v 1 -q 0,30 -e -n 0.0 -N 0.0 -u -l -p -s 2 -g 2 -S; bgzip outTable; tabix -s 1 -b 2 -e 2 -c R outTable.gz
```

## Evaluation signals

- Each outTable is successfully bgzip-compressed and tabix-indexed with -s 1 -b 2 -e 2 -c R without tabix errors
- outTable contains both variant (A-to-G) and invariant sites, since REDInet needs complete 101-nt intervals including flanking positions
- Low-stringency flags are present in the executed command (e.g. -q 0,30, -c 0,1, -v 1, -n 0.0, -N 0.0, -s 2, -g 2); positions are not dropped by premature filtering
- Downstream REDInet_Inference.py can read the indexed outTable and apply its own thresholds (min coverage 50, min 3 Gs, min A-to-G frequency 0.01), confirming the pileup fed the classifier
- On large jobs, runtime is consistent with REDItools expectations (~9 h for v1 vs <60 min for v3), indicating the intended tool version/stringency was used

## Limitations

- REDItools v1 is ~10x slower than the REDItools3 re-implementation (~9 hours vs <60 minutes for the same jobs), so version choice affects throughput
- Low-stringency settings produce large intermediate outTables because invariant and low-support positions are retained for downstream filtering
- REDInet discards positions with missing nucleotides in the 101-nt window or below its own coverage/substitution thresholds, so pileup completeness at flanking positions matters for final site yield

## Evidence
