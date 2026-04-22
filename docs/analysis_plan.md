# ARGEvoFlow — Pre-Registered Analysis Plan

**Version**: 1.0
**Locked on**: 2026-04-22
**Author**: Pranavathiyani Gnanasekar, SASTRA Deemed University
**Git commit at lock**: `<to be filled after initial commit>`

---

## Purpose of this document

This file locks the hypotheses, protein groups, scoring methods, baselines, validation datasets, statistical tests, and multiple-testing corrections **before any ESM2 inference or downstream analysis is run**. Any deviations made after the lock date will be recorded in `analysis_plan_changelog.md` with justification and timestamp.

This pre-registration is the scientific contract. Its purpose is to protect against post-hoc hypothesis drift, selective reporting, and p-hacking, and to give reviewers and readers confidence that the reported results were the pre-specified tests, not cherry-picked outcomes.

---

## 1. Scope and claim

### 1.1 What ARGEvoFlow measures

Per-residue mutational tolerance of bacterial proteins, computed as the masked-marginal log-likelihood ratio (LLR) from a zero-shot ESM2-650M protein language model, aggregated to a sequence-averaged per-protein tolerance score.

### 1.2 What ARGEvoFlow does NOT measure

- Clinical resistance spread or epidemiology
- Future ARG diversification or radiation
- MIC values or antibiotic efficacy
- Selection coefficients
- Fitness in any ecologically realistic sense (Weinstein et al. 2022 non-identifiability)

### 1.3 Positioning relative to existing work

ARGEvoFlow is not methodologically novel. Zero-shot ESM2 pseudo-likelihood scoring was established by Meier et al. (NeurIPS 2021) and productionised by Brandes et al. (ESM-variants, Nature Genetics 2023) for human proteins. The contribution of this work is the disciplined, pathogen-focused, MaveDB-compatible application to the ESKAPE resistome, with explicit statistical controls that comparative bacterial genomics commonly omits.

---

## 2. Protein groups (five-arm design)

All proteins are drawn from six ESKAPE pathogens: *Enterococcus faecium*, *Staphylococcus aureus*, *Klebsiella pneumoniae*, *Acinetobacter baumannii*, *Pseudomonas aeruginosa*, *Enterobacter cloacae*. Reference strains are specified in `group_definitions.md`.

| Group | Label | Description | Target N | Expected tolerance |
|---|---|---|---|---|
| D | Diversifying non-ARGs | Surface proteins under diversifying selection: pili (pilA), flagellin (fliC), OmpA-family, autotransporters | ~18 | HIGH (positive control) |
| A1 | Radiating β-lactamases | TEM, SHV, CTX-M, KPC, OXA, NDM, VIM family members | ~25 unique types | HIGH |
| A2 | Radiating non-β-lactamase ARGs | AAC(6′), APH, qnr, erm family members | ~15 unique types | MEDIUM-HIGH |
| C2 | Central metabolic enzymes | gapA, pykA, citA, icd, mdh | ~18 (3 × 6 species) | MEDIUM |
| C1 | Essential housekeeping | rpoB, gyrB, recA, atpD | 24 (4 × 6 species) | LOW (negative control) |

**Total target: ~100 unique proteins, ~150 protein-species combinations.**

Redundancy is removed by CD-HIT clustering at 90% sequence identity within each group.

---

## 3. Scoring methods

### 3.1 Primary method
**ESM2-650M masked-marginal LLR** per residue, aggregated as mean over all 19 alternatives and all positions to give a sequence-averaged tolerance score.

Rationale for ESM2-650M: Notin (2025) scaling-wall analysis shows ESM2-650M is the efficient frontier for zero-shot variant scoring; 3B/15B do not improve performance and sometimes degrade it.

### 3.2 First-class baseline (not a sanity check)
**Conservation + RSA log-odds score.** Per-residue Shannon entropy from Pfam/InterPro MSA, weighted by predicted relative solvent accessibility from AlphaFold2 structures.

Rationale: Tsishyn et al. (*Bioinformatics* 2025) demonstrated that this two-feature parameter-free model outranks most billion-parameter PLMs on ProteinGym. Our pre-specified expectation is that PLM and conservation+RSA will be strongly correlated and may be substitutable. We report both as primary outputs.

### 3.3 Null baseline
**BLOSUM62 position-independent substitution cost.** Establishes a floor for any signal.

### 3.4 Classical baseline (restricted scope)
**dN/dS** computed with PAML codeml, applied ONLY to:
- Housekeeping proteins across deep species divergence (valid timescale)
- β-lactamase paralogs at deep family boundaries

dN/dS is explicitly NOT reported for within-family ARG diversification (CTX-M-1 vs CTX-M-15, etc.) because of the Kryazhimskiy & Plotkin (2008) and Rocha (2006) time-dependence artefact.

