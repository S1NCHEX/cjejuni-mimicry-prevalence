# Project: population-scale ganglioside-mimicry potential in *Campylobacter jejuni*

Working context file. Keep this at the root of the project folder. Claude Code reads it
automatically; it is also the reference document for the human running the project.

**Owner:** Rezi Sichinava, Tbilisi State Medical University, American MD Program
**Started:** September 2026
**Status:** not yet begun — Phase 0
**Target:** preprint on bioRxiv, then journal submission

---

## 1. The question in one paragraph

*Campylobacter jejuni* triggers Guillain-Barré syndrome (GBS) through molecular mimicry:
sialylated lipooligosaccharide (LOS) on the bacterial surface resembles human gangliosides,
and the cross-reactive antibody response damages peripheral nerve. Which strains can do this
is determined by LOS locus class, the *cst-II* allele, and associated capsule type. These
features have been characterised almost entirely in strains isolated from GBS patients.
Nobody has measured how common they are in the *C. jejuni* population people are actually
exposed to. **This study measures that denominator.**

## 2. Why this is open, and why now

Jurić et al. (2026), *Frontiers in Microbiology*, DOI 10.3389/fmicb.2026.1943123, published
CjejuniTyper: an automated workflow performing Penner CPS complex prediction, LOS class
assignment, Cst-II allele calling, putative ganglioside-mimic prediction, virulence-factor
detection and AMR profiling from genome assemblies.

**Cite the published paper, not the GitHub repository** — confirmed by the author. The
article page is live; the formatted full text follows once publication processing completes.

Reported performance:

| Task | Validation set | Accuracy | Kappa | MCC |
| --- | --- | --- | --- | --- |
| LOS class | 617 assemblies | 96.4% (595/617) | 0.958 | 0.958 |
| Penner CPS | 96 assemblies | 96.9% (93/96) | 0.961 | 0.962 |
| LOS class, GBS genomes | 84 with published assignments | 98.8% concordance | — | — |

Their Bangladesh case-control arm (38 GBS, 24 enteritis controls) found HIGH or MODERATE
mimicry profiles in 34/38 cases and **10/24 controls** — 89.5% sensitivity, 58.3% specificity,
73.9% balanced accuracy. The authors state these indicate genomic mimicry-associated
potential rather than patient-level clinical prediction.

**That 58.3% specificity is the opening.** If nearly half of ordinary enteritis controls carry
a HIGH/MODERATE profile, the figure cannot be interpreted without a population baseline. Their
paper is a tool-validation study (617 + 96 + 113 genomes) and does not provide one. Twenty-four
Bangladeshi controls is not a denominator.

Two consequences:
1. The population-scale question is unoccupied.
2. The classifier is published, validated and citable, so no independent tool validation
   is required — only a reproduction check. This removes the largest risk that existed
   before the paper appeared.

Prior context: Hameed et al. 2022 (DOI 10.1371/journal.pone.0265585) typed 703 *C. jejuni*
genomes in silico, the largest such survey before this. Hayat et al. 2026
(DOI 10.1038/s41598-026-57842-2) characterised 90 GBS-associated strains across 10 countries.
Nayeem et al. 2025 (DOI 10.1128/spectrum.00062-25) did the Bangladesh case-control LOS analysis.

## 3. Scope — deliberately narrow

**Primary comparison: US retail/processing chicken vs US human clinical isolates.**

Everything else is paper two. The instinct to be comprehensive is what turns three months
into nine.

Numbers from the metadata file, after the QC prefilter in §6:

| Reservoir | US genomes | Distinct BioProjects |
| --- | --- | --- |
| Chicken | 20,549 | 49 |
| Human clinical | 13,408 | 69 |
| Bovine | 7,265 | — |
| Turkey | 441 | — |
| Swine | 127 | — |
| Unassigned | 18,879 | — |

Total US *C. jejuni* passing prefilter: 60,669 of 63,728.

**Temporal caution, important.** Chicken sampling is flat across 2019–2026 (roughly 1,700–2,600
per year). Human clinical sampling is wildly uneven: 97 in 2019, 28 in 2020, then 2,288 in 2024
and 3,271 in 2025. Any temporal comparison between reservoirs is confounded by when each
programme started sequencing. **Restrict temporal analysis to 2022–2025 where both are
reasonably sampled, or drop the temporal arm entirely from paper one.**

## 4. Hypotheses

- **H1 (primary, directional).** Sialylated LOS classes (A, B, C) and HIGH/MODERATE mimicry
  profiles are more frequent among human clinical isolates than among chicken isolates from
  the same country and period, after adjustment for lineage.
  *Rationale: if mimicry capacity also aids human infection, it should be enriched in the
  isolates that reached patients. If it is neutral, frequencies should match.*
- **H2 (secondary).** The *cst-II* Thr51/Asn51 ratio differs between reservoirs.
- **H3 (exploratory).** LOS class frequency differs across US surveillance programmes
  (BioProject), which would indicate sampling artefact rather than biology.
