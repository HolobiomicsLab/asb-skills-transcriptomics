---
name: promoter-sequence-retrieval-genomicfeatures
description: Use when testing whether a siRNA seed sequence matches promoter regions
  (e.g., to detect gene expression activation from high-dose GalNAc-siRNA treatments
  like Schlegel_2022_Ttr_D1_30mkg) instead of the standard transcript/3'UTR search
  space.
when_to_use_negative:
- The analysis goal is the standard seed-mediated off-target search over transcript
  3'UTRs — use the default load_annotations feature set with threeUTRsByTranscript
  instead of promoter extraction.
- No GTF/transcript annotation is available for the species/reference, since getPromoterSeq
  requires transcript ranges and genome sequence.
- The input is already a feature-table or match data frame rather than DESeq2 results
  and annotation objects — the workflow must start from results containing gene_id/log2FoldChange/padj.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SeedMatchR
  role: Loads species annotations (load_annotations), counts promoter seed matches
    (SeedMatchR, tx.id.col=TRUE for transcript-named sequences), and tests log2FC
    distributions (deseq_fc_ecdf)
  repo: https://github.com/tacazares/SeedMatchR
- name: GenomicFeatures
  role: getPromoterSeq-derived extraction of promoter sequences as the DNAStringSet
    feature set searched instead of transcript features
- name: AnnotationHub
  role: Supplies the rnor6 GTF and DNA 2bit resources (AH93578, AH95846) that GenomicFeatures
    builds on; requires >= 3.8.0
- name: DESeq2
  role: Produces the differential expression results frame (gene_id, log2FoldChange,
    padj) that promoter seed-match counts are joined to and compared with
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:2e15566cac36b0b160c09553a7612dff2dc3d74c47fee601cf2ca087de1db48a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tacazares/SeedMatchR
  tools:
  - AnnotationHub
  - DESeq2
  - GenomicFeatures
  - R
  - SeedMatchR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# promoter-sequence-retrieval-genomicfeatures

## Summary

Retrieve promoter-region sequences as the SeedMatchR feature set using GenomicFeatures::getPromoterSeq on rnor6 annotations, replacing the default transcript/3'UTR search space so seed matches can be counted against promoter DNA to probe siRNA-driven gene activation at high doses.

## When to use

Use when testing whether a siRNA seed sequence matches promoter regions (e.g., to detect gene expression activation from high-dose GalNAc-siRNA treatments like Schlegel_2022_Ttr_D1_30mkg) instead of the standard transcript/3'UTR search space.

## When NOT to use

- The analysis goal is the standard seed-mediated off-target search over transcript 3'UTRs — use the default load_annotations feature set with threeUTRsByTranscript instead of promoter extraction.
- No GTF/transcript annotation is available for the species/reference, since getPromoterSeq requires transcript ranges and genome sequence.
- The input is already a feature-table or match data frame rather than DESeq2 results and annotation objects — the workflow must start from results containing gene_id/log2FoldChange/padj.

## Inputs

- Rat rnor6 annotation (GTF + DNA via AnnotationHub, e.g. AH93578/AH95846) loaded with load_annotations(canonical=FALSE, min.feature.width=8, longest.utr=TRUE)
- Guide siRNA sequence, e.g. 'UUAUAGAGCAAGAACACUGUUUU' targeting Ttr (Schlegel et al. 2022)
- DESeq2 results data frame with gene_id, log2FoldChange, and padj columns (e.g. Schlegel_2022_Ttr_D1_30mkg from GSE184929)

## Outputs

- DNAStringSet of promoter sequences named by transcript ID
- DESeq2 results annotated with promoter seed-match counts (mer7m8, mer8 columns per gene)
- ECDF plot of log2 fold changes plus KS test statistic and p-value comparing promoter seed-match genes vs. background

## How to apply

