# Hands-on Lab Practical (60 min total)

### Part 1 --- Gather candidates (15 min)

1.  **Pick a target** (e.g., H3N2 HA, Africa, 2023-2025).

2.  From **NCBI Virus**, query for segment, host=human (or avian for H5), country/region, and date; download FASTA + a few complete annotated references (GBFF/GFF). [NCBI](https://www.ncbi.nlm.nih.gov/labs/virus/vssi/docs/help/?utm_source=chatgpt.com)

3.  Open **Nextclade Web** → choose the influenza dataset matching your target (e.g., H3N2 HA) → drag in 50--200 recent sequences to see QC & clade assignment; keep a note of dominant clades in your region. [docs.nextstrain.org](https://docs.nextstrain.org/projects/nextclade/en/stable/user/nextclade-web/getting-started.html)

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