- **H4 (descriptive, arguably the most useful output).** Point estimate with confidence
  interval for the population prevalence of HIGH/MODERATE profiles in US chicken — the
  denominator that Jurić's specificity figure lacks.

H1 is confirmatory. H2–H4 are reported as secondary or descriptive.

## 5. The one thing that will sink this if done wrong

**Lineage confounding.** LOS class travels with clonal complex. Chicken and human isolates are
dominated by different clonal complexes. A crude proportion comparison between reservoirs will
produce a clean, significant, entirely spurious result, and every figure downstream will look
fine.

Mandatory: every reservoir comparison uses mixed-effects logistic regression with clonal
complex as a random intercept, and crude proportions are reported alongside adjusted estimates
so the difference between them is visible.

**Second risk: BioProject as a hidden confounder.** 49 chicken and 69 human BioProjects means
different labs, protocols and assemblers. If LOS class frequency varies by BioProject within
reservoir (H3), that has to be modelled too. Check this before interpreting anything.

## 6. Pipeline

### Phase 0 — setup (target: 2 weeks, ~40 h)

```bash
conda create -n camp -c conda-forge -c bioconda \
    ncbi-datasets-cli unzip blast kraken2 checkm2 mash prodigal quast \
    mlst mafft iqtree biopython pandas statsmodels -y
conda activate camp
git clone https://github.com/Gagi1993/CjejuniTyper
```

Run CjejuniTyper on its bundled example data until it completes cleanly. Read the paper's
methods alongside the code so the outputs are understood, not just produced.

### Phase 1 — reproduction check (target: 4 days, ~15 h)

Run CjejuniTyper on the 92 GBS-labelled genomes in the metadata file (`host_disease` matched
loosely — the field is spelled eight different ways). Compare LOS class calls against the
published assignments in Hayat et al. 2026 and Jurić et al. 2026.

**Gate: ≥95% concordance.** If not met, the installation or the input handling is wrong.
Do not proceed until it is.

### Phase 2 — cohort assembly (target: 2 weeks, ~30 h; mostly wall-clock)

