---
name: phase-score-based-phasing-analysis
description: Use when small-RNA sequencing reads (19–25 nt) should be scanned for
  phased siRNA-producing loci (PHAS genes) against a reference transcriptome, genome,
  or full-length transcriptome, especially to recover phasiRNAs that only the phase
  score method detects.
when_to_use_negative:
- The question is whether predicted PHAS loci pass the hypergeometric pvalue cutoff
  alone — with -dp y, only-P-method results are discarded, so phase-score-only hits
  will not survive integration.
- Input is degradome data or initiator/target prediction (use 'phasiHunter target/initiator'
  or 'phasiHunter deg' instead of the phase module).
- Reads outside the 19–25 nt window or with abundance below the CPM cutoff of 1 —
  preprocess filtering will remove them before phasing is meaningful.
edam_operation: http://edamontology.org/operation_3287
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: PhasiHunter
  role: CLI tool whose 'phase' module performs phase-score (and hypergeometric) phasiRNA/PHAS
    locus prediction; installed from GitHub, Docker, or the provided conda/mamba env.yaml
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: maps filtered 19–25 nt sRNA reads to reference sequences (max 10 hits) and
    builds the reference indexes used by the phase module
- name: Trim_galore
  role: adapter trimming in the 'phasiHunter preprocess -m r' step preceding phase
    prediction
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:0a40efae61a2062524bb769af21831a3e547a80f39db0a2c933a2bba30fdbc08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Biopython
  - Bowtie
  - Perl5
  - PhasiHunter
  - Trim_galore
  edam_operation: http://edamontology.org/operation_3287
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# phase-score-based-phasing-analysis

## Summary

Predict phasiRNAs and PHAS loci from sRNA-seq data using PhasiHunter's phase score method (with optional hypergeometric-test confirmation) against transcriptome, genome, or full-length-transcriptome references. Phase scoring captures phased 21-nt (or 24-nt) siRNA accumulation patterns that hypergeometric testing alone can miss.

## When to use

Use when small-RNA sequencing reads (19–25 nt) should be scanned for phased siRNA-producing loci (PHAS genes) against a reference transcriptome, genome, or full-length transcriptome, especially to recover phasiRNAs that only the phase score method detects.

## When NOT to use

- The question is whether predicted PHAS loci pass the hypergeometric pvalue cutoff alone — with -dp y, only-P-method results are discarded, so phase-score-only hits will not survive integration.
- Input is degradome data or initiator/target prediction (use 'phasiHunter target/initiator' or 'phasiHunter deg' instead of the phase module).
- Reads outside the 19–25 nt window or with abundance below the CPM cutoff of 1 — preprocess filtering will remove them before phasing is meaningful.

## Inputs

- sRNA-seq FASTQ/FASTQ.GZ library (e.g. SRR5049781) or a preprocessed .map file
- reference transcriptome FASTA (e.g. oryza_sativa_cdna.fa) and/or reference genome FASTA (oryza_sativa_gdna.fa)
- full-length transcriptome (flnc) FASTA and its Bowtie index, for flnc-based prediction
- Bowtie indexes for each reference sequence
- length- and abundance-filtered sRNA FASTA for the -fa option

## Outputs

- phase_o.txt: phasiRNA cluster output file
- phase_a.txt: allsiRNA cluster output file
- downstream integration outputs (integration_o.txt, integration_a.txt, integration_s.txt, integration_p.txt) when 'phasiHunter integration' is run on phase results

## How to apply

First preprocess the sRNA library with 'phasiHunter preprocess -m r' (Trim_galore adapter trimming, normalization to a base of 1,000,000, 19–25 nt length filter at CPM cutoff 1, Bowtie mapping with max 10 hits). Build a Bowtie index for the chosen reference (e.g. the full-length transcriptome, flnc) and generate the flnc-based map file with 'phasiHunter preprocess -m m', supplying it via -fm together with the flnc fasta via -f in the phase module. Run 'phasiHunter phase' with -me p (or -me b to combine both methods), phase score cutoff -ps 15, phase ratio cutoff -pr 0.4, phase length -pl 21, minimum phase number -pn 4, cluster island -il 5, and parallel cores -j 20 to emit phase_a.txt and phase_o.txt. The phase score method is one of PhasiHunter's two major prediction algorithms and, when combined with the hypergeometric test (-me b, pvalue cutoff -pv 0.001), ensures integrity and accuracy; downstream 'phasiHunter integration' can discard only-P-method results with -dp y if hypergeometric confirmation is required.

## Related tools

- **PhasiHunter** (CLI tool whose 'phase' module performs phase-score (and hypergeometric) phasiRNA/PHAS locus prediction; installed from GitHub, Docker, or the provided conda/mamba env.yaml) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (maps filtered 19–25 nt sRNA reads to reference sequences (max 10 hits) and builds the reference indexes used by the phase module)
- **Trim_galore** (adapter trimming in the 'phasiHunter preprocess -m r' step preceding phase prediction) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
phasiHunter phase -cm SRR5049781_processed_cdna.map -c oryza_sativa_cdna.fa -gm SRR5049781_processed_gdna.map -g oryza_sativa_gdna.fa -fm SRR5049781_processed_flnc.map -f flnc.fa -fa SRR5049781_trimmed_format_filter.fa -o phase_o.txt -a phase_a.txt -me b -pl 21 -pn 4 -il 5 -pv 0.001 -ps 15 -pr 0.4 -j 20
```

## Evaluation signals

- phase_o.txt and phase_a.txt exist after 'phasiHunter phase' completes and are non-empty.
- Reported PHAS loci satisfy the applied cutoffs: phase score >= 15 (-ps 15), phase ratio >= 0.4 (-pr 0.4), and (when -me b) hypergeometric pvalue <= 0.001 (-pv 0.001).
- Running the demo dataset test_osa with flnc-based prediction reproduces 122 unique PHAS genes, ~96.7% of which are novel unannotated genes.
- integration_p.txt (after 'phasiHunter integration -g y -dp y') contains PHAS loci records that pass the pvalue cutoff, confirming hypergeometric-supported calls survive the discard of only-P-method results.
- Temporary .phasiHuter_bowtieIndex files are deleted when -cl y (default) is used.

## Limitations

- PhasiHunter requires a Linux platform and multiple external dependencies (Bowtie, Biopython, Bedtools, Dnapi, Trim_galore, Seqkit, Perl5, Fasta36, TarHunter); manual installation needs PATH setup, whereas the Docker image ships with all dependencies configured.
- Only a changelog entry 1.0.1 (25/02/2024) is documented; no explicit version number or discussion-section limitations/reproducibility details are extractable.
- The phase score method's results alone can be discarded at integration time (-dp y, default), so unconfirmed phase-score-only loci should be interpreted cautiously.

## Evidence

- [abstract] PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods, to ensure the integrity and accuracy of prediction: "PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods, to ensure the integrity and accuracy of prediction"
- [readme] phase module parameters: "-me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b"
- [readme] phase score cutoff default: "-ps: float --  phase score cutoff, default=15, only function with p/b method applied"
- [results] full-length transcriptome finding: [evidence span withheld — anchor status: not_found]
- [methods] integration discard option: "-dp: str   -- y | n, discard only P method result, default=y"