First build the rat rnor6 annotation with SeedMatchR::load_annotations(reference.name='rnor6', canonical=FALSE, min.feature.width=8, longest.utr=TRUE) — rnor6 cannot use ensembldb::TxIsCanonicalFilter, so canonical must be FALSE. From the ensembldb/GTF-derived transcript ranges and DNA, use GenomicFeatures::getPromoterSeq to extract promoter sequences as a DNAStringSet named by transcript ID. Because the sequence names are transcript IDs rather than gene IDs, run SeedMatchR with tx.id.col=TRUE so transcript-level matches are summarized to the gene level for joining to DESeq2 results (columns gene_id, log2FoldChange, padj required). Extract the seed with get_seed(guide.seq='UUAUAGAGCAAGAACACUGUUUU', seed.name='mer7m8'), annotate promoter seed-match counts, build mutually exclusive gene sets (promoter mer7m8 >=1 with mer8 == 0 vs. background with 0 matches for both), and compare log2 fold-change distributions with deseq_fc_ecdf(stats_test='KS', alternative='greater').

## Related tools

- **SeedMatchR** (Loads species annotations (load_annotations), counts promoter seed matches (SeedMatchR, tx.id.col=TRUE for transcript-named sequences), and tests log2FC distributions (deseq_fc_ecdf)) — https://github.com/tacazares/SeedMatchR
- **GenomicFeatures** (getPromoterSeq-derived extraction of promoter sequences as the DNAStringSet feature set searched instead of transcript features)
- **AnnotationHub** (Supplies the rnor6 GTF and DNA 2bit resources (AH93578, AH95846) that GenomicFeatures builds on; requires >= 3.8.0)
- **DESeq2** (Produces the differential expression results frame (gene_id, log2FoldChange, padj) that promoter seed-match counts are joined to and compared with)

## Examples

```
annodb = load_annotations(reference.name = "rnor6", canonical = FALSE, min.feature.width = 8, longest.utr = TRUE)
```

## Evaluation signals

- ECDF/KS analysis (alternative='greater') shows a slight enrichment of positive log2 fold changes among promoter mer7m8-match genes relative to background, consistent with the Fig. 1G pattern for Ttr D1 30 mg/kg.
- The KS test statistic and p-value are returned together with the deseq_fc_ecdf plot.
- Seed-match counting is run twice (mer7m8 and mer8) so the match gene set is mutually exclusive (mer7m8 >= 1 and mer8 == 0) against a background with zero matches for both seeds.
- Sequence names in the promoter DNAStringSet are transcript IDs, so tx.id.col=TRUE is set and matches are summarized to gene level before joining to res$gene_id.
- load_annotations for rnor6 is called with canonical=FALSE (rnor6 cannot use ensembldb::TxIsCanonicalFilter).

## Limitations

- rnor6 and mm10 cannot use ensembldb::TxIsCanonicalFilter, so the canonical argument of load_annotations must be FALSE for these references.
- rnor6/rnor7 cannot be used with ensembldb::TxSupportLevelFilter, so transcript.support should not be passed for rat transcriptomes.
- rnor7 additionally requires R >= 4.3.0 and AnnotationHub >= 3.8.0.
- Loading annotations from AnnotationHub takes several minutes and should be cached as an Rdata object for repeated workflows.

## Evidence

- [other] promoter seed-match activation finding: [evidence span withheld — anchor status: not_found]
- [readme] transcript-named sequences require tx.id.col: [evidence span withheld — anchor status: not_found]
- [methods] built-in annotation loading: "The function `load_annotations()` has built in annotation data for human, rat, and mouse annotations."
- [methods] GenomicFeatures sequence extraction context: "Examples include those generated by `GenomicFeatures::extractTranscriptSeqs()` paired with functions like `GenomicFeatures::threeUTRsByTranscript()`."
- [readme] ecdf comparison method: "Cumulative changes in the profile of genes expression can be visualized and tested with the emperical distribution function (ecdf) coupled with a statistical test"
