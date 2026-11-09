---
name: transcriptome-chimera-filtering-pipeline-execution
description: Use when a de novo assembled transcriptome (e.g. Trinity contigs from
  FASTQ reads) needs chimera removal and quality improvement before downstream gene
  discovery or differential expression analyses.
when_to_use_negative:
- When contigs representative of alternative splicing of one gene must be preserved,
  since unevenly expressed isoforms may be misclassified as chimeras by TransRate
- When a custom order of filtering steps is required — the README notes 'Custom order
  of filtering steps is disabled in this version'
- When working from a reference-guided assembly rather than a de novo transcriptome,
  since the method targets reference-free de novo assembly improvement
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Bellerophon
  role: The pipeline being executed; orchestrates quality assessment and filtering
    steps to remove chimeric contigs
  repo: github.com/JesseKerkvliet/Bellerophon
- name: TransRate
  role: Reference-free quality assessment of the assembly and final TransRate filtering
    step after TPM and CD-HIT-EST
- name: CD-HIT-EST
  role: Removes redundant/highly identical contigs at a default 95% identity cut-off
- name: Trinity
  role: De novo transcriptome assembler producing the input assembly (contigs matching
    fewer than 5 reads removed)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:68ed91e943ffb0e272e7b5248d1712f5c8fd7cbd8324962c38ec36e33227540d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTn
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  - Trinity
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# transcriptome-chimera-filtering-pipeline-execution

## Summary

Execute the Bellerophon pipeline to remove falsely assembled chimeric transcripts from de novo transcriptome assemblies, using TransRate/BUSCO quality assessment followed by TPM filtering (cut-off 1), CD-HIT-EST (95% identity), and TransRate filtering.

## When to use

Use when a de novo assembled transcriptome (e.g. Trinity contigs from FASTQ reads) needs chimera removal and quality improvement before downstream gene discovery or differential expression analyses.

## When NOT to use

- When contigs representative of alternative splicing of one gene must be preserved, since unevenly expressed isoforms may be misclassified as chimeras by TransRate
- When a custom order of filtering steps is required — the README notes 'Custom order of filtering steps is disabled in this version'
- When working from a reference-guided assembly rather than a de novo transcriptome, since the method targets reference-free de novo assembly improvement

## Inputs

- De novo assembled transcriptome in FASTA format (e.g. Trinity assembly of RNA-Seq reads)
- Paired-end RNA-Seq reads in FASTQ (left/right files, required by TransRate for reference-free quality assessment)
- Optional reference transcript set for post-hoc chimera classification via BLAST

## Outputs

- Filtered transcriptome assembly with reduced chimeric, redundant, and lowly expressed contigs
- Quality assessment reports for the unfiltered and filtered assemblies (TransRate scores, BUSCO)
- Contig classification (chimeric / nonchimeric / unidentified) if BLAST against a reference is performed

## How to apply

Run Bellerophon in its default order: first establish a ground quality score of the unfiltered assembly with Busco and TransRate, then apply a TPM filter with default cut-off 1 to remove lowly expressed contigs, then CD-HIT-EST with default 95% identity to remove redundant/highly identical contigs, and finally TransRate's filtering capabilities. The pipeline was designed for de novo assemblies where no reference genome is available; assemblies were typically built with Trinity (removing contigs matching fewer than 5 reads). Downstream, filtered contigs can be classified by BLAST against reference transcripts (e.g. E-value <= 1e-3, identity >= 90%, alignment length >= 300 nt) as chimeric (multiple non-overlapping matches to different genes), nonchimeric, or unidentified. In the paper's D. melanogaster SRA benchmark, Bellerophon removed 60.6% of chimeras, 54% of correct sequences, and 58% of unidentified contigs, and the number of unique genes dropped from 10,527 to 9,148.

## Related tools

- **Bellerophon** (The pipeline being executed; orchestrates quality assessment and filtering steps to remove chimeric contigs) — github.com/JesseKerkvliet/Bellerophon
- **TransRate** (Reference-free quality assessment of the assembly and final TransRate filtering step after TPM and CD-HIT-EST)
- **CD-HIT-EST** (Removes redundant/highly identical contigs at a default 95% identity cut-off)
- **Trinity** (De novo transcriptome assembler producing the input assembly (contigs matching fewer than 5 reads removed))

## Examples

```
python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- TransRate score of the filtered assembly should improve over the ground score of the unfiltered assembly (in the H. subflexa experiment the CD-HIT-EST-only filter scored 0.319)
- Reproduced chimera removal should fall in the reported range: 40%–91.9% of chimeras removed, with the D. melanogaster SRA benchmark at ~60.6% chimeras, ~54% correct, ~58% unidentified removed
- Unique-gene count reduction should be checked against expectation (10,527 to 9,148 in the D. melanogaster benchmark)
- Pipeline must run to completion inside the Vagrant VM or conda environment (conda env create envs/Bellerophon.yml) with TransRate compatibility intact
- BLAST-based validation should classify contigs using E-value <= 1e-3, identity >= 90%, alignment length >= 300 nt, chimeras defined as two non-overlapping matches to different genes

## Limitations

- Bellerophon may erroneously remove contigs from alternative splicing that are misclassified as chimeras by TransRate when isoforms are unevenly expressed along their sequence
- Custom order of filtering steps is disabled in the current version (README pending issue)
- The Vagrant VM disk size is not large enough for most full transcriptome analyses; output must go to /vagrant/ on the host system
- ORF-length filtering removed too many transcripts and is not in the default order (default is TPM then CD-HIT-EST)

## Evidence

- [abstract] The developed method is incorporated into a pipeline, which we named Bellerophon, that is broadly applicable and easy to use.: "The developed method is incorporated into a pipeline, which we named Bellerophon, that is broadly applicable and easy to use."
- [methods] Bellerophon filters the assembly using a TPM filter with a default cut-off score of 1.: [evidence span withheld — anchor status: not_found]
- [methods] Bellerophon then uses CDHIT-EST to filter redundant contigs, using a default cut-off of 95% identity.: [evidence span withheld — anchor status: not_found]
- [methods] Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes.: "Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes."
- [abstract] the Bellerophon pipeline was able to remove between 40% and 91.9% of the chimeras in transcriptome assemblies: "the Bellerophon pipeline was able to remove between 40% and 91.9% of the chimeras in transcriptome assemblies"
- [discussion] Contigs representative of transcripts arising from alternative splicing of one gene might be considered as chimera by Transrate: "Contigs representative of transcripts arising from alternative splicing of one gene might be considered as chimera by Transrate if they are unevenly expressed along their sequence"
- [readme] python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun: "python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun"
