---
name: computational-runtime-benchmarking
description: Use when comparing the computational performance (wall-clock runtime)
  of two tool versions on identical inputs, e.g. REDItools v1 vs v3 full-genome pileups
  of large RNAseq BAMs, where the paper's claim must be reproduced quantitatively.
when_to_use_negative:
- The analysis goal is to compare editing-detection accuracy (e.g. REDInet vs EditPredict,
  RED-ML, RDDpred precision/specificity) rather than computational runtime
- The input BAMs are small test files (e.g. the chromosome-10 subset bundled in REDInet/Package/Data)
  where runtime differences are not meaningful and cannot reproduce the 277M-read
  benchmark scale
- You need biologically validated editing calls or REDInet classification results
  — this skill only measures pileup generation time and output validity, not editing
  performance
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: REDItools v1
  role: Baseline tool timed running REDItoolDnaRna.py full pileup jobs (~9 hours reported)
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: REDItools3
  role: Faster re-implementation timed on the identical BAM/reference (<60 minutes
    reported), installed via pip and run with python -m reditools
  repo: https://github.com/BioinfoUNIBA/REDItools3
- name: Samtools
  role: Used to index the GRCh37 reference FASTA (samtools faidx) consumed by both
    pileup runs
- name: Tabix
  role: Used with bgzip to validate that outTables from both timed runs are indexable,
    REDInet-consumable outputs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:dcc1ab3d08e140ed8600e316b312529e26c93210aeb6944c551a7a9eba0580f6
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
  - REDItools v1
  - REDItools3
  - Samtools
  - Tabix
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# computational-runtime-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for head-to-head wall-clock benchmarking of two tool versions on identical, massive inputs — here quantifying the ~10x speedup of REDItools v3 over REDItools v1 when generating complete RNAseq pileup profiles from ~277 million-read HEK293T KO BAMs.

## When to use

Use when comparing the computational performance (wall-clock runtime) of two tool versions on identical inputs, e.g. REDItools v1 vs v3 full-genome pileups of large RNAseq BAMs, where the paper's claim must be reproduced quantitatively.

## When NOT to use

- The analysis goal is to compare editing-detection accuracy (e.g. REDInet vs EditPredict, RED-ML, RDDpred precision/specificity) rather than computational runtime
- The input BAMs are small test files (e.g. the chromosome-10 subset bundled in REDInet/Package/Data) where runtime differences are not meaningful and cannot reproduce the 277M-read benchmark scale
- You need biologically validated editing calls or REDInet classification results — this skill only measures pileup generation time and output validity, not editing performance

## Inputs

- Coordinate-sorted BAM files of stranded paired-end RNAseq alignments (~277 million reads per BAM, e.g. SRR12492027/SRR12492028 from HEK293T KO samples)
- GRCh37 primary assembly reference FASTA indexed with samtools faidx
- REDItools v1 (REDItoolDnaRna.py) and REDItools3 installed in dedicated conda environments

## Outputs

- reditools_runtime_benchmark.tsv containing recorded wall-clock runtimes for REDItools v1 and REDItools3 and the computed v1/v3 runtime ratio
- bgzip-compressed, tabix-indexed REDItools outTable files (-s 1 -b 2 -e 2 -c R) confirming both tools emitted valid, REDInet-consumable pileup tables

## How to apply

Obtain coordinate-sorted BAM files of stranded paired-end RNAseq (e.g. SRA accessions SRR12492027/SRR12492028, ~277 million aligned reads per BAM) prepared via the REDItools protocol, plus the GRCh37 primary assembly FASTA indexed with samtools faidx. Run REDItools v1 REDItoolDnaRna.py with the low-stringency settings required to retrieve complete 101-base REDInet intervals (-t 40 threads, -c 0,1, -m 0,255, -v 1, -q 0,30, -e, -n 0.0, -N 0.0, -u, -l, -p, -s 2, -g 2, -S), recording total wall-clock time; then run REDItools3 (installed via pip in a dedicated conda environment, invoked as python -m reditools) on the identical BAM and reference with equivalent settings. Validate that both tools produced REDInet-consumable outputs by compressing with bgzip and indexing with tabix -s 1 -b 2 -e 2 -c R. Compile recorded runtimes into reditools_runtime_benchmark.tsv and compute the v1/v3 ratio.

## Related tools

- **REDItools v1** (Baseline tool timed running REDItoolDnaRna.py full pileup jobs (~9 hours reported)) — https://github.com/BioinfoUNIBA/REDItools
- **REDItools3** (Faster re-implementation timed on the identical BAM/reference (<60 minutes reported), installed via pip and run with python -m reditools) — https://github.com/BioinfoUNIBA/REDItools3
- **Samtools** (Used to index the GRCh37 reference FASTA (samtools faidx) consumed by both pileup runs)
- **Tabix** (Used with bgzip to validate that outTables from both timed runs are indexable, REDInet-consumable outputs)

## Examples

```
python ../REDItools/main/REDItoolDnaRna.py -o ../REDInet/Package/Data -i ../REDInet/Package/Data/SRR12492027.SRR12492028.Aligned.sortedByCoord.out.chr10.bam -f ../REDInet/Package/Utilities/GRCh37.primary_assembly.genome.fa -t 40 -c 0,1 -m 0,255 -v 1 -q 0,30 -e -n 0.0 -N 0.0 -u -l -p -s 2 -g 2 -S
```

## Evaluation signals

- REDItools3 completes the full pileup profile in under 60 minutes while REDItools v1 requires roughly 9 hours on the same job, yielding a v1/v3 ratio of approximately 10x
- Both outTables pass bgzip compression and tabix indexing with -s 1 -b 2 -e 2 -c R, confirming valid, REDInet-consumable pileup tables
- Both tools were run on the identical BAM, the identical faidx-indexed GRCh37 reference, and equivalent filter settings (-t 40, -c 0,1, -m 0,255, -v 1, -q 0,30, -e, -n 0.0, -N 0.0, -u, -l, -p, -s 2, -g 2, -S), so the timing comparison is controlled
- reditoools_runtime_benchmark.tsv contains recorded wall-clock times for both tools plus the computed runtime ratio
- Each tool ran in a dedicated conda environment (per the REDInet README recommendation) so dependency conflicts do not skew timing

## Limitations

- The ~10x speedup claim is specific to full pileup profiles on massive RNAseq data (~277 million aligned reads per BAM); runtimes may differ on smaller or chromosome-subset BAMs
- Low-stringency REDItools settings (-n 0.0, -N 0.0, etc.) are required to retrieve the complete 101-base intervals REDInet needs for prediction, making the job more expensive than typical stringent pileups
- REDInet and REDItools should be installed and run in distinct Anaconda/Miniconda environments, so benchmark runs must account for environment setup time separately from tool runtime
- Only wall-clock time of the full pileup job is benchmarked; memory, CPU, and output equivalence at base level are not explicitly assessed in the reported benchmark

## Evidence
