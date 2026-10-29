---
name: rna-seq-read-depth-subsampling
description: Use when benchmarking fusion-gene detection sensitivity (or similar read-count-dependent
  callers) across decreasing sequencing depths from a single deeply sequenced paired-end
  FASTQ sample.
when_to_use_negative:
- Input is single-end RNA-seq, whole-genome, or whole-exome data — subsampled reads
  cannot support paired-end fusion anchoring
- The goal is to detect fusion genes not involving a user-provided target gene; anchored
  methods only return fusions containing the anchored gene
- The original FASTQ depth is already too shallow for 7 halvings to leave meaningful
  read counts
edam_operation: http://edamontology.org/operation_3230
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3522
tools:
- name: samtools
  role: Subsample read pairs by successive halving to create depth series
- name: Anchored-fusion
  role: Fusion caller under test that anchors the gene of interest; uses BWA-MEM alignment
    and BLAT local alignment
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: STAR-Fusion v1.12.0
  role: Competing de novo bRNA-seq fusion detection method run at each depth
  repo: https://github.com/STAR-Fusion
- name: FusionInspector v2.8.0
  role: Competing bRNA-seq fusion detection method run at each depth
  repo: https://github.com/FusionInspector/FusionInspector
- name: FusionCatcher v1.30
  role: Competing bRNA-seq fusion detection method run at each depth
  repo: https://github.com/ndaniel/fusioncatcher
- name: Arriba v2.4.0
  role: Competing STAR-based bRNA-seq fusion detection method run at each depth
  repo: https://github.com/suhrig/arriba
- name: BWA-MEM
  role: Aligner used by Anchored-fusion to map reads against the anchored gene and
    genome
- name: BLAT
  role: Local alignment of candidate chimeric reads to predict precise fusion location
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:2b2b5a2e3c7b7f94a0f153b1a7fa62e9c7e941e5769ce2a2eb3f07511e72c5cc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  repo_url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  tools:
  - Anchored-fusion
  - Arriba
  - Arriba v2.4.0
  - BLAT
  - BWA-MEM
  - FusionCatcher
  - FusionCatcher v1.30
  - FusionInspector
  - FusionInspector v2.8.0
  - STAR-Fusion v1.12.0
  - samtools
  edam_operation: http://edamontology.org/operation_3230
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3522
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# rna-seq-read-depth-subsampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates progressive sequencing-depth dilution series from paired-end RNA-seq FASTQ data by repeated random halving of read pairs, to benchmark caller sensitivity as depth is reduced below ~1 million reads.

## When to use

Use when benchmarking fusion-gene detection sensitivity (or similar read-count-dependent callers) across decreasing sequencing depths from a single deeply sequenced paired-end FASTQ sample.

## When NOT to use

- Input is single-end RNA-seq, whole-genome, or whole-exome data — subsampled reads cannot support paired-end fusion anchoring
- The goal is to detect fusion genes not involving a user-provided target gene; anchored methods only return fusions containing the anchored gene
- The original FASTQ depth is already too shallow for 7 halvings to leave meaningful read counts

## Inputs

- Paired-end RNA-seq FASTQ files (R1/R2), e.g. CCLE cell line bRNA-seq from SRA:SRP186687
- Reference genome FASTA (hg38) and compatible GTF annotation
- Target gene transcript reference FASTA for the anchored fusion search (from NCBI Gene)

## Outputs

- 8 subsampled FASTQ depth levels per sample (7 halvings, final depth <1 million read pairs)
- Per-depth fusion calls for each tool
- Tool-by-depth detection matrix of fusion genes per caller per subsampling level

## How to apply

Start from paired-end FASTQ files (e.g. CCLE bRNA-seq cell lines K562, TE441T, NALM6, NCI-H660, NCI-H3122, KM12 from SRA accession SRP186687) and generate 8 depths per sample via 7 successive rounds of random halving of read pairs, e.g. with samtools view subsampling, ending below 1 million reads. At each depth, run each caller (STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, Arriba v2.4.0 against hg38 fasta plus matching GTF, and the anchored method via BWA-MEM plus BLAT) with identical inputs. Collate detections into a tool-by-depth detection matrix recording which fusions each caller reports per depth. The subsampling rationale is that sensitivity loss at halved depth reveals the shallow-sequencing limit of each algorithm; Anchored-fusion was validated to detect fusions even after 7 halvings (<1 M depth) where all competitors failed.

## Related tools

- **samtools** (Subsample read pairs by successive halving to create depth series)
- **Anchored-fusion** (Fusion caller under test that anchors the gene of interest; uses BWA-MEM alignment and BLAT local alignment) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **STAR-Fusion v1.12.0** (Competing de novo bRNA-seq fusion detection method run at each depth) — https://github.com/STAR-Fusion
- **FusionInspector v2.8.0** (Competing bRNA-seq fusion detection method run at each depth) — https://github.com/FusionInspector/FusionInspector
- **FusionCatcher v1.30** (Competing bRNA-seq fusion detection method run at each depth) — https://github.com/ndaniel/fusioncatcher
- **Arriba v2.4.0** (Competing STAR-based bRNA-seq fusion detection method run at each depth) — https://github.com/suhrig/arriba
- **BWA-MEM** (Aligner used by Anchored-fusion to map reads against the anchored gene and genome)
- **BLAT** (Local alignment of candidate chimeric reads to predict precise fusion location)

## Examples

```
samtools view -s 0.0078125 -b aligned.bam > depth7.bam; python Anchored_Fusion.py --file_anchored_cds=target_gene.fasta --fastq1=sample_R1.fastq.gz --fastq2=sample_R2.fastq.gz --out_folder=out --file_ref_seq=hg38.fa --file_ref_ann=hg38.gtf
```

## Evaluation signals

- Subsampling yields exactly 8 depth levels per sample after 7 halvings, with the final level below 1 million read pairs
- Known fusions (e.g. BCR-ABL1 in K562, EML4-ALK in NCI-H3122) are detected at full depth by at least one tool, confirming the pipeline before dilution
- Tool-by-depth matrix shows monotonic loss of detections as depth decreases for most callers; Anchored-fusion detections persist at all depths
- Cross-depth calls at full depth should be a superset of (or equal to) calls at halved depths for deterministic callers, modulo stochastic subsampling noise

## Limitations

- Random halving is stochastic; single subsample draws per depth can vary, so near-threshold calls may fluctuate
- Subsampling reads changes the effective fragment support but not read quality profiles, so it approximates rather than reproduces true shallow sequencing runs
- Anchored methods miss fusions that do not include the supplied target gene; de novo competitors detect all fusions but with lower sensitivity at low depth

## Evidence

- [other] Generate 8 sequencing depths per sample by 7 successive rounds of random halving of read pairs, ending at a depth below 1 million reads: [evidence span withheld — anchor status: not_found]
- [other] Subsampling finding: [evidence span withheld — anchor status: not_found]
