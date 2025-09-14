# Step 6: Troubleshooting (quick crib)

-   **Docker not found** (Windows): ensure WSL2 & Docker Desktop installed; restart Docker.
-   **Nextclade TSV columns**: adjust `--group-by` to actual header names; ensure `name` matches FASTA headers.
-   **Alignment reference missing**: supply `reference_h3n2.fasta` for the chosen segment; or use Nextclade's aligned output with `--use-existing-alignment`.
-   **Too few sequences kept**: relax to `--group-by clade year` (drop month) or increase `--sequences-per-group`.
-   **Auspice not loading**: ensure file in `auspice/` and port 4000 free; re-run `nextstrain view auspice/`.

* * * * *
