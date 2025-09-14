* * * * *

6) Interpretation & discussion (10 min)
---------------------------------------

-   Did the curated set avoid dominance by one month/outbreak?

-   What changed when grouping by `year` vs `year+month`?

-   Would grouping by **province** help if metadata were complete?

-   What are the risks of **over-curation** (dropping real diversity)?

-   How would you adapt rules for **H1N1**, **IBV**, or a **different country**?

* * * * *

7) Troubleshooting (quick crib)
-------------------------------

-   **Docker not found** (Windows): ensure WSL2 & Docker Desktop installed; restart Docker.

-   **Nextclade TSV columns**: adjust `--group-by` to actual header names; ensure `name` matches FASTA headers.

-   **Alignment reference missing**: supply `reference_h3n2.fasta` for the chosen segment; or use Nextclade's aligned output with `--use-existing-alignment`.

-   **Too few sequences kept**: relax to `--group-by clade year` (drop month) or increase `--sequences-per-group`.

-   **Auspice not loading**: ensure file in `auspice/` and port 4000 free; re-run `nextstrain view auspice/`.

* * * * *

8) Mini-exercise (if time or as homework)
-----------------------------------------

-   Change `--group-by` to `clade year` and set `--sequences-per-group 8`.

-   Rebuild and compare the Auspice tree.

-   Write 3 sentences: what improved, what degraded, and why.

* * * * *
