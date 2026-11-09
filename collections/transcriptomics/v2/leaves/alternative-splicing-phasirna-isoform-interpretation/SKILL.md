---
name: alternative-splicing-phasirna-isoform-interpretation
description: Use when an sRNA-seq analysis needs to distinguish PHAS loci predicted
  on transcript isoforms versus genomic sequence, e.g. to detect isoform-specific
  phasiRNA loss caused by alternative splicing or intron clipping.
when_to_use_negative:
- The goal is degradome target or initiator (trigger miRNA) prediction rather than
  isoform-level phasiRNA interpretation — use phasiHunter target/initiator/deg modules
  instead
- The sRNA data are already clean and abundance-filtered with no reference transcriptome
  available, so isoform-level (cDNA vs gDNA) comparison cannot be made
- Input is a single-reference workflow where only genome-based PHAS loci are needed
  and no transcript isoform distinction is sought (integration with -g y would discard
  only-P results)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0659
tools:
- name: PhasiHunter
  role: Runs the preprocess, phase, integration, and visulization submodules that
    produce dual-reference PHAS predictions, integrated results, and cluster plots
    used to interpret isoform-specific phasiRNA generation
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: Maps length- and abundance-filtered sRNA reads to both cDNA and gDNA Bowtie
    indexes during preprocess (max hits 10), yielding the .map files needed for isoform-vs-genome
    comparison
- name: Biopython
  role: Dependency used by PhasiHunter for sequence handling in phasiRNA prediction
    and output generation
- name: Bedtools
  role: Dependency used by PhasiHunter for genomic interval operations during integration
    with the gff3 annotation
- name: Seqkit
  role: Dependency used by PhasiHunter for fasta/fastq manipulation in the preprocess
    step
- name: Perl5
  role: Dependency (runtime language) for PhasiHunter CLI submodules
  repo: https://www.perl.org
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:fa0ab7671c5bf5c02afd88417963248c8cfd64ae80288fbb106ce17b1491ee87
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# alternative-splicing-phasirna-isoform-interpretation

## Summary

Interprets PhasiHunter's integrated genome- and transcriptome-based phasiRNA predictions to identify phasiRNAs arising specifically through alternative splicing, using the maize gene LOC103650958 as the demonstration case where four transcript isoforms exist but one isoform cannot generate phasiRNAs due to intron clipping.

## When to use

Use when an sRNA-seq analysis needs to distinguish PHAS loci predicted on transcript isoforms versus genomic sequence, e.g. to detect isoform-specific phasiRNA loss caused by alternative splicing or intron clipping.

## When NOT to use

- The goal is degradome target or initiator (trigger miRNA) prediction rather than isoform-level phasiRNA interpretation — use phasiHunter target/initiator/deg modules instead
- The sRNA data are already clean and abundance-filtered with no reference transcriptome available, so isoform-level (cDNA vs gDNA) comparison cannot be made
- Input is a single-reference workflow where only genome-based PHAS loci are needed and no transcript isoform distinction is sought (integration with -g y would discard only-P results)

## Inputs

- sRNA-seq fastq/fastq.gz file (e.g. SRR5049781.fastq.gz)
- reference transcriptome fasta (e.g. oryza_sativa_cdna.fa) and its Bowtie index
- reference genome fasta (e.g. oryza_sativa_gdna.fa) and its Bowtie index
- genome gff3 annotation (e.g. oryza_sativa_gdna.gff3)
- preprocessed cDNA- and gDNA-based .map files from phasiHunter preprocess

## Outputs

- phase_o.txt (phasiRNA clusters) and phase_a.txt (allsiRNA clusters)
- integration_o.txt, integration_a.txt, integration_s.txt, integration_p.txt merging genome- and transcriptome-based PHAS loci
- phasiRNA.fa, PHAS.fa, alignment.txt
- cdna- and gdna-based phasiRNA cluster plots (visualizable in IGV-sRNA)
- isoform-level interpretation: which transcript isoforms produce phasiRNAs and which are phasiRNA-deficient due to intron clipping

## How to apply

