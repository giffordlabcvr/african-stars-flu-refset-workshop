# Step 3 — Curate with augur filter (35 min, hands-on)

## 3.1 Open a Nextstrain shell (Docker)

From the directory containing your files:

```
docker run -it --rm -v "$PWD":/data -w /data nextstrain/base bash
```

## 3.2 Index sequences (speeds up filtering)

```
augur index \
  --sequences tutorial_data/raw/sa_h3n2_2018_2025.fasta \
  --output raw.idx
```

## 3.3 Rule-based subsampling (clade × year × month)

Start with 5 per group to land ~150–200 sequences:

```
augur filter \
  --metadata tutorial_data/prepared/nextclade.tsv \
  --sequences tutorial_data/raw/sa_h3n2_2018_2025.fasta \
  --sequence-index raw.idx \
  --metadata-id-columns name \
  --group-by clade year month \
  --sequences-per-group 5 \
  --seed 4242 \
  --output-sequences curated.fasta \
  --output-metadata curated.tsv
```

**Tune size quickly**

-   Too big? add `--subsample-max-sequences 160`
-   Too small? increase `--sequences-per-group` or drop `month` from `--group-by`.
  
## 3.4 (Optional) Add a few global anchors

Create prepared/anchors.txt with ~10–20 global reference IDs and include:

```
--include tutorial_data/prepared/anchors.txt
```

**What students should notice**

-   CLI reports drops/kept (e.g., "741 dropped... 149 kept").
-   `curated.tsv` mirrors the distribution across clades/years/months.


* * * * *
