# ARGEvoFlow — Protein Group Definitions

**Version**: 1.0 (frozen at analysis plan lock)
**Linked to**: `analysis_plan.md` §2

---

## Reference strains (one canonical genome per ESKAPE species)

All proteins extracted from these single reference proteomes for cleanliness.

| Species | Strain | RefSeq Assembly | Rationale |
|---|---|---|---|
| *Enterococcus faecium* | DO (TX16) | GCF_000174395.2 | Standard clinical reference |
| *Staphylococcus aureus* | NCTC 8325 | GCF_000013425.1 | Genetic reference strain |
| *Klebsiella pneumoniae* | MGH 78578 | GCF_000016305.1 | First sequenced clinical KP |
| *Acinetobacter baumannii* | ATCC 17978 | GCF_000015425.1 | Widely-used lab strain |
| *Pseudomonas aeruginosa* | PAO1 | GCF_000006765.1 | Canonical reference |
| *Enterobacter cloacae* | ATCC 13047 | GCF_000025565.1 | Type strain |

---

## Group D — Diversifying non-ARGs (positive control, HIGH tolerance expected)

Surface and surface-adjacent proteins under diversifying selection from host immunity or niche variability. These are not under purifying selection and should tolerate many mutations.

| Protein | Gene | Typical length | Rationale |
|---|---|---|---|
| Pilin / type IV pilus A | *pilA* | ~150 aa | Classic antigenic variation target |
| Flagellin | *fliC* / *flaA* | ~500 aa | High surface variability |
| Outer membrane protein A | *ompA* | ~350 aa | Well-known evolvable outer membrane protein |
| Autotransporter adhesin | varies (*ata*, *uspA*) | ~600+ aa | Large diversifying surface family |

Target: 3 proteins × 6 species = ~18 (species-specific presence/absence permitting).

---

## Group A1 — Radiating β-lactamases (ARG test group with DMS anchor)

β-lactamase families with documented extensive clinical diversification. All four Ambler classes represented.

| Family | Class | Example members | Primary hosts in ESKAPE |
|---|---|---|---|
| TEM | A | TEM-1, TEM-10, TEM-52, TEM-150 | *K. pneumoniae*, *E. cloacae* |
| SHV | A | SHV-1, SHV-12, SHV-28 | *K. pneumoniae* |
| CTX-M | A | CTX-M-1, CTX-M-14, CTX-M-15, CTX-M-27 | *K. pneumoniae*, *E. cloacae* |
| KPC | A | KPC-2, KPC-3 | *K. pneumoniae* |
| OXA | D | OXA-23, OXA-48, OXA-58 | *A. baumannii*, *K. pneumoniae* |
| NDM | B | NDM-1, NDM-5, NDM-7 | *K. pneumoniae*, *A. baumannii* |
| VIM | B | VIM-1, VIM-2 | *P. aeruginosa*, *K. pneumoniae* |

Target: ~25 unique gene types after CD-HIT clustering at 90% identity.

Source: CARD canonical sequences filtered to ESKAPE genera.

---

## Group A2 — Radiating non-β-lactamase ARGs

Non-β-lactamase ARG families with documented diversification, to control for the β-lactamase bias in Group A1.

| Family | Function | Example members |
|---|---|---|
| AAC(6′) | Aminoglycoside acetyltransferase | AAC(6′)-Ib, AAC(6′)-IIa |
| APH(3′) | Aminoglycoside phosphotransferase | APH(3′)-IIa, APH(3′)-VIa |
| qnr | Quinolone resistance protein | QnrA1, QnrB1, QnrS1 |
| erm | rRNA methyltransferase | ErmA, ErmB, ErmC |
| tet(M/O) | Ribosomal protection tetracycline resistance | TetM, TetO |

Target: ~15 unique gene types after CD-HIT clustering.

---

## Group C2 — Central metabolic enzymes (MEDIUM tolerance expected)

Enzymes under moderate stabilising selection — essential for fitness but not as constrained as information-processing machinery. Provides an intermediate-tolerance tier.

| Protein | Gene | Function |
|---|---|---|
| Glyceraldehyde-3-phosphate dehydrogenase | *gapA* / *gap* | Glycolysis |
| Pyruvate kinase | *pykA* / *pykF* | Glycolysis |
| Citrate synthase | *citA* / *gltA* | TCA cycle |
| Isocitrate dehydrogenase | *icd* | TCA cycle |
| Malate dehydrogenase | *mdh* | TCA cycle |

Target: 3 proteins × 6 species = ~18.

Extraction: BLAST against *E. coli* K-12 canonical sequences, best reciprocal hit per species.

---

## Group C1 — Essential housekeeping (negative control, LOW tolerance expected)

Core information-processing machinery under strong purifying selection. These are the canonical MLST markers; strong conservation is expected.

| Protein | Gene | Function | Typical length |
|---|---|---|---|
| RNA polymerase β subunit | *rpoB* | Transcription | ~1340 aa |
| DNA gyrase B subunit | *gyrB* | DNA supercoiling | ~800 aa |
| Recombinase A | *recA* | Homologous recombination | ~350 aa |
| ATP synthase β subunit | *atpD* | Energy metabolism | ~460 aa |

Target: 4 proteins × 6 species = 24.

Note on length confounding: Group C1 has the widest length range (350–1340 aa). This is why the LMM includes `length_z` as a fixed covariate (see analysis_plan.md §6.1).

Note on drug targets: gyrB is a quinolone target and rpoB is a rifampicin target. Known resistance mutations in these genes will appear in our data. These are real biology, not contamination — but we will report tolerance at known drug-target residues separately as a sanity check.

---

## Redundancy removal

All proteins within each group are clustered with CD-HIT at 90% sequence identity. One representative per cluster is retained.

Rationale: TEM-1, TEM-2, TEM-10 differ by a few residues and would be statistical pseudo-replicates if retained separately. Clustering gives us unique gene types as statistical units.

---

## Annotation features computed per protein

| Feature | Tool | Purpose |
|---|---|---|
| Length | seqkit | LMM covariate |
| Transmembrane topology | DeepTMHMM | LMM covariate; membrane proteins handled differently by ESM2 |
| Predicted disorder | IUPred3 | Filter disordered regions from per-residue analysis |
| Pfam/InterPro domain | InterProScan | For MSA-based conservation baseline |
| AlphaFold2 structure | AlphaFold DB / ColabFold | For RSA calculation |
| Active/catalytic site residues | UniProt features, CDD | For residue-category stratification |
| Plasmid vs chromosomal encoding | CARD metadata, NCBI | Metadata only, not used in primary analysis |

---

## Exclusion criteria

A protein is excluded if:
1. Length < 50 aa or > 1500 aa (ESM2 context window and edge cases)
2. >30% predicted disordered content (IUPred3 score > 0.5)
3. Not unambiguously assignable to one of the five groups
4. AlphaFold2 structure unavailable and cannot be predicted with ColabFold within compute budget (for RSA baseline — a fallback score is used if structure is missing)

---

## Version control

This document is frozen at analysis plan lock (2026-04-22). Changes are documented in `docs/group_definitions_changelog.md` with timestamp and rationale.
