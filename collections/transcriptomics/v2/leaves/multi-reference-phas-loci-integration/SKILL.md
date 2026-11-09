---
name: multi-reference-phas-loci-integration
description: Use when sRNA-seq data must be mined for phased siRNA (phasiRNA) clusters
  against both a genome and a transcriptome reference, and when integrated multi-reference
  PHAS loci sets or alternative-splicing-derived phasiRNAs are the target output.
when_to_use_negative:
- Only one reference sequence (single transcriptome or single genome) is available
  and no integration of multi-reference results is intended — run phasiHunter phase
  alone instead.
- The goal is degradome-based target verification or initiator (trigger) miRNA prediction
  rather than PHAS loci discovery and integration — use the target/initiator/deg submodules
  instead.
- Input sRNA reads are already mapped (no preprocess step needed) and no PHAS loci
  prediction is requested.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: PhasiHunter
  role: 'primary tool: preprocess, phase, and integration submodules for multi-reference
    PHAS loci prediction and merging'
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: short-read alignment of sRNA reads to cDNA and gDNA index prefixes during
    preprocess
- name: Trim_galore
  role: adapter trimming and read filtering before mapping-mode preprocess
  repo: https://github.com/FelixKrueger/TrimGalore
- name: Seqkit
  role: FASTA/FASTQ manipulation in the preprocessing pipeline
- name: Biopython
  role: dependency used by PhasiHunter for sequence handling and cluster plotting
- name: Bedtools
  role: dependency used by PhasiHunter for genomic interval operations during integration
    with GFF3 annotation
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:71fbbf64ee42e69facd970271885b0c3824d7970308c2b491565e7c0d6d0d22f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Bedtools
  - Biopython
  - Bowtie
  - Perl5
  - PhasiHunter
  - Seqkit
  - Trim_galore
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# multi-reference-phas-loci-integration

## Summary

Predict PHAS loci independently against multiple reference sequences (transcriptome cDNA and genome gDNA) with PhasiHunter's phase module, then merge the results with the integration module to recover PHAS genes missed by single-reference tools, including phasiRNAs arising from alternative splicing.

## When to use

Use when sRNA-seq data must be mined for phased siRNA (phasiRNA) clusters against both a genome and a transcriptome reference, and when integrated multi-reference PHAS loci sets or alternative-splicing-derived phasiRNAs are the target output.

## When NOT to use

- Only one reference sequence (single transcriptome or single genome) is available and no integration of multi-reference results is intended — run phasiHunter phase alone instead.
- The goal is degradome-based target verification or initiator (trigger) miRNA prediction rather than PHAS loci discovery and integration — use the target/initiator/deg submodules instead.
- Input sRNA reads are already mapped (no preprocess step needed) and no PHAS loci prediction is requested.

## Inputs

- raw sRNA-seq FASTQ/FASTQ.GZ (e.g. SRR5049781.fastq.gz) or trimmed/filtered FASTA
- reference transcriptome cDNA FASTA (e.g. oryza_sativa_cdna.fa) and its Bowtie index prefix
- reference genome gDNA FASTA (e.g. oryza_sativa_gdna.fa) and its Bowtie index prefix
- genome annotation GFF3 (e.g. oryza_sativa_gdna.gff3)
- phase module outputs phase_o.txt and phase_a.txt

## Outputs

- SRR5049781_processed_cdna.map and SRR5049781_processed_gdna.map (processed Bowtie map files)
- phase_a.txt (all siRNA clusters) and phase_o.txt (phasiRNA clusters)
- integration_o.txt, integration_a.txt, integration_s.txt (integrated PHAS loci), integration_p.txt
- visualization outputs: phasiRNA.fa, PHAS.fa, alignment.txt, cdna/gdna phasiRNA cluster plots

## How to apply

