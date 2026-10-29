---
name: chimera-removal-count-analysis
description: Use when a Trinity de novo transcriptome assembly (with paired Illumina
  RNA-Seq reads) must be screened for chimeric contigs and you need per-step chimera
  removal counts, e.g. testing filter orders or validating ~97% chimera removal.
when_to_use_negative:
- Input is a genome-guided or reference-based assembly where chimeras can be checked
  directly against a reference genome instead of reference-free BLASTX marking
- No RNA-Seq read data is available, since the TPM filter (cut-off 1) and TransRate
  require read mapping to the assembly
- The assembly is known to contain many contigs from alternative splicing of a single
  gene, which TransRate may misclassify as chimeras
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Bellerophon
  role: Pipeline orchestrating the TPM, CD-HIT-EST and TransRate filtering steps and
    tracking chimera removal
  repo: github.com/JesseKerkvliet/Bellerophon
- name: TransRate
  role: TransRate-C filtering step that removes chimeric contigs and assesses assembly
    quality
- name: CD-HIT-EST
  role: Removes redundant/highly identical contigs at 95% identity, contributing to
    chimera removal
- name: BLASTX
  role: Marks chimeric contigs by detecting two non-overlapping matches to different
    genes against NCBI NR (E-value 1e-4)
- name: Trinity
  role: De novo assembler producing the transcriptome assemblies in which chimeras
    are quantified
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:f464525299e6427a896f5e794ebee6a12397c49eaefb1891875bf78dad178c35
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTX
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  - Trinity
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# chimera-removal-count-analysis

## Summary

A benchmarking skill for the Bellerophon pipeline that quantifies how many chimeric contigs are removed at each filtering step (TPM cut-off 1, CD-HIT-EST 95% identity, TransRate-C) of a de novo transcriptome assembly, and compares chimera vs. non-chimera removal rates.

## When to use

Use when a Trinity de novo transcriptome assembly (with paired Illumina RNA-Seq reads) must be screened for chimeric contigs and you need per-step chimera removal counts, e.g. testing filter orders or validating ~97% chimera removal.

## When NOT to use

- Input is a genome-guided or reference-based assembly where chimeras can be checked directly against a reference genome instead of reference-free BLASTX marking
- No RNA-Seq read data is available, since the TPM filter (cut-off 1) and TransRate require read mapping to the assembly
- The assembly is known to contain many contigs from alternative splicing of a single gene, which TransRate may misclassify as chimeras

## Inputs

- De novo transcriptome assembly in FASTA format (e.g. Trinity assembly of H. subflexa pheromone gland transcriptome)
- Paired-end Illumina RNA-Seq FASTQ reads (left R1 and right R2 files)
- Optional: set of computationally created chimeras (500 synthetic chimeras built by concatenating sequence pairs with 30-70% random overlap), repeated five times
- NCBI NR protein database for BLASTX chimera marking

## Outputs

- Per-filter-step chimera count table (chimeras remaining after TPM, CD-HIT-EST, and TransRate-C steps)
- Filtered transcriptome assembly (FASTA) after each filter stage
- TransRate quality scores and BUSCO assessments of filtered assemblies
- Chimera vs. non-chimera removal percentage comparison with statistical test results

## How to apply

Mark chimeric contigs by BLASTX against NCBI NR (E-value 1e-4), flagging sequences with two non-overlapping matches to different genes. Run Bellerophon with default filtering: TPM filter at cut-off 1, CD-HIT-EST at 95% identity, then TransRate filtering, supplying the de novo assembly (FASTA) and paired-end FASTQ reads. After each filtering step, re-run BLASTX chimera marking and count remaining marked chimeras to build a per-step removal table. To test filter ordering (e.g. TransRate-C before RSEM-based TPM filtering), re-run with the custom order and re-count chimeras — the paper reports this variant reduced isoform-rich contig-group chimeras from 74 to 18. Compare per-step chimera removal percentages against non-chimera removal with an unpaired t test plus Bonferroni correction; the paper reports 97.04% ± 0.61 of 500 planted chimeras removed versus 69.89% ± 0.03 of other sequences (adjusted p = 5.83 × 10−6).

## Related tools

- **Bellerophon** (Pipeline orchestrating the TPM, CD-HIT-EST and TransRate filtering steps and tracking chimera removal) — github.com/JesseKerkvliet/Bellerophon
- **TransRate** (TransRate-C filtering step that removes chimeric contigs and assesses assembly quality)
- **CD-HIT-EST** (Removes redundant/highly identical contigs at 95% identity, contributing to chimera removal)
- **BLASTX** (Marks chimeric contigs by detecting two non-overlapping matches to different genes against NCBI NR (E-value 1e-4))
- **Trinity** (De novo assembler producing the transcriptome assemblies in which chimeras are quantified)

## Examples

```
python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- Overall chimera removal on 500 planted chimeras should be approximately 97% (about 485 of 500) versus roughly 70% removal of non-chimeric sequences, with the TPM step alone discarding about 96% of chimeras (95.72% vs 60.77% for others)
- Per-step accounting: TPM filtering first removed 227 (8.5%) of chimeras, CD-HIT-EST removed 600 (24.5%) of remaining chimeras, and TransRate-C removed 792 (42.9%) in the paper's tracked run
- Variant order with TransRate-C before RSEM/TPM filtering should reduce chimeric isoforms in the isoform-rich contig groups from 74 to 18 (56 removed), matching the paper's benchmark
- BLASTX marking must use the chimera criterion of two non-overlapping matches to different genes with E-value cut-off 1e-4
- The unpaired t test with Bonferroni correction should show significantly higher chimera than non-chimera removal (adjusted p = 5.83 × 10−6)

## Limitations

- Contigs representing alternative splicing of one gene might be considered chimeras by TransRate if unevenly expressed along their sequence, so Bellerophon may erroneously remove legitimate isoforms
- Custom order of filtering steps is disabled in the current version of the tool (README pending issue)
- The Vagrant VM disk size is not large enough for most full transcriptome analyses; output must be written to the /vagrant/ shared directory
- Backwards compatibility problems with TransRate are avoided by using the provided Vagrant box

## Evidence

- [readme] Bellerophon is a pipeline created to remove falsely assembled chimeric transcripts in de novo transcriptome assemblies.: "Bellerophon is a pipeline created to remove falsely assembled chimeric transcripts in de novo transcriptome assemblies."
- [methods] Bellerophon filters the assembly using a TPM filter with a default cut-off score of 1.: [evidence span withheld — anchor status: not_found]
- [methods] CDHIT-EST to filter redundant contigs, using a default cut-off of 95% identity.: [evidence span withheld — anchor status: not_found]
- [methods] Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes.: "Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes."
- [results] The final Transrate‐C filtering step removed 792 (42.9%) chimeras: "The final Transrate‐C filtering step removed 792 (42.9%) chimeras"
