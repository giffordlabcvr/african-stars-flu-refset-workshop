# Step 2: QC & genotyping with Nextclade

## Option 1: Nextclade Desktop (preferred for class)

1.  Open Nextclade Desktop → choose **Influenza A H3N2** dataset.

2.  Load `sa_h3n2_2018_2025.fasta`.

3.  Run; inspect QC (frameshifts, Ns) & **Clade** column.

4.  **Export table** → save as `nextclade.tsv`.

    -   Ensure it includes `name`, `clade`, `totalMissing`, `year`, `month`, `country` (some may come from your metadata join).

## Option 2: Nextclade CLI (advanced)

```
nextclade run \
  --input-dataset <path-to-H3N2-nextclade-dataset> \
  --output-all nextclade_out/ \
  sa_h3n2_2018_2025.fasta
# Use nextclade_out/nextclade.tsv
```



* * * * *



