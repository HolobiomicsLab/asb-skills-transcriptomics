---
name: hypergeometric-phase-enrichment-testing
description: Use when predicting PHAS loci/phasiRNAs from a preprocessed sRNA map
  file against a transcriptome or genome reference and you need statistically phased
  clusters (e.g. 21-nt phasiRNAs) rather than heuristic-only phase calls.
when_to_use_negative:
- The goal is calling miRNA targets or degradome validation (use the target/initiator
  modules with eTM search or T-plot instead of the phase module).
- Input sRNA reads have not been length/abundance filtered (19-25 nt, CPM >= 1) and
  mapped — the phase module expects preprocessed map files, not raw FASTQ.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: PhasiHunter
  role: primary tool executing the phase module that combines the hypergeometric test
    (-me h) and phase score (-me p) algorithms with forward/reverse search
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: aligns length/abundance-filtered sRNA reads to cDNA and genome reference indices
    to produce the map files consumed by the phase module
- name: Seqkit
  role: sequence format handling in the preprocessing step upstream of phase detection
- name: Trim_galore
  role: adapter trimming of raw sRNA reads before preprocessing and phase prediction
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:e6e41ed1067e1dce210c559b08651dc42699f7c880d355e90c1c8f11a9d358a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Biopython
  - Bowtie
  - PhaseTank
  - PhasiHunter
  - PhasiRNAnalyzer
  - Seqkit
  - Trim_galore
  - unitas
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6956521739130435
n_claims: 23
---

# hypergeometric-phase-enrichment-testing

## Summary

Dual-algorithm PHAS loci detection as implemented in PhasiHunter's phase module: a hypergeometric distribution test augmented with forward and reverse search plus a phase-score calculation, to sensitively identify phased siRNA (phasiRNA) clusters from mapped sRNA reads.

## When to use

Use when predicting PHAS loci/phasiRNAs from a preprocessed sRNA map file against a transcriptome or genome reference and you need statistically phased clusters (e.g. 21-nt phasiRNAs) rather than heuristic-only phase calls.

## When NOT to use

- The goal is calling miRNA targets or degradome validation (use the target/initiator modules with eTM search or T-plot instead of the phase module).
- Input sRNA reads have not been length/abundance filtered (19-25 nt, CPM >= 1) and mapped — the phase module expects preprocessed map files, not raw FASTQ.

## Inputs

- cDNA-based sRNA map file (e.g. SRR5049781_processed_cdna.map) from PhasiHunter preprocess
- genome-based sRNA map file (e.g. SRR5049781_processed_gdna.map) from PhasiHunter preprocess
- reference transcriptome FASTA (e.g. oryza_sativa_cdna.fa)
- reference genome FASTA (e.g. oryza_sativa_gdna.fa)
- filtered/trimmed sRNA FASTA (e.g. SRR5049781_trimmed_format_filter.fa)

## Outputs

- phase_a.txt — redundant allsiRNA cluster output with columns gene/PHAS_gene, strand, position, abundance, record, sequence, length, pvalue, phase_ratio, phase_number, phase_abundance, phase_score, marker
- phase_o.txt — redundant phasiRNA cluster output with the same documented columns
- .phasiHuter_bowtieIndex temporary files (deleted with -cl y)

## How to apply

After preprocessing sRNA reads (19-25 nt length cutoff, CPM cutoff 1, normalization base 1,000,000, bowtie max hits 10) into map files, run `phasiHunter phase` with prediction method -me b (both hypergeometric test and phase score; default) over the cDNA map (-cm) and genome map (-gm) with their reference FASTAs. Key parameters: phase length 21 (-pl 21), phase number 4 (-pn 4), island 5 (-il 5), p-value cutoff 0.001 (-pv 0.001), phase score cutoff 15 (-ps 15), phase ratio cutoff 0.4 (-pr 0.4), and parallel jobs (-j 20). The hypergeometric algorithm implements both forward and reverse search to enhance PHAS loci detection sensitivity. Downstream, use the integration module with -dp y to discard loci supported only by the phase-score method, keeping only results passing both or the hypergeometric criterion.

## Related tools

- **PhasiHunter** (primary tool executing the phase module that combines the hypergeometric test (-me h) and phase score (-me p) algorithms with forward/reverse search) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (aligns length/abundance-filtered sRNA reads to cDNA and genome reference indices to produce the map files consumed by the phase module)
- **Seqkit** (sequence format handling in the preprocessing step upstream of phase detection)
- **Trim_galore** (adapter trimming of raw sRNA reads before preprocessing and phase prediction) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
phasiHunter phase -cm SRR5049781_processed_cdna.map -c oryza_sativa_cdna.fa -gm SRR5049781_processed_gdna.map -g oryza_sativa_gdna.fa -fa SRR5049781_trimmed_format_filter.fa -a phase_a.txt -o phase_o.txt -me b -pl 21 -pn 4 -il 5 -pv 0.001 -ps 15 -pr 0.4 -j 20 -cl y
```

## Evaluation signals

- phase_a.txt and phase_o.txt both exist and contain the documented table header including pvalue and phase_score columns
- every reported PHAS locus satisfies the applied cutoffs: p-value <= 0.001, phase score >= 15, phase ratio >= 0.4, phase number >= 4, phase length 21, island 5
- temporary .phasiHuter_bowtieIndex directories are removed when -cl y is set
- after integration with -dp y, loci supported only by the phase-score method (not the hypergeometric test) are discarded
- on the maize transcriptome test dataset, the consolidated 21-PHAS gene count reproduces 181 with 96.7% (175/181) cross-tool overlap, and integration over genome+transcriptome yields 99 additional specific PHAS genes versus other software

## Limitations

- The phase module's sensitivity depends on max hits (-mh 10) during mapping; multi-mapping reads beyond this cutoff are lost.
- Phase-score-only results are considered weaker and are discarded by default during integration (-dp y), so loci detected solely by phase score will not be reported.
- The article provides only an update log (1.0.1 - 25/02/2024) with no discussion of limitations or reproducibility details.

## Evidence

- [abstract] PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods, to ensure the integrity and accuracy of prediction: "PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods"
- [readme] phase module method option: "-me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b"
- [readme] cutoff parameters: "-pv: float --  pvalue cutoff, default=0.001, only function with h/b method applied"
- [supplementary] dual-direction search finding: [evidence span withheld — anchor status: not_found]
- [methods] one-command defaults: "-pl: int   --  phase length, 21 | 24, default=21"
