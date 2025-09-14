# Step 6: Interpretation & discussion (10 min)

-   Did the curated set avoid dominance by one month/outbreak?
-   What changed when grouping by `year` vs `year+month`?
-   Would grouping by **province** help if metadata were complete?
-   What are the risks of **over-curation** (dropping real diversity)?
-   How would you adapt rules for **H1N1**, **IBV**, or a **different country**?

* * * * *

# Step 7: Troubleshooting (quick crib)

-   **Docker not found** (Windows): ensure WSL2 & Docker Desktop installed; restart Docker.
-   **Nextclade TSV columns**: adjust `--group-by` to actual header names; ensure `name` matches FASTA headers.
-   **Alignment reference missing**: supply `reference_h3n2.fasta` for the chosen segment; or use Nextclade's aligned output with `--use-existing-alignment`.
-   **Too few sequences kept**: relax to `--group-by clade year` (drop month) or increase `--sequences-per-group`.
-   **Auspice not loading**: ensure file in `auspice/` and port 4000 free; re-run `nextstrain view auspice/`.

* * * * *


# Step 8: Mini-exercise (if time or as homework)

-   Change `--group-by` to `clade year` and set `--sequences-per-group 8`.
-   Rebuild and compare the Auspice tree.
-   Write 3 sentences: what improved, what degraded, and why.

# 9: Student handout

**Commands to run (inside Docker):**

```
# index
augur index --sequences sa_h3n2_2018_2025.fasta --output raw.idx

# subsample (edit column names as needed)
augur filter \
  --metadata nextclade.tsv \
  --sequences sa_h3n2_2018_2025.fasta \
  --sequence-index raw.idx \
  --metadata-id-columns name \
  --group-by clade year month \
  --sequences-per-group 5 \
  --seed 4242 \
  --output-sequences curated.fasta \
  --output-metadata curated.tsv

# build + view
augur align --sequences curated.fasta --reference-sequence reference_h3n2.fasta --output aligned.fasta --fill-gaps
augur tree --alignment aligned.fasta --output tree_raw.nwk --nthreads 2
augur refine --tree tree_raw.nwk --alignment aligned.fasta --metadata curated.tsv --output-tree tree.nwk --output-node-data branch_lengths.json --timetree
augur export v2 --tree tree.nwk --metadata curated.tsv --node-data branch_lengths.json --output auspice/sa-h3n2.json
nextstrain view auspice/
```

That's the full lesson: structured, time-boxed, with safety nets and exact commands. if you want, i can turn the "pipeline" into a single slide graphic you can drop into your deck.



* * * * *