### 3.5 Species correction
All PLM tolerance scores are reported in two forms:
1. Raw masked-marginal LLR
2. Species-corrected (following Ding & Steinhardt 2024, Elo-style adjustment for UniRef sampling frequency)

Cross-species comparisons use species-corrected scores. Within-species rankings use raw scores.

---

## 4. DMS validation corpus

Validation is restricted to available bacterial saturation DMS datasets.

| Assay | Protein | Variants | Source | Year |
|---|---|---|---|---|
| DMS-TEM1-AMP | TEM-1 β-lactamase | ~4,997 | Stiffler et al., *Cell* | 2015 |
| DMS-TEM1-CTX | TEM-1 β-lactamase | ~4,997 | Stiffler et al., *Cell* | 2015 |
| DMS-TEM1-Firn | TEM-1 β-lactamase | ~5,200 | Firnberg et al., *MBE* | 2014 |
| DMS-TEM1-Deng | TEM-1 β-lactamase | ~1,500 | Deng et al. (Jacquier), *PNAS* | 2012/2013 |
| DMS-VIM2 | VIM-2 metallo-β-lactamase | ~39,000 | Chen et al., *eLife* | 2020 |
| DMS-NDM1-VIM2 | NDM-1 / VIM-2 family | varies | Gonzalez et al., *Nat Commun* | 2024 |

Target: 4–6 datasets, all β-lactamases. This is the honest ceiling; comprehensive DMS for KPC, CTX-M, OXA, IMP, or aminoglycoside-modifying enzymes does not exist in MaveDB as of April 2026.

**Caveat acknowledged**: Our DMS validation set is effectively β-lactamases only. Generalisation to other ARG classes is not directly validated and will be noted in all reporting.

---

## 5. Pre-registered hypotheses

### H1 — Biological gradient (primary)
**Sequence-averaged tolerance differs across the five protein groups in the expected direction:**
$$\text{tolerance}(D) \geq \text{tolerance}(A1) \geq \text{tolerance}(A2) > \text{tolerance}(C2) > \text{tolerance}(C1)$$

Test: Linear mixed-effects model (see §6.1). Pairwise contrasts via Tukey HSD.

### H2 — DMS validation (primary)
**PLM tolerance correlates with experimental DMS fitness on β-lactamases with Spearman ρ ≥ 0.35.**

Threshold rationale: ProteinGym average Spearman for top models is ~0.45–0.50 (Notin 2025); ρ ≥ 0.35 is a conservative pre-specified minimum for "the method works on this domain."

Test: Spearman ρ with 10,000-iteration BCa bootstrap 95% CI, per assay and aggregated.

### H3 — PLM beyond MSA conservation (primary)
**PLM tolerance captures information beyond conservation+RSA alone.**

Test: Partial Spearman correlation of PLM with DMS, controlling for conservation+RSA. If partial ρ > 0 with 95% BCa CI excluding zero, H3 is supported.

**Pre-registered expectation**: H3 may fail. The Tsishyn (2025) and Dutton (2024) literature suggests conservation+RSA may match or exceed PLM on small bacterial DMS panels. Failure of H3 is not a project failure — it becomes a finding about where PLMs add value for bacterial proteins.

### H4 — Housekeeping vs radiating separation (secondary)
**Mann-Whitney U test rejects the null of equal tolerance distributions between Group A1 (β-lactamases) and Group C1 (housekeeping), effect size Cliff's δ > 0.5 (large).**

### H5 — No within-housekeeping variation (robustness)
**Tolerance scores for rpoB orthologs across the 6 ESKAPE species do not differ significantly after species correction.**

This is a null check: if our species correction is working, orthologs of the same protein should give similar scores regardless of host.

---

## 6. Statistical framework

### 6.1 Primary analysis

**Linear mixed-effects model (LMM)** in R with `lme4`:

```r
fit <- lmer(
  tolerance_score ~ group + length_z + is_transmembrane +
                    (1 | species) + (1 | gene_family),
  data = proteins,
  REML = TRUE
)
```

- `group` = five-level factor (D, A1, A2, C2, C1)
- `length_z` = z-scored protein length (confounder control)
- `is_transmembrane` = binary from DeepTMHMM (confounder control)
- `(1 | species)` = random intercept for ESKAPE species (phylogenetic non-independence)
- `(1 | gene_family)` = random intercept for gene family (within-family correlation)

Contrasts: Tukey HSD for all pairwise comparisons between groups.

Effect sizes: Cliff's δ with 95% BCa bootstrap CI, computed pairwise between groups.

### 6.2 Robustness check: PGLS

**Phylogenetic Generalised Least Squares** on the housekeeping subset only (where a clean phylogeny is buildable from a concatenated MLST tree of the 6 ESKAPE species):

```r
library(ape)
library(nlme)
pgls_fit <- gls(
  tolerance_score ~ is_housekeeping,
  correlation = corBrownian(1, phylo_tree),
  data = housekeeping_data
)
```

