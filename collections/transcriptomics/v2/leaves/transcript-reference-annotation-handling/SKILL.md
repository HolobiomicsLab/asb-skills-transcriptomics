---
name: transcript-reference-annotation-handling
description: Use when assembled transcript contigs must be assigned to a reference
  isoform annotation (e.g. GENCODE GTF) to quantify isoform-level precision and recall
  of a de novo assembler, as done with IsoQuant in the cloudrnaSPAdes evaluation.
when_to_use_negative:
- Input is still raw barcoded FASTQ that has not been assembled — annotation handling
  applies to assembled contigs, so run error correction (BayesHammer) and cloudrnaSPAdes
  first.
- No reference annotation is available or desired — cloudrnaSPAdes is designed for
  reference-free isoform assembly, so there is nothing to assign against.
- Goal is genome-assembly quality assessment (e.g. E. coli isolate benchmarks in the
  SPAdes manual) rather than isoform-level transcript evaluation.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: IsoQuant
  role: Assigns assembled contigs to annotated GENCODE v32 isoforms; assignments feed
    the per-gene precision/recall/fixed-recall computation.
- name: cloudrnaSPAdes
  role: Produces the per-gene contigs.fasta (reference-free isoform assembly from
    barcoded linked reads) that are the input to reference assignment.
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: InSilicoSeq
  role: Simulates the barcoded SPIso-seq-style paired-end reads from the three GRCh38/GENCODE
    v32 gene loci used to generate the annotation-assignment benchmark.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:8cab53b096d3b1278106688845350480387d64bdc7f6794cd5c1553e30cb6913
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - BayesHammer
  - InSilicoSeq
  - IsoQuant
  - SPAdes
  - cloudrnaSPAdes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# transcript-reference-annotation-handling

## Summary

Assign isoform-assembly contigs (e.g. cloudrnaSPAdes contigs.fasta for GYPC, MAPT and BIN1) to annotated GENCODE v32 reference isoforms with IsoQuant, then compute per-gene precision, recall, and 'Fixed recall' (after barcode-based read-cloud filtering) in gene_isoform_assignment_metrics.tsv. This is the reference-annotation half of benchmarking reference-free isoform assembly against annotated truth.

## When to use

Use when assembled transcript contigs must be assigned to a reference isoform annotation (e.g. GENCODE GTF) to quantify isoform-level precision and recall of a de novo assembler, as done with IsoQuant in the cloudrnaSPAdes evaluation.

## When NOT to use

- Input is still raw barcoded FASTQ that has not been assembled — annotation handling applies to assembled contigs, so run error correction (BayesHammer) and cloudrnaSPAdes first.
- No reference annotation is available or desired — cloudrnaSPAdes is designed for reference-free isoform assembly, so there is nothing to assign against.
- Goal is genome-assembly quality assessment (e.g. E. coli isolate benchmarks in the SPAdes manual) rather than isoform-level transcript evaluation.

## Inputs

- contigs.fasta per gene from cloudrnaSPAdes assembly (unfiltered and barcode-filtered runs)
- GENCODE v32 annotation for the target gene loci (GYPC, MAPT, BIN1)
- BayesHammer-corrected simulated barcoded SPIso-seq-style FASTQ reads (InSilicoSeq simulation of GRCh38 loci)

## Outputs

- gene_isoform_assignment_metrics.tsv with per-gene precision, recall, and fixed-recall values
- contig-to-GENCODE v32 isoform assignment map from IsoQuant (unfiltered and filtered assemblies)

## How to apply

Take the per-gene contigs.fasta produced by cloudrnaSPAdes (run on BayesHammer-corrected barcoded reads for each gene locus separately) and submit them to IsoQuant for assignment against the GENCODE v32 annotation for the corresponding gene (GYPC: 5 isoforms, MAPT: 16, BIN1: 15). Run the assignment twice — once on the unfiltered assembly and once after barcode-based read filtering that removes read clouds producing multiple isoforms for the same gene (the 'Fixed recall' procedure). From the IsoQuant assignments, compute per-gene precision, recall, and fixed-recall values and write them to gene_isoform_assignment_metrics.tsv. Judge correctness by the paper's thresholds: more than 98% of assembled isoforms should be accurately assigned to reference isoforms, and the filtered run should show a recall gain, exceeding the 'Separate strategy' competitor on the complex MAPT and BIN1 genes.

## Related tools

- **IsoQuant** (Assigns assembled contigs to annotated GENCODE v32 isoforms; assignments feed the per-gene precision/recall/fixed-recall computation.)
- **cloudrnaSPAdes** (Produces the per-gene contigs.fasta (reference-free isoform assembly from barcoded linked reads) that are the input to reference assignment.) — https://github.com/1dayac/cloudrnaSPAdes-release
- **InSilicoSeq** (Simulates the barcoded SPIso-seq-style paired-end reads from the three GRCh38/GENCODE v32 gene loci used to generate the annotation-assignment benchmark.)

## Evaluation signals

- Per-gene isoform-assignment accuracy from IsoQuant exceeds the reported >98% on GYPC, MAPT and BIN1.
- The barcode-filtered run shows a recall gain over the unfiltered run, with 'Fixed recall' exceeding the 'Separate strategy' competitor on MAPT and BIN1.
- gene_isoform_assignment_metrics.tsv contains precision, recall, and fixed-recall for each of the three genes.
- Assignment coverage is consistent with the annotated isoform counts per gene (GYPC: 5, MAPT: 16, BIN1: 15 in GENCODE v32).

## Limitations

- The reported accuracy (98% accurate isoform assignment) was established on simulated GYPC, MAPT and BIN1 datasets plus real human data; performance on other genes or annotations is not asserted in the available context.
- Article discussion-section content is absent and no changelog was found, so caveats on assignment methodology are not available from this source.
- SPAdes-based assembly was 'initially designed for small genomes' and is 'not intended for larger genomes (e.g. mammalian size genomes)', so per-gene loci rather than whole-genome input should be used.

## Evidence

- [abstract] cloudrnaSPAdes assembles isoforms assigned to reference annotations: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [abstract] Evaluation on simulated and real human data supports the accuracy claim: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
- [readme] Error-correction stage recommended before assembly whose contigs are annotated: "We recommend to run SPAdes with BayesHammer/IonHammer to obtain high-quality assemblies."
- [intro] SPAdes scope caveat for large genomes: "SPAdes was initially designed for small genomes. It was tested on bacterial (both single-cell MDA and standard isolates), fungal and other small genomes. SPAdes is not intended for larger genomes"
