# MEEHubs 2026 — Submitted Abstract

**Submission ID**: 181
**Submitted**: April 24, 2026
**Deadline**: April 26, 2026
**Conference**: Microbial Ecology and Evolution Hub-based Conference 2026
**Portal**: app.oxfordabstracts.com/stages/78998

## Title

Do protein language models predict mutational tolerance of antibiotic resistance proteins?

## Author

Gnanasekar Pranavathiyani
SASTRA Deemed to be University, Thanjavur, India
pranavathiyani@scbt.sastra.edu

## Abstract (as submitted)

The evolution of antibiotic resistance depends on the mutational tolerance of resistance proteins, how much sequence variation they accommodate without losing activity. Deep mutational scans have quantified this for a handful of β-lactamases, but residue-resolved coverage across the ESKAPE resistome remains experimentally out of reach. This study asks whether protein language models can provide reliable tolerance scores at scale, and probes where performance degrades. Using the 650M-parameter ESM2 model, every amino-acid substitution in TEM-1 β-lactamase and AAC(6′)-Ib aminoglycoside acetyltransferase was scored via masked-marginal log-likelihood ratios. Scores were validated against all available deep mutational scans: four independent TEM-1 studies (Stiffler 2015, Firnberg 2014, Jacquier 2013, Deng 2012; 989–4,996 variants each) and one AAC(6′) study (Dandage 2018, 1,801 variants). On TEM-1, model scores recapitulated experimental fitness at Spearman ρ = 0.73–0.74 on the two highest-resolution assays, with overlapping confidence intervals. The model substantially exceeded both a classical substitution matrix (BLOSUM62; Δρ = +0.22 to +0.40) and a sequence-conservation score from 209,644 β-lactamase homologs (Δρ = +0.27 to +0.30); after controlling for conservation, partial Spearman ρ ≈ 0.71, indicating capture of position-dependent context beyond per-column sequence frequency. On the more sparsely sampled AAC(6′)-Ib, performance dropped to ρ = 0.49, consistent with taxonomic sampling biases in UniRef. These pilot results on two representative resistance proteins frame a tractable path to an ESKAPE tolerance atlas (ARGEvoFlow; github.com/pranavathiyani/ARGEvoFlow), with family-dependent accuracy bounds and full-proteome coverage pending.

## Metadata

| Field | Value |
|---|---|
| Geographical region | Virtual only |
| Presentation type | Poster |
| Topics | Selection, Pathogenicity, Genome |
| Study system | Bacteria |
| Approach/methodology | Bioinformatics |
| Study environment | Human host |
| Application | Human/animal health |

## Notes

This document records the exact text submitted for MEEHubs 2026. The submission itself cannot be edited after the April 26, 2026 deadline. This file serves as an immutable local reference.

The work supports two pre-registered hypotheses from `docs/analysis_plan.md`:
- H2 (ESM2 vs DMS Spearman ρ ≥ 0.35 with CI excluding 0)
- H3 (ESM2 captures signal beyond MSA conservation)

Headline evidence: TEM-1 Spearman ρ = 0.73–0.74 (two high-resolution DMS assays), ESM2 − conservation gap Δρ = +0.27 to +0.30 (four independent TEM-1 DMS), partial Spearman ρ ≈ 0.71 after controlling for conservation, cross-class ρ = 0.49 on AAC(6')-Ib.

Full numerical results in `results/pilot/master_results.json` and `docs/pilot_notes.md`.
