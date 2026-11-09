---
name: multi-reference-result-integration
description: Use when phasiRNA predictions have been run separately against cDNA,
  gdna, and/or flnc references (phase_a.txt/phase_o.txt outputs) and the goal is to
  merge them into one annotation-anchored result and quantify reference-specific loci.
when_to_use_negative:
- Only a single reference sequence was used, so there are no multi-reference phase
  outputs to integrate — run the phase module alone instead
- The goal is initiator/target prediction or differential expression of phasiRNAs
  (use the 'target'/'initiator' or 'deg' modules), not merging multi-reference PHAS
  loci
- No genome gff3 annotation is available and genomic anchoring of integrated loci
  is required but impossible
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: PhasiHunter
  role: Provides the integration module that merges cdna-, gdna-, and flnc-based phase
    results using the genome gff3 annotation, plus the phase and visulization modules
    feeding it
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bedtools
  role: Used to classify reference-unique PHAS loci by interval overlap with exon/intron
    features from the gff3 annotation
- name: Bowtie
  role: Maps preprocessed sRNA reads to each reference (transcriptome, genome, flnc)
    to produce the map files consumed by phase and integration
- name: Trim_galore
  role: Trims adapters during the preprocess step ('-m r') that generates the sRNA
    inputs for multi-reference prediction
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:2059967d81b40e62663c9c76f92ea3333024e812550490c510caa7a7dffb5947
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# multi-reference-result-integration

## Summary

Integrates phasiRNA/PHAS loci predictions generated against multiple reference sequences (transcriptome, genome, full-length transcriptome) with the PhasiHunter integration module, then computes reference-unique PHAS gene sets and classifies them by exon/intron structure.

## When to use

Use when phasiRNA predictions have been run separately against cDNA, gdna, and/or flnc references (phase_a.txt/phase_o.txt outputs) and the goal is to merge them into one annotation-anchored result and quantify reference-specific loci.

## When NOT to use

- Only a single reference sequence was used, so there are no multi-reference phase outputs to integrate — run the phase module alone instead
- The goal is initiator/target prediction or differential expression of phasiRNAs (use the 'target'/'initiator' or 'deg' modules), not merging multi-reference PHAS loci
- No genome gff3 annotation is available and genomic anchoring of integrated loci is required but impossible

## Inputs

- phase module outputs phase_o.txt (phasiRNA clusters) and phase_a.txt (allsiRNA clusters) from each reference (cdna-, gdna-, flnc-based)
- genome gff3 annotation file (e.g. oryza_sativa_gdna.gff3)
- reference sequence fasta files (transcriptome cdna.fa, genome gdna.fa, optional full-length flnc fasta)
- preprocessed sRNA map files (*_processed_cdna.map, *_processed_gdna.map, *_processed flnc map) with 19-25 nt length filter, CPM cutoff 1, normalization base 1000000, max hits 10

## Outputs

- integration_o.txt (integrated phasiRNA cluster output)
- integration_a.txt (integrated allsiRNA clusters)
- integration_s.txt
- integration_p.txt (PHAS loci records passing pvalue cutoff 0.001 with only-P-method results discarded)
- reference-unique PHAS gene sets and exon/intron classification table (e.g. phas_loci_comparison_summary.tsv)

## How to apply

Run 'phasiHunter integration' on the phase module outputs (-io phase_o.txt, -ia phase_a.txt) against the genome gff3 annotation (-an), with gdna-based PHAS loci enabled (-g y) and only-P-method results discarded (-dp y, default), reusing the prediction settings (phase length 21, phase number 4, pvalue cutoff 0.001, island 5). This produces integration_o.txt, integration_a.txt, integration_s.txt, and integration_p.txt that merge cdna-, gdna-, and flnc-based loci. Then compute set differences of PHAS gene IDs between references to obtain reference-unique loci (e.g. 30 transcriptome-unique vs 95 genome-unique in the maize test). Finally classify unique loci with the gff3 annotation and Bedtools interval overlap: genome-unique loci by intron/exon span (reported 69.5% spanning both intron and exon, 1% intronic) and transcriptome-unique loci by mono- vs multi-exon transcript status (reported 40%/60%). Rationale: multi-reference prediction captures loci such as alternatively spliced phasiRNAs that a single reference misses, and the gff3-anchored integration keeps results comparable across references.

## Related tools

- **PhasiHunter** (Provides the integration module that merges cdna-, gdna-, and flnc-based phase results using the genome gff3 annotation, plus the phase and visulization modules feeding it) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bedtools** (Used to classify reference-unique PHAS loci by interval overlap with exon/intron features from the gff3 annotation)
- **Bowtie** (Maps preprocessed sRNA reads to each reference (transcriptome, genome, flnc) to produce the map files consumed by phase and integration)
- **Trim_galore** (Trims adapters during the preprocess step ('-m r') that generates the sRNA inputs for multi-reference prediction) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
phasiHunter integration -io phase_o.txt -ia phase_a.txt -an oryza_sativa_gdna.gff3 -g y -o integration_o.txt -a integration_a.txt -s integration_s.txt -po integration_p.txt -pn 4 -pl 21 -pv 0.001 -il 5
```

## Evaluation signals

- integration_p.txt exists and contains PHAS loci records passing the hypergeometric pvalue cutoff 0.001, with only-P-method results discarded (-dp y)
- Set-difference counts of PHAS genes match reported values for the test dataset: 30 transcriptome-unique and 95 genome-unique loci
- Classification percentages reproduce the reported values: 60%/40% multi-/mono-exon for transcriptome-unique loci and 69.5% intron+exon-spanning, 1% intronic for genome-unique loci
- Integrated outputs (integration_o.txt, integration_a.txt, integration_s.txt, integration_p.txt) contain loci from all provided references (cdna, gdna, and flnc where supplied)
- The comparison summary table contains the required exon/intron classification fields for both reference-unique sets

## Limitations

- Only-P-method (phase score only) results are discarded by default (-dp y), so loci supported solely by the phase score method will not appear in integration_p.txt
- Only the update log 1.0.1 (25/02/2024) is given; no explicit software version, changelog-based limitations, or reproducibility details are extractable
- Genome-based prediction yields a much larger locus set than transcriptome-based prediction (95 vs 30 unique loci in the maize test), so reference choice strongly affects results and unique-loci classification depends on gff3 annotation quality

## Evidence

- [abstract] PhasiHunter can identify phasiRNAs and their regulatory networks based on multiple reference sequences and the predicted results can be automatically integrated: "PhasiHunter can identify phasiRNAs and their regulatory networks based on multiple reference sequences and the predicted results can be automatically integrated"
- [readme] integration module call with gff3 and cutoffs: "phasiHunter integration -io /home/user/test_osa/phase_o.txt -ia /home/user/test_osa/phase_a.txt -an /home/user/test_osa/oryza_sativa_gdna.gff3 -g y -o /home/user/test_osa/integration_o.txt"
- [methods] dp discard only P method result: "-dp: str   -- y | n, discard only P method result, default=y"
- [other] reference-unique counts from the maize comparison: [evidence span withheld — anchor status: not_found]
