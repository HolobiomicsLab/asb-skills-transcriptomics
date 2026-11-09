---
name: phasirna-cluster-integration-across-reference-sequences
description: Use when separate PhasiHunter phase runs on gDNA and cDNA (or full-length
  transcriptome) references have produced phase_o.txt and phase_a.txt outputs and
  the clusters must be merged into a single genome-annotated result set.
when_to_use_negative:
- You have only run PhasiHunter's phase module against a single reference sequence
  and need no cross-reference merging or coordinate transform — the phase module outputs
  are already the final per-reference result.
- You need to predict phasiRNAs or PHAS loci themselves rather than merge/annotate
  existing phase predictions — use the phase module with its -me h|p|b method selection
  instead.
- Your input is raw sRNA-seq FASTQ data not yet preprocessed or phased — the integration
  module requires phase_o.txt/phase_a.txt from completed phase runs, not raw reads.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: PhasiHunter
  role: provides the `integration` submodule that deduplicates, merges, annotates,
    and classifies phasiRNA clusters across coordinate systems
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bedtools
  role: implements the coordinate system transform between genome- and transcriptome-based
    cluster predictions
- name: Biopython
  role: python3 dependency used by PhasiHunter (dictionary data structure for merging
    results)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:20225aaea5579b1596666d87983c66f70d33773b7a88078669a80cbbc89d5518
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Bedtools
  - Biopython
  - Bowtie
  - PhasiHunter
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# phasiRNA-cluster-integration-across-reference-sequences

## Summary

Integrates phasiRNA/PHAS loci and all-siRNA cluster predictions that PhasiHunter's phase module produced separately against multiple reference sequences (genome/gDNA, transcriptome/cDNA, full-length transcriptome) by transforming coordinates with bedtools and a python3 dictionary, then deduplicating, merging, annotating, and classifying the clusters.

## When to use

Use when separate PhasiHunter phase runs on gDNA and cDNA (or full-length transcriptome) references have produced phase_o.txt and phase_a.txt outputs and the clusters must be merged into a single genome-annotated result set.

## When NOT to use

- You have only run PhasiHunter's phase module against a single reference sequence and need no cross-reference merging or coordinate transform — the phase module outputs are already the final per-reference result.
- You need to predict phasiRNAs or PHAS loci themselves rather than merge/annotate existing phase predictions — use the phase module with its -me h|p|b method selection instead.
- Your input is raw sRNA-seq FASTQ data not yet preprocessed or phased — the integration module requires phase_o.txt/phase_a.txt from completed phase runs, not raw reads.

## Inputs

- phase module phasiRNA cluster output file (phase_o.txt, passed via -io)
- phase module all-siRNA cluster output file (phase_a.txt, passed via -ia)
- reference genome GFF3 annotation file (e.g. oryza_sativa_gdna.gff3, passed via -an)
- flag enabling gDNA-based PHAS loci (-g y)
- cutoff parameters matching the phase run: -pn 4, -pl 21, -pv 0.001, -il 5

## Outputs

- integration_o.txt — integrated phasiRNA clusters (columns: PHAS_gene, strand, phasiRNA_position, phasiRNA_abundance, phasiRNA_record, phasiRNA_sequence, phasiRNA_length, phase_ratio, phase_number, phase_abundance, phase_score, pvalue, PHAS_gene_annotation, marker)
- integration_a.txt — integrated all-siRNA clusters
- integration_s.txt — integration summary
- integration_p.txt — PHAS loci information

## How to apply

Run `phasiHunter integration` taking the phase module outputs (-io phase_o.txt phasiRNA clusters and -ia phase_a.txt all-siRNA clusters) from the cDNA and gDNA reference mappings, together with a reference genome GFF3 annotation via -an, and enable gDNA-based PHAS loci with `-g y` so genome- and transcriptome-coordinate clusters are jointly processed. The module performs a coordinate system transform implemented with bedtools and the python3 dictionary data structure, then deduplicates and merges overlapping phasiRNA clusters across the two coordinate systems under the same cutoffs used in the phase step (phase number cutoff 4 via -pn, phase length 21 via -pl, pvalue cutoff 0.001 via -pv, cluster island 5 via -il). Prediction results supported only by the phase-score method are discarded (discard only P method result = y), consistent with PhasiHunter's use of both phase score and hypergeometric-distribution methods to ensure prediction integrity. The integrated PHAS loci and siRNA clusters are then annotated against the GFF3 gene annotation and emitted as four outputs: integration_o.txt, integration_a.txt, integration_s.txt, and integration_p.txt.

## Related tools

- **PhasiHunter** (provides the `integration` submodule that deduplicates, merges, annotates, and classifies phasiRNA clusters across coordinate systems) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bedtools** (implements the coordinate system transform between genome- and transcriptome-based cluster predictions)
- **Biopython** (python3 dependency used by PhasiHunter (dictionary data structure for merging results))

## Examples

```
phasiHunter integration -io /home/user/test_osa/phase_o.txt -ia /home/user/test_osa/phase_a.txt -an /home/user/test_osa/oryza_sativa_gdna.gff3 -g y -o /home/user/test_osa/integration_o.txt -a /home/user/test_osa/integration_a.txt -s /home/user/test_osa/integration_s.txt -po /home/user/test_osa/integration_p.txt -j 1 -pn 4 -pl 21 -pv 0.001 -il 5
```

## Evaluation signals

- integration_s.txt exists after the run (the workflow validates this file as the success signal).
- integration_o.txt contains the expected table header fields: PHAS_gene, strand, phasiRNA_position, phasiRNA_abundance, phasiRNA_record, phasiRNA_sequence, phasiRNA_length, phase_ratio, phase_number, phase_abundance, phase_score, pvalue, PHAS_gene_annotation, marker.
- The cutoffs passed to integration (-pn 4, -pl 21, -pv 0.001, -il 5) match those used in the upstream phase module run.
- Clusters that appear in both gDNA- and cDNA-coordinate predictions are merged/deduplicated rather than duplicated across rows.
- PHAS_gene_annotation fields in integration_o.txt are populated from the supplied GFF3 gene annotation.

## Limitations

- Integration requires phase outputs from multiple reference mappings; it cannot create cross-reference results from a single phase run.
- By default (-dp y / discard only P method result = y) results supported only by the phase-score method are discarded, so phase-score-only clusters will not appear in integrated output.
- No changelog or discussion-section limitations were available; reproducibility details beyond the README workflow are not documented (only update log 1.0.1 given).

## Evidence

- [other] integration via bedtools coordinate transform and dictionary merging: [evidence span withheld — anchor status: not_found]
- [methods] integration module responsibilities: [evidence span withheld — anchor status: not_found]
- [readme] integration command line: "phasiHunter integration -io /home/user/test_osa/phase_o.txt -ia /home/user/test_osa/phase_a.txt -an /home/user/test_osa/oryza_sativa_gdna.gff3 -g y"
- [abstract] two prediction algorithms rationale: "PhasiHunter employs two major phasiRNA prediction algorithms, namely phase score and hypergeometric distribution-based methods, to ensure the integrity and accuracy of prediction"
- [abstract] automatic integration capability: "PhasiHunter can identify phasiRNAs and their regulatory networks based on multiple reference sequences and the predicted results can be automatically integrated"
