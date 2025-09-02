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





