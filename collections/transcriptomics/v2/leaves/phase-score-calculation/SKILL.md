---
name: phase-score-calculation
description: Use when detecting phased siRNA (phasiRNA) clusters/PHAS loci from sRNA-seq
  reads mapped to a transcriptome or genome and you need a combined phase-score +
  hypergeometric detection with bidirectional search.
when_to_use_negative:
- When only one algorithm is desired — use -me h (hypergeometric only) or -me p (phase
  score only) instead of the combined default, e.g. if downstream analysis discards
  only-P-method results (-dp y).
- When the input sRNA reads are untrimmed raw FASTQ — the phase module expects preprocessed
  map files and filtered FASTA produced by the `phasiHunter preprocess` step (trim
  adaptor, normalization, length/abundance filter, mapping).
tools:
- name: PhasiHunter
  role: executes the dual-algorithm (phase score + hypergeometric) PHAS loci detection
    with forward/reverse search via the `phasihunter phase` submodule
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: alignment engine underlying the .phasiHuter_bowtieIndex built by the phase
    module from the reference sequences
- name: Biopython
  role: dependency used within the PhasiHunter CLI implementation on the linux platform
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:8f0a888dffef49ef32d48065153f10aefdcf4cdd08be399baf66540ac5ab5c15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Biopython
  - Bowtie
  - PhasiHunter
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# phase-score-calculation

## Summary

PhasiHunter's PHAS loci detection runs two complementary phasiRNA prediction algorithms — a phase score method and a hypergeometric distribution-based method — with forward and reverse search to maximize PHAS loci detection sensitivity and prediction integrity.

## When to use

Use when detecting phased siRNA (phasiRNA) clusters/PHAS loci from sRNA-seq reads mapped to a transcriptome or genome and you need a combined phase-score + hypergeometric detection with bidirectional search.

## When NOT to use

- When only one algorithm is desired — use -me h (hypergeometric only) or -me p (phase score only) instead of the combined default, e.g. if downstream analysis discards only-P-method results (-dp y).
- When the input sRNA reads are untrimmed raw FASTQ — the phase module expects preprocessed map files and filtered FASTA produced by the `phasiHunter preprocess` step (trim adaptor, normalization, length/abundance filter, mapping).

## Inputs

- cDNA-based sRNA map file (-cm, e.g. SRR5049781_processed_cdna.map)
- genome-based sRNA map file (-gm, e.g. SRR5049781_processed_gdna.map)
- reference transcriptome FASTA (-c, e.g. oryza_sativa_cdna.fa)
- reference genome FASTA (-g, e.g. oryza_sativa_gdna.fa)
- length- and abundance-filtered sRNA FASTA (-fa, e.g. SRR5049781_trimmed_format_filter.fa)

## Outputs

- redundant allsiRNA cluster output (phase_a.txt)
- redundant phasiRNA cluster output (phase_o.txt) with columns: gene/PHAS_gene, strand, position, abundance, record, sequence, length, pvalue, phase_ratio, phase_number, phase_abundance, phase_score, marker

## How to apply

Run `phasiHunter phase` on cDNA-based (-cm) and genome-based (-gm) map files with the reference FASTAs (-c, -g) and the filtered sRNA FASTA (-fa). Select the combined method with `-me b` so both the phase score and the hypergeometric test are applied, and enable both forward and reverse search directions to enhance PHAS loci detection sensitivity. Use the demo-validated settings: island 5 (-il 5), phase length 21 (-pl 21), phase number 4 (-pn 4), max hits 10 (-mh 10), 20 parallel jobs (-j 20). Apply cutoffs p-value 0.001 (-pv 0.001, active only for h/b methods), phase score 15 (-ps 15, active only for p/b methods), and phase ratio 0.4 (-pr 0.4). Delete the temporary bowtie index with -cl y and collect the redundant allsiRNA cluster output (phase_a.txt) and phasiRNA cluster output (phase_o.txt).

## Related tools

- **PhasiHunter** (executes the dual-algorithm (phase score + hypergeometric) PHAS loci detection with forward/reverse search via the `phasihunter phase` submodule) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (alignment engine underlying the .phasiHuter_bowtieIndex built by the phase module from the reference sequences)
- **Biopython** (dependency used within the PhasiHunter CLI implementation on the linux platform)

## Examples

```
phasiHunter phase -cm SRR5049781_processed_cdna.map -c oryza_sativa_cdna.fa -gm SRR5049781_processed_gdna.map -g oryza_sativa_gdna.fa -fa SRR5049781_trimmed_format_filter.fa -a phase_a.txt -o phase_o.txt -me b -il 5 -pl 21 -pn 4 -pv 0.001 -ps 15 -pr 0.4 -cl y
```

## Evaluation signals

- Both output files (phase_a.txt and phase_o.txt) exist and their table headers include the documented pvalue and phase_score fields.
- Every reported locus satisfies the applied cutoffs: pvalue ≤ 0.001 (-pv), phase_score ≥ 15 (-ps), and phase_ratio ≥ 0.4 (-pr).
- Detected loci show the expected phasing parameters: phase length 21 (-pl 21) and phase number 4 (-pn 4) with island 5 (-il 5).
- Because both directions were searched (forward and reverse), hits on both strands are present rather than a single-strand bias.
- The temporary .phasiHuter_bowtieIndex was removed after prediction (-cl y).

## Limitations

- The -pv (pvalue) cutoff only functions when the h or b method is applied; the -ps (phase score) and -pr (phase ratio) cutoffs only function when the p or b method is applied, so single-method runs ignore the other threshold.
- Only prediction results from the phase score method can be discarded at integration (-dp y default), so the hypergeometric method's hits are always retained.
- Version provenance is weak: only an update log entry 1.0.1 (25/02/2024) is documented, with no discussion-section limitations or reproducibility details.

## Evidence

- [abstract] PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods, to ensure the integrity and accuracy of prediction: "PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods, to ensure the integrity and accuracy of prediction"
- [readme] -me method options: "-me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b"
- [readme] pvalue cutoff applies only to h/b methods: "-pv: float --  pvalue cutoff, default=0.001, only function with h/b method applied"
- [readme] phase ratio and phase score cutoffs apply only to p/b methods: "-pr: float --  phase ratio cutoff, default=0.4, only function with p/b method applied"
- [methods] demo phase command with all thresholds: "-me b -il 5 -pl 21 -pn 4 -mh 10 -j 20 -pv 0.001 -ps 15 -pr 0.4 -cl y"