Reservoir mapping **must combine `host` and `isolation_source`**. The `host` field is empty for
most US isolates; `isolation_source` recovers them ("raw intact chicken" n=7,017, "chicken
carcass" n=3,642, "young chicken carcass rinse (post-chill)" n=3,084, "stool" n=5,750).
Using `host` alone loses roughly two thirds of the usable data.

The mapping is a deterministic rules file, version-controlled and published as a supplementary
table so the assignment is auditable.

QC prefilter (metadata-level, confirmed after download with QUAST):

| Metric | Threshold |
| --- | --- |
| Assembly length | 1.5–2.0 Mb |
| Contig count | ≤ 200 |
| N50 | ≥ 20 kb |

**Species confirmation: Kraken2 with the MiniKraken database.** Do not trust
`scientific_name` — Jurić flagged explicitly that NCBI metadata are not uniformly consistent
or correct, and confirming species independently is the first thing he recommended.

Kraken2/MiniKraken is the method used in the CjejuniTyper paper itself. Matching it means
species calls are directly comparable to the paper whose accuracy figures this study relies
on, which matters more than any marginal gain from a different tool. Supporting options he
suggested: CheckM2 for quality and contamination assessment, and rMLST for species ID, which
runs online without a local setup and is therefore useful for hand-checking a small sample.

Practical: keep Mash as a cheap prefilter to flag obvious non-*jejuni* before running Kraken2
across the full cohort, but Kraken2 is the call of record.

Download: process-and-discard in batches of 500. Never hold the full set on disk. Keep
assemblies gzipped. Peak disk stays around 1–2 GB.

### Phase 3 — typing run (target: 2 weeks, ~20 h; mostly compute)

CjejuniTyper across the cohort, batched, resumable, logging failures. One row per genome
joined to the curated metadata. The output table is the artefact — a few MB, and the thing
that gets deposited.

### Phase 4 — analysis (target: 4 weeks, ~80 h) — **the hard part**

1. Descriptive prevalence with Wilson 95% CIs, by reservoir, overall and by year.
2. MLST → sequence type and clonal complex for every genome.
3. Mixed-effects logistic regression: outcome = HIGH/MODERATE profile (and separately,
   sialylated LOS class); fixed effect = reservoir; random intercept = clonal complex;
   covariates = collection year, BioProject where estimable.
4. Report crude and adjusted side by side.
5. Sensitivity analyses, all pre-specified:
   - one genome per Mash cluster (removes clonal expansions)
   - stricter QC (contigs ≤ 100)
   - restricted to 2022–2025
   - BioProject as additional random effect
   - indeterminate calls treated as positive, then negative, to bound estimates

**Minimum difference of interest: 2 percentage points.** With tens of thousands of genomes
everything reaches significance. Anything smaller is reported as no meaningful difference
regardless of p-value.

### Phase 5 — write-up and preprint (target: 4 weeks, ~70 h)

Preprint to bioRxiv as soon as the analysis is defensible, not when the prose is polished.
A preprint can be revised; priority cannot be reclaimed.

Deposit: code + conda environment (GitHub, MIT), per-genome call table (Zenodo, DOI),
reservoir mapping rules, accession lists with the PDG snapshot pinned.

**Total: roughly 3–4 months at 4–5 h/day.**

## 7. Delegate vs own

| Task | Claude Code | Human |
| --- | --- | --- |
| Environment setup, dependency debugging | all | — |
| Download scripts, batching, resume logic | all | check file counts match |
| Reservoir mapping code | all | **write the rules; inspect 50 assignments by hand** |
| QC and species confirmation runs | all | **set thresholds and be able to defend them** |
| Running CjejuniTyper at scale | all | — |
| Parsing outputs into tables | all | spot-check 20 rows against raw output |
| MLST / lineage assignment | all | **understand why it is the whole ballgame** |
| Reproduction check vs published calls | runs comparison | **adjudicate every discordance individually** |
| Statistical models | writes the code | **choose the model structure; interpret** |
| Sensitivity analyses | all | **decide what a changed result means** |
| Figures, tables, repo | all | check nothing is misleading |
| Every sentence of the manuscript | drafts | **every claim is yours** |

**The rule:** anything where a wrong answer looks identical to a right one stays with the
human. Everything else announces its own failure with an error message.

## 8. Limitations — write these before the results, not after

- Archive convenience sample, not a random sample. Estimates describe what has been
  sequenced. Report by BioProject as well as pooled.
- Host and source metadata are submitter-reported and unverifiable. Misclassification biases
  reservoir comparisons toward the null.
- Genotype is capacity, not expression. Phase variation and regulation intervene. No claim
  that any genome expresses ganglioside mimics.
- **No genome here is linked to a GBS case.** This measures exposure prevalence, not risk.
  Any causal phrasing about GBS is wrong and will be caught.
- Short-read assemblies mis-call homopolymer length, affecting phase-variation inference.
- Chicken and human sequencing programmes ramped up at different times (see §3).
- Single analyst, no independent check. Mitigated by pre-specification here, published code,
  and a retained manual-inspection set.

## 9. Known unknowns — resolve before Phase 4

- [x] **Ask Jurić whether he is planning the population-scale application.** Resolved
      Sept 2026. He replied "definitely go for it" and said he hopes the paper helps with
      study design and shows "where there may still be room to do something different or
      even improve on what we have done." He is not pursuing the population-scale arm.
      He has twice offered technical help with CjejuniTyper.
      Contact: dragan.juric@hzjz.hr. Use sparingly — try for an hour first, then send the
      exact command and error, not "how do I do X."
- [x] **Species confirmation method.** Resolved: Kraken2 + MiniKraken, per his own workflow.
      See §6 Phase 2.
- [ ] Read the formatted Jurić paper and supplementary material when they appear. The article
      page is live but the full text is pending publication/payment processing, so the
      supplementary tables are not yet readable. Check whether any supplementary analysis
      applies the profiles beyond the 617-genome validation set.
- [ ] Confirm exactly how HIGH/MODERATE profiles are defined so they are described correctly.
- [ ] Check whether TSMU has a compute cluster, or budget ~€25/month for a VPS.
- [ ] Check APC waiver eligibility for target journals before submission, not after acceptance.
- [ ] Find a senior co-author at TSMU — helps with revisions, credibility and possibly the APC.

## 10. Anti-stall rules

Upfront learning blocks are what kill these projects. Countermeasures:

- **Ship something visible every week.** Week 1: tool runs. Week 2: LOS table for 92 genomes.
  Week 3: 500 genomes with source labels. A project that goes dark for a month gets abandoned.
- **Five hours of shell up front, not six weeks.** `cd`, `ls`, paths, tab completion, reading
  an error. Everything else is learned when it blocks you.
- **Before running any command Claude gives you, predict what it will output.** Ten seconds.
  This is the entire difference between learning and cargo-culting.
- **Week 5 is the danger point** — when something breaks and there is no visible progress.
  Expect it.

## 11. Parallel track

An updated systematic review and meta-analysis of GBS incidence following *Campylobacter*
infection. The last pooled estimate is Keithlin et al. 2014 (DOI 10.1186/1471-2458-14-1203):
0.07%, 95% CI 0.03–0.15%, from 8 studies searching literature published before July 2011.
Esan et al. 2016 (DOI 10.1016/j.ebiom.2016.12.006) extended to 2016 but could not pool due to
>90% heterogeneity.

Fifteen years out of date, needs no command line, cannot be scooped by this field, and runs
in the gaps while downloads and compute jobs are running.

## 12. Standing alerts

- Google Scholar: `Campylobacter "LOS class" lipooligosaccharide`
- Google Scholar: `Campylobacter cstII ganglioside mimicry`
- Google Scholar: new citations of Hameed et al. 2022 (the 703-genome survey)
- Add: new citations of Jurić et al. 2026 — **highest-signal alert now.** Anyone doing the
  population-scale application will cite it.
- GitHub: watch Gagi1993/CjejuniTyper
