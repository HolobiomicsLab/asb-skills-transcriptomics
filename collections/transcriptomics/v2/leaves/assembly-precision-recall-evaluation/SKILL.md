---
name: assembly-precision-recall-evaluation
description: Use when you have assembled contigs from barcoded RNA-seq linked reads
  (e.g. cloudrnaSPAdes output) plus a reference annotation, and need to quantify which
  isoforms were reconstructed accurately per gene.
when_to_use_negative:
- Input reads are standard (non-barcoded) short-read RNA-seq without linked-read barcodes,
  since the read-cloud filtering strategy depends on barcode grouping
- Genome-scale assembly where SPAdes is inappropriate — the README states 'SPAdes
  was initially designed for small genomes' and 'is not intended for larger genomes
  (e.g. mammalian size genomes)'
- Assemblies already verified by other means (e.g. full-length long-read validation)
  where IsoQuant reassignment would add no information
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3308
tools:
- name: cloudrnaSPAdes
  role: Feature release of SPAdes for reference-free isoform assembly from barcoded
    RNA-seq linked reads; produces the contigs.fasta evaluated here
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: BayesHammer
  role: SPAdes read error correction module applied to the simulated FASTQ files before
    assembly, per the recommendation to run it for high-quality assemblies
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: IsoQuant
  role: Assigns assembled contigs to annotated GENCODE v32 isoforms to yield precision/recall
    per gene
- name: InSilicoSeq
  role: Simulates barcoded SPIso-seq-style paired-end reads from the GYPC, MAPT, and
    BIN1 loci for the benchmark
- name: Bandage
  role: GUI visualization of the assembly graphs to inspect read-cloud/isoform graph
    structure during troubleshooting
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:3b6a748629ee362c19e7b0fa036157174207b10018910b82a032d719a97c2732
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - Bandage
  - BayesHammer
  - InSilicoSeq
  - IsoQuant
  - SPAdes
  - cloudrnaSPAdes
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# assembly-precision-recall-evaluation

## Summary

Evaluates de novo isoform assemblies by assigning assembled contigs to annotated reference isoforms (e.g. IsoQuant against GENCODE v32) and computing per-gene precision, recall, and 'fixed recall' so that assembly accuracy and the effect of barcode-based read-cloud filtering can be quantified.

## When to use

Use when you have assembled contigs from barcoded RNA-seq linked reads (e.g. cloudrnaSPAdes output) plus a reference annotation, and need to quantify which isoforms were reconstructed accurately per gene.

## When NOT to use

- Input reads are standard (non-barcoded) short-read RNA-seq without linked-read barcodes, since the read-cloud filtering strategy depends on barcode grouping
- Genome-scale assembly where SPAdes is inappropriate — the README states 'SPAdes was initially designed for small genomes' and 'is not intended for larger genomes (e.g. mammalian size genomes)'
- Assemblies already verified by other means (e.g. full-length long-read validation) where IsoQuant reassignment would add no information

## Inputs

- contigs.fasta per gene from cloudrnaSPAdes (unfiltered and barcode-filtered runs)
- Barcoded SPIso-seq-style paired-end FASTQ reads (e.g. simulated with InSilicoSeq from GRCh38/GENCODE v32 loci)
- Reference transcript annotation, e.g. GENCODE v32 GTF/FASTA for the target gene loci (GYPC, MAPT, BIN1)

## Outputs

- gene_isoform_assignment_metrics.tsv with per-gene precision, recall, and fixed-recall values
- IsoQuant isoform-to-contig assignment report per assembly run
- Per-gene accuracy summary (assignment accuracy should exceed 98%)

## How to apply

Assemble each gene locus separately with cloudrnaSPAdes (after BayesHammer error correction of the simulated SPIso-seq-style FASTQs), producing contigs.fasta per gene. Run the assembly twice: unfiltered, and after barcode-based read filtering that removes read clouds producing multiple isoforms for the same gene (the 'Fixed recall' procedure). Assign contigs to annotated isoforms with IsoQuant for both runs — e.g. against GENCODE v32 for GYPC (5 annotated isoforms), MAPT (16), and BIN1 (15) — then compute per-gene precision, recall, and fixed-recall from the assignments and write gene_isoform_assignment_metrics.tsv. The rationale is that reference-free isoform assembly can produce extra contigs (hurting precision) and miss low-coverage isoforms (hurting recall), so filtering ambiguous read clouds is expected to shift recall, particularly for highly multi-isoform genes like MAPT and BIN1.

## Related tools

- **cloudrnaSPAdes** (Feature release of SPAdes for reference-free isoform assembly from barcoded RNA-seq linked reads; produces the contigs.fasta evaluated here) — https://github.com/1dayac/cloudrnaSPAdes-release
- **BayesHammer** (SPAdes read error correction module applied to the simulated FASTQ files before assembly, per the recommendation to run it for high-quality assemblies) — https://github.com/1dayac/cloudrnaSPAdes-release
- **IsoQuant** (Assigns assembled contigs to annotated GENCODE v32 isoforms to yield precision/recall per gene)
- **InSilicoSeq** (Simulates barcoded SPIso-seq-style paired-end reads from the GYPC, MAPT, and BIN1 loci for the benchmark)
- **Bandage** (GUI visualization of the assembly graphs to inspect read-cloud/isoform graph structure during troubleshooting) — https://github.com/rrwick/Bandage

## Examples

```
spades.py --only-error-correction -1 gypc_sim_R1.fastq -2 gypc_sim_R2.fastq -o gypc_ec && cloudrnaspades.py --pe-corrected gypc_ec/corrected/*.fastq -o gypc_asm
```

## Evaluation signals

- Per-gene isoform-assignment accuracy from IsoQuant exceeds the reported >98% threshold for GYPC, MAPT, and BIN1
- gene_isoform_assignment_metrics.tsv contains non-empty precision, recall, and fixed-recall values for all three genes
- The barcode-filtered ('Fixed recall') run shows a recall gain over the unfiltered run for the complex MAPT (16 isoforms) and BIN1 (15 isoforms) loci
- Filtered-run recall on MAPT and BIN1 exceeds the 'Separate strategy' competitor as reported in the paper's benchmark
- Contigs from every assembly run are assignable to a GENCODE v32 isoform rather than remaining unclassified (spot-checkable in IsoQuant output)

## Limitations

- The >98% accuracy and recall-gain results were demonstrated on simulated human data for three selected genes; they do not guarantee generalization to real or other-locus data without re-benchmarking
- Barcode-based filtering removes read clouds that produce multiple isoforms for the same gene, which can trade precision against recall — the unfiltered run should be retained for comparison
- SPAdes 3.13.1 was not intended for mammalian-size genomes; the benchmark works only because assembly is run per gene locus, not genome-wide

## Evidence

- [abstract] we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [abstract] Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity.: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
- [readme] BayesHammer read correction recommendation: "We recommend to run SPAdes with BayesHammer/IonHammer to obtain high-quality assemblies."
- [methods] spades-read-filter description: "spades-read-filter is a tool for filtering reads with median kmer coverage less than threshold."
- [readme] BayesHammer tool description: "BayesHammer – read error correction tool for Illumina reads, which works well on both single-cell and standard data sets."
