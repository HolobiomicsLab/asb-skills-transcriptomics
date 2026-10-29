---
name: kta-ari-correlation-analysis
description: Use when you need to check that KTA-based unsupervised selection of the
  transfer mixture parameter theta coincides with the theta maximizing supervised
  ARI in simulated scRNA-Seq transfer learning experiments.
when_to_use_negative:
- 'Real target datasets without ground-truth labels: supervised ARI cannot be computed
  (the README notes supervised accuracy remains 0 when no ground truth labels are
  given), so this validation is restricted to simulated settings'
- When the mixture parameter theta has already been fixed (e.g. by KTA alone) and
  only needs applying — this skill is for validating the selection itself
- Non-simulated data (e.g. Hockley/Usoskin or subsampled Tasic experiments) where
  no simulation ground truth exists for ARI correlation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SC3
  role: Clusters each modified target dataset Xtrg_new so that the kernel of the predicted
    cell-type labels can be compared to the kernel of Xtrg_new via KTA
- name: scRNA (cmd_generate_data.py)
  role: Generates the simulated source/target count datasets and ground-truth labels
    needed to compute supervised ARI for the correlation analysis
  repo: https://github.com/nicococo/scRNA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:4721c121f94f3cc747da2b4d6f0209674ae6224be0df7e4632ca0eb3284ae382
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicococo/scRNA
  tools:
  - SC3
  - scRNA (cmd_generate_data.py)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# kta-ari-correlation-analysis

## Summary

Validates unsupervised selection of the NMF transfer mixture parameter theta by computing Kernel Target Alignment (KTA) scores per theta on SC3-clustered modified target data and correlating them with supervised ARI values against simulation ground truth.

## When to use

Use when you need to check that KTA-based unsupervised selection of the transfer mixture parameter theta coincides with the theta maximizing supervised ARI in simulated scRNA-Seq transfer learning experiments.

## When NOT to use

- Real target datasets without ground-truth labels: supervised ARI cannot be computed (the README notes supervised accuracy remains 0 when no ground truth labels are given), so this validation is restricted to simulated settings
- When the mixture parameter theta has already been fixed (e.g. by KTA alone) and only needs applying — this skill is for validating the selection itself
- Non-simulated data (e.g. Hockley/Usoskin or subsampled Tasic experiments) where no simulation ground truth exists for ARI correlation

## Inputs

- Simulated count-level scRNA-Seq datasets (10,000 genes x 1,800 cells) for complete, incomplete, and no-overlap settings
- NMF-learned source dictionary Hsrc (elastic-net regularized, initialized from one-hot-encoded source labels)
- Simplified binary target membership matrix W'trg obtained by column-wise maxima of Wtrg
- Original target expression matrix Xtrg
- Per-theta supervised ARI values against simulation ground-truth labels

## Outputs

- Per-theta KTA scores between the linear kernel of Xtrg_new and the linear kernel of SC3-predicted cell-type labels
- KTA-ARI correlation summary per overlap setting
- kta_ari_correlation.csv

## How to apply

Generate simulated count-level scRNA-Seq datasets (10,000 genes, 1,800 cells; 1,000 source cells, target downsampled from 800) for the complete, incomplete, and no-overlap settings using the scRNA package's scRNA-generate-data.sh / cmd_generate_data.py. For each candidate theta in a range within [0,1], build the modified target Xtrg_new = theta * Hsrc * W'trg + (1-theta) * Xtrg, where Hsrc is the elastic-net-regularized NMF source dictionary and W'trg is the max-simplified binary target membership matrix, then cluster each Xtrg_new with SC3. Compute the KTA score between the linear kernel of Xtrg_new over cells and the linear kernel of the SC3-predicted labels. Retrieve the supervised ARI against the known simulation ground-truth labels per theta from the reproduced comparison table, correlate KTA with ARI per overlap setting, and verify that the theta maximizing KTA coincides with the theta maximizing ARI, as reported in Supplementary Section 2.4 and Supplementary Fig. S4. Persist per-theta KTA scores and the correlation summary to kta_ari_correlation.csv.

## Related tools

- **SC3** (Clusters each modified target dataset Xtrg_new so that the kernel of the predicted cell-type labels can be compared to the kernel of Xtrg_new via KTA)
- **scRNA (cmd_generate_data.py)** (Generates the simulated source/target count datasets and ground-truth labels needed to compute supervised ARI for the correlation analysis) — https://github.com/nicococo/scRNA

## Examples

```
pip install git+https://github.com/nicococo/scRNA.git && scRNA-generate-data.sh && scRNA-target.sh --no-tsne
```

## Evaluation signals

- Per overlap setting, the KTA scores correlate positively with the supervised ARI values across the theta grid
- The theta maximizing KTA coincides with the theta maximizing ARI, matching Supplementary Section 2.4 and Supplementary Fig. S4
- kta_ari_correlation.csv contains one KTA value per theta per overlap setting plus the correlation summary
- Each Xtrg_new is a valid convex combination: theta * Hsrc * W'trg + (1-theta) * Xtrg with theta in [0,1]
- Simulated data match the specified dimensions (10,000 genes, 1,800 cells; 1,000 source cells, target downsampled from 800)

## Limitations

- Correlation validation relies on simulation ground truth; KTA-based theta selection itself is unsupervised, but its alignment with ARI can only be checked where labels exist
- The correlation may differ across the complete, incomplete, and no-overlap settings, so it must be assessed per setting
- Depending on the data, cluster range and mixture range, the target clustering step can take a long time; t-SNE plots can be disabled via --no-tsne to speed it up

## Evidence

- [methods] KTA scores define the unsupervised theta selection being validated against ARI: "θ is automatically chosen via an unsupervised assessment of the clustering quality through Kernel Target Alignment (KTA) scores"
- [methods] The modified target dataset whose kernel enters the KTA score is a convex combination parameterized by theta: [evidence span withheld — anchor status: not_found]
- [methods] Simulation scale and overlap settings used for the KTA-ARI correlation analysis: "100 sets of simulated data were generated for each of the three settings simulating the expression levels of 10 000 genes in 1800 cells"
- [readme] Supervised ARI-based accuracy requires ground-truth labels, which only simulations provide: "Command line output shows a number of results: unsupervised and supervised (if no ground truth labels are given this will remain 0.) accuracy measures."
