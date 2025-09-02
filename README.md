# African STARS Fellowship: Influenza Virus Reference Set Workshop
Training materials for the African STARS Fellowship — selecting influenza virus reference sets, with tutorials using Nextstrain and other tools.

2-hour workshop plan: Selecting a reference sequence set for influenza (with Nextstrain)
========================================================================================

**Audience:** university students / early-career scientists in Africa\
**Goal:** by the end, students can assemble a defensible "reference set" for influenza (seasonal & avian), wire it into a Nextstrain build, and QC with Nextclade.

* * * * *

0) Prereqs (send out beforehand)
--------------------------------

-   Install **Nextstrain CLI** (pick the OS tab and use the one-line installer). [docs.nextstrain.org+1](https://docs.nextstrain.org/en/latest/install.html?utm_source=chatgpt.com)

-   Install **Nextclade** (web is fine; CLI recommended for batches). [docs.nextstrain.org+1](https://docs.nextstrain.org/projects/nextclade/en/stable/user/nextclade-web/getting-started.html?utm_source=chatgpt.com)

-   Make sure you can view a seasonal flu build (e.g., H3N2 HA, 2--12y). [nextstrain.org+1](https://nextstrain.org/flu/seasonal/h3n2/ha/12y?utm_source=chatgpt.com)

* * * * *

1) Learning outcomes (5 min)
----------------------------

Students will be able to:

1.  Define what a "reference set" is and why it matters for **alignment, clade assignment, and downsampling**.

2.  Choose references per **lineage/segment** (H1N1pdm, H3N2, B/Victoria; plus optional H5) appropriate for African surveillance contexts.

3.  Configure **Nextstrain seasonal-flu** to use chosen references (reference FASTA, gene map GFF, clade files, exclude sites). [GitHub](https://github.com/nextstrain/seasonal-flu)

4.  Use **Nextclade** to QC candidate references & sample data; fetch official influenza datasets when appropriate. [docs.nextstrain.org](https://docs.nextstrain.org/projects/nextclade/en/stable/?utm_source=chatgpt.com)[GitHub](https://github.com/nextstrain/nextclade_data/releases?utm_source=chatgpt.com)

5.  Apply **Augur filtering/subsampling** to build balanced trees (time/region/clade). [docs.nextstrain.org+1](https://docs.nextstrain.org/en/latest/guides/bioinformatics/filtering-and-subsampling.html?utm_source=chatgpt.com)

* * * * *

2) Short conceptual primer (10 min)
-----------------------------------

-   **What counts as a "reference sequence set"?**\
    A small, curated panel used to: (i) anchor alignments & numbering (per segment), (ii) guarantee clade coverage, (iii) seed QC and annotation. In **Nextstrain seasonal-flu**, each lineage/segment expects: `reference.fasta`, `genemap.gff`, optional `exclude-sites.txt`. Non-HA segments borrow clade labels from HA. [GitHub](https://github.com/nextstrain/seasonal-flu)

-   **Scope choices:** seasonal (H1N1pdm, H3N2, B/Vic) vs avian (e.g., H5). Note B/Yamagata has not been seen widely since 2020, but extinction is debated---teach students to exclude by default while keeping a "sentinel" watch. [Nature](https://www.nature.com/articles/s41541-024-01010-y?utm_source=chatgpt.com)[ScienceDirect](https://www.sciencedirect.com/science/article/pii/S0264410X24011320?utm_source=chatgpt.com)

* * * * *

3) Data sources & Africa context (10 min)
-----------------------------------------

-   **Open access sequence data:** **NCBI Virus** (search, filter, download; provides GFF/GBFF for many RefSeqs). [NCBI](https://www.ncbi.nlm.nih.gov/labs/virus/vssi/docs/help/?utm_source=chatgpt.com)

-   **Surveillance context & metadata:** **WHO FluNet** for country-level time trends (helps decide what clades/years to cover); Africa CDC IPG/Africa PGI for regional genomics context. [World Health Organization+1](https://www.who.int/tools/flunet?utm_source=chatgpt.com)[africacdc.org](https://africacdc.org/institutes/ipg/?utm_source=chatgpt.com)

-   **Nextstrain reference visualizations:** explore current diversity in H3N2/H1N1pdm/B(Vic) and avian-flu H5 to inform coverage. [nextstrain.org+2nextstrain.org+2](https://nextstrain.org/flu/seasonal/h3n2/ha/12y?utm_source=chatgpt.com)\
    *(Note: GISAID access may be required for some community builds; respect data-use terms. The exercises below use open data.)*

* * * * *

4) How to pick a solid reference set (20 min)
---------------------------------------------

**A. Decide scope**

-   Seasonal: H1N1pdm (HA/NA ± internal segments), H3N2, B/Victoria. Consider time window(s) you plan to analyze (e.g., last 2--3 years). Use FluNet to check which subtypes circulate in your country/region. [World Health Organization](https://www.who.int/tools/flunet/flunet-summary?utm_source=chatgpt.com)

-   Avian option: H5 HA (clade 2.3.4.4b predominates globally; verify relevance to your question). Use Nextstrain H5 pages to see geography/clades. [nextstrain.org](https://nextstrain.org/avian-flu/h5n1/ha/2y?utm_source=chatgpt.com)

**B. Criteria for each reference sequence (per segment)**

-   Complete ORF with correct annotation; **no gaps in reference FASTA** (Nextclade rejects gapped references). [docs.nextstrain.org](https://docs.nextstrain.org/projects/nextclade/en/3.13.2/changes/CHANGELOG.html?utm_source=chatgpt.com)

-   Representative of **target clade(s)**; include **vaccine-like** or widely used anchor strains where helpful.

-   Clean metadata (collection date, location, host).

-   Availability of **gene map (GFF)**; if none, derive from GenBank/RefSeq records provided by NCBI Virus. [NCBI](https://www.ncbi.nlm.nih.gov/labs/virus/vssi/docs/help/?utm_source=chatgpt.com)

**C. Ensure clade coverage**

-   For the time window & region, list **expected clades** (e.g., H3N2 J.* subclades; H1N1pdm D.*; B/Vic C.*---see Nextclade dataset release notes for current subclade systems). This helps you decide the minimal set of anchors. [GitHub](https://github.com/nextstrain/nextclade_data/releases?utm_source=chatgpt.com)

**D. Segment strategy**

-   For **seasonal builds**, HA and NA are primary; internal segments optional. The seasonal-flu workflow expects per-segment references and explicitly notes HA-driven clade annotation for other segments. [GitHub](https://github.com/nextstrain/seasonal-flu)

* * * * *

5) Hands-on Lab (60 min total)
------------------------------

### Part 1 --- Gather candidates (15 min)

1.  **Pick a target** (e.g., H3N2 HA, Africa, 2023-2025).

2.  From **NCBI Virus**, query for segment, host=human (or avian for H5), country/region, and date; download FASTA + a few complete annotated references (GBFF/GFF). [NCBI](https://www.ncbi.nlm.nih.gov/labs/virus/vssi/docs/help/?utm_source=chatgpt.com)

3.  Open **Nextclade Web** → choose the influenza dataset matching your target (e.g., H3N2 HA) → drag in 50--200 recent sequences to see QC & clade assignment; keep a note of dominant clades in your region. [docs.nextstrain.org](https://docs.nextstrain.org/projects/nextclade/en/stable/user/nextclade-web/getting-started.html?utm_source=chatgpt.com)

> **Tip:** You can also pull datasets for CLI work:

```
nextclade dataset list | grep flu
nextclade dataset get --name 'nextstrain/flu/h3n2/ha/EPI1857216' --output-dir datasets/flu_h3n2_ha

```
*(dataset names evolve; check the release notes/listing first)* [docs.nextstrain.org](https://docs.nextstrain.org/projects/nextclade/en/stable/user/nextclade-cli/reference.html?utm_source=chatgpt.com)[GitHub]  
(https://github.com/nextstrain/nextclade_data/releases?utm_source=chatgpt.com)

### Part 2 --- Pick & check references (15 min)

1.  Choose 1--2 **reference anchors per lineage/segment** that: (i) reflect circulating clades you observed, (ii) have clean annotation.

2.  Confirm **no "-" gaps** in your chosen reference FASTA (or strip them). [docs.nextstrain.org](https://docs.nextstrain.org/projects/nextclade/en/3.13.2/changes/CHANGELOG.html?utm_source=chatgpt.com)

3.  Export/prepare matching **`genemap.gff`** (from GBFF/GFF via NCBI Virus). [NCBI](https://www.ncbi.nlm.nih.gov/labs/virus/vssi/docs/help/?utm_source=chatgpt.com)

### Part 3 --- Wire into Nextstrain seasonal-flu (20 min)

1.  Clone the repo and run the included example once:

```
git clone https://github.com/nextstrain/seasonal-flu
cd seasonal-flu
nextstrain build . --configfile profiles/example/builds.yaml
nextstrain view auspice/
```

1.  Drop your reference assets into the expected paths:

    -   `config/{lineage}/{segment}/reference.fasta`

    -   `config/{lineage}/{segment}/genemap.gff`

    -   (optional) `config/{lineage}/{segment}/exclude-sites.txt`\
        *(add or replace per your scope)* [GitHub](https://github.com/nextstrain/seasonal-flu)

2.  Point your **build config** (`profiles/your_build/builds.yaml`) at your lineages/segments and set a subsampling policy (see next part).

### Part 4 --- Subsample fairly (10 min)

Use **Augur filter** to avoid bias:

```
subsamples:
  global:
    # these strings are passed to `augur filter`
    filters: "--min-date 2023-01-01 --max-date 2025-09-01 --min-length 1600 --exclude-ambiguous-dates-by any"
    group-by: "year month region"
    max-sequences: 600
```

Key ideas: **grouped sampling**, **probabilistic sampling**, and **caps per group** to keep representation across time & regions (important for under-sampled areas). [docs.nextstrain.org+1](https://docs.nextstrain.org/en/latest/guides/bioinformatics/filtering-and-subsampling.html?utm_source=chatgpt.com)

* * * * *

6) Quality control & iteration (10 min)
---------------------------------------

-   After a build, inspect trees in **Auspice** and compare to **Nextstrain seasonal flu** public builds for plausibility. [nextstrain.org](https://nextstrain.org/flu/seasonal/h3n2/ha/12y?utm_source=chatgpt.com)

-   If clades are missing or local diversity is under-represented, widen the time window, adjust groupings (e.g., `group-by: "country month"`), or raise `max-sequences`. [docs.nextstrain.org](https://docs.nextstrain.org/en/latest/guides/bioinformatics/filtering-and-subsampling.html?utm_source=chatgpt.com)

-   For **non-HA segments**, remember clade labels derive from HA---ensure your HA reference/clade file is correct. [GitHub](https://github.com/nextstrain/seasonal-flu)

* * * * *

7) Avian influenza (H5) option block (10--15 min, optional)
----------------------------------------------------------

-   Show **Nextstrain avian-flu H5N1** to motivate region/host coverage. [nextstrain.org](https://nextstrain.org/avian-flu/h5n1/ha/2y?utm_source=chatgpt.com)

-   Explain **Nextclade H5 datasets**: they pair **reference phylogenies** with **HA references + gene maps** to enable clade assignment & mutation calls---useful when building your own datasets. [PubMed](https://pubmed.ncbi.nlm.nih.gov/39829835/?utm_source=chatgpt.com)[BioRxiv](https://www.biorxiv.org/content/10.1101/2025.01.07.631789.full?utm_source=chatgpt.com)

* * * * *

8) "What good looks like" checklist (5 min)
-------------------------------------------

A strong reference set...

-   Covers **all circulating lineages/clades** for your scope & region/time.

-   Uses **ungapped** reference FASTAs with **correct gene maps**. [docs.nextstrain.org](https://docs.nextstrain.org/projects/nextclade/en/3.13.2/changes/CHANGELOG.html?utm_source=chatgpt.com)[NCBI](https://www.ncbi.nlm.nih.gov/labs/virus/vssi/docs/help/?utm_source=chatgpt.com)

-   Is **small** (1--3 per lineage/segment) but **sufficiently representative**.

-   Produces builds whose diversity patterns **match surveillance** (sanity-check via FluNet trend signals). [World Health Organization](https://www.who.int/tools/flunet/flunet-summary?utm_source=chatgpt.com)

* * * * *

9) Common pitfalls (5 min)
--------------------------

-   Using B/Yamagata by default (keep a sentinel view, but don't assume circulation). [Nature](https://www.nature.com/articles/s41541-024-01010-y?utm_source=chatgpt.com)[ScienceDirect](https://www.sciencedirect.com/science/article/pii/S0264410X24011320?utm_source=chatgpt.com)

-   Forgetting to supply `genemap.gff` or using gapped references (Nextclade will fail). [GitHub](https://github.com/nextstrain/seasonal-flu)[docs.nextstrain.org](https://docs.nextstrain.org/projects/nextclade/en/3.13.2/changes/CHANGELOG.html?utm_source=chatgpt.com)

-   Over-subsampling well-sampled countries while **under-representing** African sites---tune `group-by` and caps. [docs.nextstrain.org](https://docs.nextstrain.org/en/latest/guides/bioinformatics/filtering-and-subsampling.html?utm_source=chatgpt.com)

* * * * *

10) Mini-project (take-home or group work, 10 min intro)
--------------------------------------------------------

-   **Brief:** Build an **Africa-focused H3N2 HA** tree (2023--2025).

-   **Deliverables:**

    1.  your **reference files** (FASTA + GFF) & justification (which clades/why),

    2.  a `builds.yaml` with subsampling tuned for African regions,

    3.  a 1-slide "lessons learned" comparing your tree to FluNet trends. [World Health Organization](https://www.who.int/tools/flunet/flunet-summary?utm_source=chatgpt.com)

* * * * *

Slide/handout outline (you can copy straight into slides)
---------------------------------------------------------

1.  **Why references?** (alignment, clades, reproducibility)

2.  **Seasonal flu basics** (lineages/segments; what to include; B/Yam caveat). [Nature](https://www.nature.com/articles/s41541-024-01010-y?utm_source=chatgpt.com)[ScienceDirect](https://www.sciencedirect.com/science/article/pii/S0264410X24011320?utm_source=chatgpt.com)

3.  **Where to get data** (NCBI Virus; FluNet for trends; Africa CDC IPG). [NCBI](https://www.ncbi.nlm.nih.gov/labs/virus/vssi/docs/help/?utm_source=chatgpt.com)[World Health Organization](https://www.who.int/tools/flunet?utm_source=chatgpt.com)[africacdc.org](https://africacdc.org/institutes/ipg/?utm_source=chatgpt.com)

4.  **Nextstrain seasonal-flu anatomy** (refs, GFF, clade TSVs; example repo). [GitHub](https://github.com/nextstrain/seasonal-flu)

5.  **Nextclade for QC & clades** (web + CLI; dataset names). [docs.nextstrain.org+1](https://docs.nextstrain.org/projects/nextclade/en/stable/user/nextclade-web/getting-started.html?utm_source=chatgpt.com)

6.  **Subsampling patterns** (grouped, probabilistic, caps; YAML examples). [docs.nextstrain.org+1](https://docs.nextstrain.org/en/latest/guides/bioinformatics/filtering-and-subsampling.html?utm_source=chatgpt.com)

7.  **Case study:** H3N2 Africa 2023--2025 (walkthrough)

8.  **Option:** H5 block (datasets & clade assignment). [PubMed](https://pubmed.ncbi.nlm.nih.gov/39829835/?utm_source=chatgpt.com)

9.  **Pitfalls & checklist**

10. **Mini-project brief**

* * * * *

Copy-paste snippets for the lab
-------------------------------

**Create a minimal build profile** (`profiles/africa/builds.yaml`):

```
builds:
  h3n2_ha_africa_2023_2025:
    lineage: h3n2
    segment: ha
    subsamples:
      global:
        filters: "--min-date 2023-01-01 --max-date 2025-09-01 --min-length 1600 --exclude-ambiguous-dates-by any"
        group-by: "country month"
        max-sequences: 600
```

Run:

```
nextstrain build . --configfile profiles/africa/builds.yaml --forceall
nextstrain view auspice/
```