Run the full PhasiHunter pipeline on a single sRNA-seq library (e.g. rice SRR5049781 from test_osa.tar.gz): preprocess in raw mode with sRNA length cutoffs 19–25 nt, CPM cutoff 1, normalization base 1000000, and bowtie max hits 10 against both the cDNA and gDNA Bowtie indexes; then run phasiHunter phase with both prediction methods (-me b), phase length 21, phase number 4, island 5, pvalue 0.001, phase score 15, and phase ratio 0.4. Merge genome- and transcriptome-based PHAS loci with phasiHunter integration using the genome gff3 annotation (-g y, discard only-P results y) to obtain integration_o/a/s/p.txt. Finally run phasiHunter visulization to produce phasiRNA.fa, PHAS.fa, alignment.txt, and cdna/gdna phasiRNA cluster plots; inspect alignment.txt and the cluster plot at the target locus (LOC103650958) to confirm the four transcript isoforms and locate the isoform whose intron-clipped region cannot generate phasiRNAs. The rationale is that dual reference (transcriptome + genome) prediction plus integration isolates phasing signal that exists only on spliced transcripts.

## Related tools

- **PhasiHunter** (Runs the preprocess, phase, integration, and visulization submodules that produce dual-reference PHAS predictions, integrated results, and cluster plots used to interpret isoform-specific phasiRNA generation) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (Maps length- and abundance-filtered sRNA reads to both cDNA and gDNA Bowtie indexes during preprocess (max hits 10), yielding the .map files needed for isoform-vs-genome comparison)
- **Biopython** (Dependency used by PhasiHunter for sequence handling in phasiRNA prediction and output generation)
- **Bedtools** (Dependency used by PhasiHunter for genomic interval operations during integration with the gff3 annotation)
- **Seqkit** (Dependency used by PhasiHunter for fasta/fastq manipulation in the preprocess step)
- **Perl5** (Dependency (runtime language) for PhasiHunter CLI submodules) — https://www.perl.org

## Examples

```
phasiHunter integration -io /home/user/test_osa/phase_o.txt -ia /home/user/test_osa/phase_a.txt -an /home/user/test_osa/oryza_sativa_gdna.gff3 -g y -o /home/user/test_osa/integration_o.txt -a /home/user/test_osa/integration_a.txt -s /home/user/test_osa/integration_s.txt -po /home/user/test_osa/integration_p.txt -j 1 -pn 4 -pl 21 -pv 0.001 -il 5
```

## Evaluation signals

- integration_p.txt reports the LOC103650958 PHAS locus meeting the prediction cutoffs (phase number ≥ 4, pvalue ≤ 0.001)
- The cdna-based cluster plot renders phased 21-nt siRNA stacks across the locus with an expected isoform-specific loss of phasing in the intron-clipped isoform
- alignment.txt at the LOC103650958 locus shows sRNA alignments consistent with four transcript isoforms, with phasing restricted to the phasiRNA-competent isoforms
- Both phase_a.txt and phase_o.txt are produced with -me b (hypergeometric test plus phase score) and pass the phase ratio cutoff of 0.4
- Integration with -g y and discard-only-P y successfully merges genome- and transcriptome-based results without empty output files

## Limitations

- Only transcriptome-based results contribute isoform resolution; integration with discard-only-P (-g y) removes PHAS loci supported only by the phase-score method, so weak splicing-specific signals may be discarded
- PhasiHunter prediction sensitivity depends on the chosen cutoffs (21-nt phase length, phase number 4, pvalue 0.001, phase score 15, phase ratio 0.4); alternative phase lengths (24 nt) are supported but not demonstrated in this case
- Version history in the README only documents 1.0.1 (25/02/2024); no discussion section content was available, so limitations and reproducibility details from the paper are not extractable

## Evidence

- [abstract] PhasiHunter can efficiently identify the phasiRNAs generated through alternative splicing events: "PhasiHunter can efficiently identify the phasiRNAs generated through alternative splicing events"
- [results] the maize gene LOC103650958 produces four transcript isoforms, one of which is unable to generate phasiRNAs due to intron clipping: [evidence span withheld — anchor status: not_found]
- [readme] raw(mode): trim adaptor --> normalization --> length and abundance filter --> mapping: "raw(mode): trim adaptor --> normalization --> length and abundance filter --> mapping"
- [readme] -me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b: "-me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b"
- [abstract] PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods, to ensure the integrity and accuracy of prediction: "PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods, to ensure the integrity and accuracy of prediction"