### 6.3 Multiple testing correction

- Primary endpoints (H1, H2, H3): no correction (pre-specified single tests)
- Secondary endpoints (H4, H5, per-species breakdowns): **Benjamini-Hochberg FDR** q < 0.05 across all secondary tests
- Per-residue tests across proteins (exploratory): reported with raw p-values, explicitly labelled "exploratory," no significance claims

### 6.4 Confidence intervals

**BCa bootstrap with 10,000 iterations** for all reported correlations and effect sizes (Efron 1987 standard).

### 6.5 Null / permutation test

**1,000-iteration label-shuffle permutation test** on H1: randomise group labels, recompute the LMM F-statistic, build an empirical null distribution, compare observed F to the null.

### 6.6 Reporting metric suite for DMS validation

Following ProteinGym convention, report ALL five metrics per DMS assay:
- Spearman ρ (primary)
- AUC (binary fit/unfit separation using median fitness cutoff)
- Matthews Correlation Coefficient (MCC)
- NDCG@10% (top 10% predictive ranking quality)
- Top-recall (fraction of top-10% experimental hits in top-10% predictions)

All with 10,000-iteration BCa 95% CIs.

---

## 7. Outcome decision rules

The analysis is designed so that every outcome leads to a publishable, honest finding.

| Outcome | Interpretation | Paper claim |
|---|---|---|
| H1 ✓ + H2 ✓ + H3 ✓ | PLM adds value beyond conservation+RSA; method works on bacterial ARGs | "Zero-shot PLM tolerance captures bacterial resistome evolvability information beyond conservation; atlas useful" |
| H1 ✓ + H2 ✓ + H3 ✗ | PLM matches conservation+RSA but doesn't exceed it | "PLM is a scalable, alignment-free alternative with comparable accuracy to conservation+RSA on bacterial ARGs" |
| H1 ✓ + H2 ✗ | Group separation exists but weak DMS correlation | "Method separates biological categories but limited predictive accuracy on available β-lactamase DMS; scope restricted" |
| H1 ✗ | No group separation | "Despite theoretical motivation, zero-shot PLM tolerance does not distinguish ESKAPE ARG evolvability classes; reasons discussed" (negative result, still publishable) |

No outcome is a project failure. The scientific value is in the disciplined test, not in the sign of the result.

---

## 8. Data provenance and licensing

### 8.1 Input data sources
- **CARD canonical ARG sequences**: downloaded from card.mcmaster.ca, version pinned in `assets/card_version.txt`
- **RefSeq ESKAPE reference proteomes**: NCBI accession numbers pinned in `assets/reference_strains.tsv`
- **Pfam/InterPro MSAs**: version pinned in `assets/pfam_version.txt`
- **DMS datasets**: pulled from MaveDB and published supplementary data; checksums stored in `assets/dms_references/manifest.sha256`
- **AlphaFold2 structures**: downloaded from AFDB, version pinned per protein

### 8.2 Output licensing
- **Code**: MIT
- **Atlas data (ARGEvoAtlas)**: CC BY 4.0
- **Model weights**: not redistributed; ESM2-650M pulled from Meta AI release under MIT

### 8.3 Archival
- GitHub: version-controlled development
- Zenodo: DOI at release; all outputs archived
- MaveDB: submission at release for atlas integration with Atlas of Variant Effects Alliance

---

## 9. Compute and reproducibility

### 9.1 Environment
- **ESM2 inference**: Google Colab (T4 GPU, free tier); notebook version-controlled in `notebooks/02_colab_esm2_embeddings.ipynb`
- **Downstream analysis**: WSL2 / Ubuntu 24.04 on Windows, conda environment specified in `environment.yml`
- **R statistics**: R 4.3+ with `renv` lockfile in `r_env/renv.lock`

### 9.2 Version pinning
All dependencies pinned in:
- `environment.yml` for Python
- `r_env/renv.lock` for R
- `nextflow.config` for Nextflow and container versions

### 9.3 Reproducibility
Random seeds are set at the top of every notebook and in `argevo/__init__.py` (numpy, torch, Python `random`). Seed value: `42`. 

ESM2 inference is deterministic given fixed seeds and fixed model weights; Colab GPU results should be bit-identical across runs barring framework version changes.

---

## 10. Changes and deviations

Any deviation from this plan after the lock date must be documented in `analysis_plan_changelog.md` with:
1. What changed
2. Why it changed
3. Date and git commit of the change
4. Whether the change was made before or after seeing relevant results (pre-hoc vs post-hoc)

Post-hoc changes will be clearly flagged in any publication or poster.

---

## 11. Lock signature

This plan is locked as of 2026-04-22 by Pranavathiyani Gnanasekar.

Hash of this file at lock time: `<to be filled with sha256 of this file at initial commit>`

First commit reference: `<to be filled>`
