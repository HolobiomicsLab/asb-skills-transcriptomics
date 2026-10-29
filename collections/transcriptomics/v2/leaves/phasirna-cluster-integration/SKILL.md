---
name: phasirna-cluster-integration
description: Use when small RNA-seq reads are already mapped to a reference (transcriptome
  and/or genome) and you need to call 21-nt or 24-nt phased siRNA loci and integrate
  multi-reference predictions into gene-level PHAS sets.
when_to_use_negative:
- Input reads are raw FASTQ that have not been adapter-trimmed, normalized, length-filtered
  (19–25 nt), or bowtie-mapped — run the preprocess module first
- The analysis requires phasiRNA trigger/initiator miRNA identification or degradome
  target validation — use the target/initiator/deg modules instead
- No GFF3 annotation is available for the reference genome, since the integration
  module requires -an annotation to consolidate loci
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: PhasiHunter
  role: Core tool providing the phase (PHAS loci prediction via hypergeometric and
    phase-score methods) and integration (multi-reference result consolidation) submodules
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: Short-read aligner used by the preprocess module to produce the .map files
    consumed by the phase module
- name: Bedtools
  role: Dependency supporting interval operations underlying cluster/loci handling
    in the workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:efa26fcdaf70c81e2f17887593c3e7d28be6bf93e1fbe75b9b0305fb0b0374c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Bedtools
  - Bowtie
  - PhaseTank
  - PhasiHunter
  - PhasiRNAnalyzer
  - unitas
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# phasiRNA-cluster-integration

## Summary

Predict phased siRNA (phasiRNA) clusters and PHAS loci from small RNA-seq mapping data using PhasiHunter's phase module (hypergeometric and phase-score methods), then consolidate results across reference sequences with the integration module against a GFF3 annotation.

## When to use

Use when small RNA-seq reads are already mapped to a reference (transcriptome and/or genome) and you need to call 21-nt or 24-nt phased siRNA loci and integrate multi-reference predictions into gene-level PHAS sets.

## When NOT to use

- Input reads are raw FASTQ that have not been adapter-trimmed, normalized, length-filtered (19–25 nt), or bowtie-mapped — run the preprocess module first
- The analysis requires phasiRNA trigger/initiator miRNA identification or degradome target validation — use the target/initiator/deg modules instead
- No GFF3 annotation is available for the reference genome, since the integration module requires -an annotation to consolidate loci

## Inputs

- phasiHunter preprocess .map files of length- and abundance-filtered sRNA reads mapped to reference transcriptome (-cm) and/or genome (-gm)
- Reference sequence FASTA files (transcriptome cDNA and/or genome gDNA)
- Trimmed, format-filtered sRNA FASTA file (-fa)
- Reference genome annotation in GFF3 format (-an) for the integration module

## Outputs

- allsiRNA cluster output file (phase_a.txt)
- phasiRNA cluster output file (phase_o.txt)
- Integrated results: integration_o.txt (phasiRNA/PHAS loci), integration_a.txt, integration_s.txt, integration_p.txt
- phasiRNA_cluster plot, phasiRNA.fa, and PHAS.fa from the visulization step

## How to apply

Run PhasiHunter's preprocess output .map files (length 19–25 nt, CPM cutoff 1, normalization base 1000000, bowtie max hits 10) into the phase module with phase length 21 (or 24), phase number 4, prediction method 'b' (both hypergeometric test and phase score), p-value cutoff 0.001, phase score cutoff 15, and phase ratio cutoff 0.4. The dual-method rationale is to combine a statistical test with a phasing-pattern score for integrity and accuracy of prediction. Then run the integration module on the phase module's -o (phasiRNA cluster) and -a (allsiRNA cluster) outputs together with a reference GFF3 annotation to merge loci and assign them to genomic features. In benchmark usage (maize test dataset, transcriptome reference), the integrated 21-PHAS gene set should reach counts comparable to other tools (e.g., 181 PhasiHunter genes with 96.7% overlap to other software). Finally, the visulization step prints the phasiRNA_cluster plot and phasiRNA.fa/PHAS.fa outputs for inspection.

## Related tools

- **PhasiHunter** (Core tool providing the phase (PHAS loci prediction via hypergeometric and phase-score methods) and integration (multi-reference result consolidation) submodules) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (Short-read aligner used by the preprocess module to produce the .map files consumed by the phase module)
- **Bedtools** (Dependency supporting interval operations underlying cluster/loci handling in the workflow)

## Examples

```
phasiHunter phase -cm SRR5049781_processed_cdna.map -c oryza_sativa_cdna.fa -fa SRR5049781_trimmed_format_filter.fa -a phase_a.txt -o phase_o.txt -me b -pl 21 -pn 4 -mh 10 -pv 0.001 -ps 15 -pr 0.4; phasiHunter integration -io phase_o.txt -ia phase_a.txt -an oryza_sativa_gdna.gff3 -g y -o integration_o.txt -pn 4 -pl 21 -pv 0.001
```

## Evaluation signals

- Phase module completes with both methods enabled (-me b) and applies cutoffs pv=0.001, ps=15, pr=0.4 to the phasiRNA cluster output
- Integration module output assigns predicted loci to annotated features from the supplied GFF3 (e.g., PHAS genes)
- On a maize transcriptome-reference benchmark, the integrated 21-PHAS gene count reproduces the reported value of 181 with 96.7% (175/181) overlap to other tools
- The visulization step produces phasiRNA_cluster plot plus phasiRNA.fa and PHAS.fa files without errors
- In-phase siRNA dominance check: phase ratio (pr) cutoff 0.4 filters clusters whose reads do not predominantly occupy the expected 21-nt register

## Limitations

- Only P-method (phase score-only) results are discarded by default (-dp y), which can remove loci supported solely by the phase score
- Prediction sensitivity depends on mapping stringency: max hits default 10 (-mh) may drop reads from repetitive loci
- The -less flag restricts output to cat_1 and cat_2 information, so downstream users may lose lower-confidence category detail

## Evidence

- [abstract] PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods, to ensure the integrity and accuracy of prediction: "PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods, to ensure the integrity and accuracy of prediction"
- [readme] phase module method and cutoff options: "-me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b"
- [readme] phase module cutoff defaults: "-pv: float --  pvalue cutoff, default=0.001, only function with h/b method applied"
- [methods] integration as a distinct workflow step: "3. PhasiRNA and PHAS loci result integration"
- [methods] visulization outputs: "4. Print phasiRNA_cluster plot, phasiRNA.fa, PHAS.fa"