First run `phasiHunter preprocess` twice: mode r on the raw FASTQ (e.g. SRR5049781.fastq.gz) against the cDNA Bowtie index, and mode m on the trimmed/filtered FASTA against the gDNA index, using length cutoffs 19–25 nt, CPM cutoff 1, normalization base 1,000,000, and max hits 10, yielding *_processed_cdna.map and *_processed_gdna.map. Then run `phasiHunter phase` with both map files and both reference FASTAs (-cm/-c and -gm/-g), using prediction method b (hypergeometric test plus phase score), phase length 21, phase number 4, island 5, p-value cutoff 0.001, phase score cutoff 15, phase ratio cutoff 0.4, and parallel threads (e.g. 20), producing phase_a.txt and phase_o.txt. Finally run `phasiHunter integration` with -io phase_o.txt, -ia phase_a.txt, the genome GFF3 annotation, -g y (genome-based PHAS loci present), -pn 4, -pl 21, -pv 0.001, -il 5, and -dp y to discard loci supported only by the phase-score method, producing integration_o.txt, integration_a.txt, integration_s.txt, and integration_p.txt. The rationale is that transcriptome-based prediction captures isoform-specific phasing (e.g. maize LOC103650958 yields four isoforms, one phasiRNA-deficient due to intron clipping) while genome-based prediction is annotation-independent; integration reconciles both. Optionally run `phasiHunter visulization` to produce phasiRNA.fa, PHAS.fa, alignment.txt, and cdna/gdna cluster plots for IGV-style inspection.

## Related tools

- **PhasiHunter** (primary tool: preprocess, phase, and integration submodules for multi-reference PHAS loci prediction and merging) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (short-read alignment of sRNA reads to cDNA and gDNA index prefixes during preprocess)
- **Trim_galore** (adapter trimming and read filtering before mapping-mode preprocess) — https://github.com/FelixKrueger/TrimGalore
- **Seqkit** (FASTA/FASTQ manipulation in the preprocessing pipeline)
- **Biopython** (dependency used by PhasiHunter for sequence handling and cluster plotting)
- **Bedtools** (dependency used by PhasiHunter for genomic interval operations during integration with GFF3 annotation)

## Examples

```
phasiHunter integration -io phase_o.txt -ia phase_a.txt -an oryza_sativa_gdna.gff3 -g y -o integration_o.txt -a integration_a.txt -s integration_s.txt -po integration_p.txt -pn 4 -pl 21 -pv 0.001 -il 5
```

## Evaluation signals

- All four integration outputs exist: integration_o.txt, integration_a.txt, integration_s.txt, integration_p.txt, with integration_s.txt reporting PHAS loci integrated from both reference-based predictions.
- Reported PHAS loci satisfy the prediction cutoffs: phase number >= 4, p-value <= 0.001, phase score >= 15, phase ratio >= 0.4, phase length 21, island 5.
- With -dp y, loci supported only by the phase-score (p) method are absent from the final integration results.
- On the rice test dataset, the integrated results reproduce the paper's claim of 99 additional specific PHAS genes versus other software (Supplementary Fig. S4B, Supplementary Table S4).
- Cluster plots render phased 21-nt siRNA stacks across PHAS loci, and alternative-splicing cases (e.g. LOC103650958) show isoform-specific loss of phasing in the intron-clipped region.

## Limitations

- Loci supported only by the phase-score method are discarded when -dp y (the default), which may remove weakly supported but real phasiRNA-producing regions.
- Integration requires the genome GFF3 annotation when genome-based PHAS loci are present (-g y); missing or mismatched annotations will break locus merging.
- Only phase lengths 21 or 24 are supported (-pl); other phasiRNA sizes are out of scope.
- The README provides an update log (1.0.1, 25/02/2024) but no explicit software version number in the article and no discussion-section limitations.

## Evidence

- [abstract] PhasiHunter can identify phasiRNAs and their regulatory networks based on multiple reference sequences and the predicted results can be automatically integrated: "PhasiHunter can identify phasiRNAs and their regulatory networks based on multiple reference sequences and the predicted results can be automatically integrated"
- [readme] integration module parameters: "-dp: str   -- y | n, discard only P method result, default=y"
- [readme] phase module method options: "-me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b"
- [methods] workflow steps in methods: "2. PhasiRNA and PHAS loci prediction"
- [methods] demo data link: "https://cbi.njau.edu.cn/PhasiHunter_demo_data/test_osa.tar.gz"
