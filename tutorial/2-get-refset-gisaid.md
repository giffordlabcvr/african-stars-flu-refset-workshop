## 2. GISAID-based CLI approach

<img src="../images/gisaid.jpg" align="right" alt="" width="180"/>

### Download from GISAID (recommended filters)

1.  Open GISAID's **[EpiFlu](https://www.epicov.org/epi3/)** interface.

2.  Use the **Query Builder** (example):

    -   Virus/Subtype: **A/H3N2**
    -   Location: **South Africa**
    -   Collection date: **2018--2025**
    -   (Add any other constraints you need for your class/demo.)

3.  **Export sequences** as **FASTA**

    -   **Header template:** ensure the **first token is the GISAID ID** (`EPI_ISL_...`).
    -   It's fine to include extra fields after that, separated by `|`.
    -   Save as **`gisaid_ha.fasta`** in your working directory.

4.  **Export metadata** as **TSV/CSV**

    -   Choose **tab-delimited (TSV)**.
    -   Make sure the table includes at least: `Isolate_Id`, `Location`, `Collection_Date`, and (if available) `Clade`, `Host`, `Isolate_Name`.
    -   Save as **`gisaid_meta.tsv`**.

> Tip: Don't overthink column selection---our cleaning step later safely parses TSV and derives the fields Augur/Auspice need.

* * * * *

## Quick pre-flight checks (on your downloads)

Run these from your working directory (macOS/Linux):

### 1) Count sequences

`grep -c '^>' gisaid_ha.fasta`

### 2) Spot-check a few headers – the first token should be EPI_ISL_...

`grep '^>' gisaid_ha.fasta | head -5`

### 2) Confirm a '|' is present (not strictly required, but common)

`grep '^>' gisaid_ha.fasta | grep -c '|'`


## Next Steps

In the next steps we will:

1.  **Normalize** FASTA headers to the **left of the first `|`** and **de-duplicate** IDs;
2.  Run **Nextclade** (Docker) for QC + clades;
3.  Clean/merge **metadata**, then **downsample, align, and tree** with **Augur**;
4.  Export to **Auspice**;

Each step includes a brief explanation and a **checkpoint** so students can verify they're on track.

* * * * *

### 2.0. (One-time) Pull required images

```
docker pull nextstrain/nextclade:latest
docker pull nextstrain/base
```

* * * * *

### 2.1.  Normalize FASTA headers 

**a)** Normalize headers to the token before the first '|'.

Keep only the **ID** (left of the first `|`), then drop any duplicate IDs (keep first).

```
awk 'BEGIN{OFS=""}
  /^>/ { h=$0; sub(/^>/,"",h); split(h,a,"|"); id=a[1]; sub(/[[:space:]]+$/,"",id); print ">", id; next }
  { print }
' gisaid_ha.fasta > gisaid_ha.acc.fasta
```

**b)** De-duplicate by normalized ID

```
awk '/^>/{k=$0; keep=!seen[k]++} { if(keep) print }' \
  gisaid_ha.acc.fasta > gisaid_ha.acc.uniq.fasta
```

✅ **Checkpoint (should print nothing):**

```
grep '^>' gisaid_ha.acc.uniq.fasta | sort | uniq -d | sed -n '1,10p'
```

* * * * *

### 2.2. Nextclade QC + clades

<img src="../images/nextstrain.png" align="right" alt="" width="100"/> 

**What is Nextclade doing here?**

Nextclade compares each sequence to a **curated dataset** (reference, gene coordinates, clade rules, QC thresholds) and returns:

-   **Clade calls** (e.g., Nextstrain H3N2 clades),
-   **QC metrics** (overall pass/warn/fail + reasons),
-   Optional **aligned sequences** (if you ask for them).

We'll use the **H3N2 / HA** dataset so clade/QC logic matches your input.

**Why "dataset get"?**

This **downloads & caches** the pathogen-/gene-specific bundle under `~/.nextclade/...` so runs are fast and reproducible. We mount your host `~/.nextclade` into the container so the cache persists across runs.

**a)** Download the H3N2-HA dataset (cached under `~/.nextclade`)

```
docker run --rm -it -v "$HOME/.nextclade":/root/.nextclade nextstrain/nextclade:latest \
  nextclade dataset get --name nextstrain/flu/h3n2/ha \
  --output-dir /root/.nextclade/datasets/h3n2_ha
```

**b)** Run Nextclade on your normalized + deduped FASTA

```
docker run --rm -it \
  -v "$PWD":/data \
  -v "$HOME/.nextclade":/root/.nextclade \
  nextstrain/nextclade:latest \
  nextclade run \
    --input-dataset /root/.nextclade/datasets/h3n2_ha \
    --output-tsv /data/nextclade.tsv \
    /data/gisaid_ha.acc.uniq.fasta
```

✅ **Checkpoint**

-   `ls -lh nextclade.tsv` (file exists)
-   `wc -l nextclade.tsv` ≈ **#sequences + 1** (header)
-   Quick peek: `cut -f1,2,3,8 nextclade.tsv | head`\
    (you should see columns like `seqName`, `clade`, `qc.overallStatus`)

**What's in `nextclade.tsv` that we'll use later?**

-   `clade` → becomes `nextclade_clade` in our merged metadata.
-   `qc.overallStatus` → simple pass/warn/fail summary you can use to filter.
-   (There are many other QC fields; we keep it light for the tutorial.)

* * * * *

### 2.3. Clean GISAID metadata

<img src="../images/python.png" align="right" alt="" width="100"/>

**What & why?**\
GISAID's TSV often contains **quoted fields with commas and line breaks** (e.g., in *Publication*). Shell tools like `cut` / `awk` can choke on these. We therefore use **Python's CSV reader** (which correctly handles quoting and embedded newlines) to:

-   **Keep** core fields we need downstream (`Isolate_Id`, `Isolate_Name`, `Subtype`, `Clade`, `Host`, `Location`, `Collection_Date`).

-   **Split** the single `Location` string into **`region / country / division / location`** (Nextstrain expects this hierarchy).

-   **Normalize** dates from `Collection_Date` into **`date / year / month`**:

    -   accepts `YYYY`, `YYYY-MM`, or `YYYY-MM-DD`

    -   fills missing parts with `01` so Augur can time-scale (e.g., `2023` → `2023-01-01`)

-   Write a tidy **`meta_clean.tsv`**, keyed by `Isolate_Id`.\
    (We'll harmonize South African province names later in **3.5b** so maps render cleanly.)

**Why Python *in Docker*?**\
You don't need to install anything locally. The `nextstrain/base` image already has Python, so the command below runs a one-off Python script **inside the container**, reading/writing files from your current folder (mounted at `/data`).

```
docker run --rm -i -v "$PWD":/data -w /data nextstrain/base \
  python3 - <<'PY'
import csv, re
from datetime import datetime

inp  = "gisaid_meta.tsv"
outp = "meta_clean.tsv"

def split_loc(s):
    parts = [p.strip() for p in (s or "").split('/') if p.strip()]
    parts += [""] * (4 - len(parts))
    return parts[:4]  # region, country, division, location

def norm_date(s):
    if not s: return ("","","")
    s = s.strip()
    for fmt in ("%Y-%m-%d","%Y-%m","%Y"):
        try:
            dt = datetime.strptime(s, fmt)
            y = dt.year; m = dt.month if "%m" in fmt else 1
            return (dt.strftime("%Y-%m-%d") if "%d" in fmt else f"{y:04d}-{m:02d}-01",
                    str(y), f"{m:02d}")
        except ValueError: pass
    m = re.search(r"(\d{4})", s)
    if m: return (f"{m.group(1)}-01-01", m.group(1), "01")
    return ("","","")

with open(inp, newline='') as f, open(outp, "w", newline='') as g:
    r = csv.DictReader(f, delimiter="\t")
    cols_out = ["Isolate_Id","Isolate_Name","Subtype","Clade","Host",
                "region","country","division","location","Collection_Date",
                "date","year","month"]
    w = csv.DictWriter(g, delimiter="\t", fieldnames=cols_out); w.writeheader()
    for row in r:
        reg, ctry, div, loc = split_loc(row.get("Location",""))
        d, y, m = norm_date(row.get("Collection_Date",""))
        w.writerow({
          "Isolate_Id": row.get("Isolate_Id",""),
          "Isolate_Name": row.get("Isolate_Name",""),
          "Subtype": row.get("Subtype",""),
          "Clade": row.get("Clade",""),
          "Host": row.get("Host",""),
          "region": reg, "country": ctry, "division": div, "location": loc,
          "Collection_Date": row.get("Collection_Date",""),
          "date": d, "year": y, "month": m
        })
print(f"Wrote {outp}")
PY
```

✅ **Checkpoint**

-   `ls -lh meta_clean.tsv` (file exists)
-   Rows roughly match your sequence count:

```
echo "FASTA seqs:" $(grep -c '^>' gisaid_ha.acc.uniq.fasta)
echo "metadata rows:" $(tail -n +2 meta_clean.tsv | wc -l)
```

-   Quick peek at columns:
  
```
head -1 meta_clean.tsv | tr '\t' '\n' | nl
head -3 meta_clean.tsv
```

**Notes & gotchas**

-   Leave empty values as empty strings; Augur/Auspice tolerate missing fields.
-   Don't "fix" the TSV in Excel/Numbers (they may change tabs/quotes). If you must open it, re-export as **tab-delimited UTF-8**.
-   `division` is the **province/state** level for mapping. We'll normalize South African province strings in **3.5b** for clean map layers


* * * * *


### 2.4 Merge metadata + Nextclade

<img src="../images/python.png" align="right" alt="" width="100"/>

**What this step does (and why):**

-   **Joins** your cleaned GISAID metadata (`meta_clean.tsv`) with **Nextclade** results (`nextclade.tsv`).
-   Creates a **`strain`** column (required by Augur/Auspice) that **must exactly match** the sequence names in your FASTA.\
    In our workflow we normalized headers to the GISAID ID, so we set: **`strain = Isolate_Id`** (e.g., `EPI_ISL_1234567`).
-   Copies key Nextclade outputs:
    -   `clade` → saved as **`nextclade_clade`**
    -   `qc.overallStatus` → overall QC (`pass`/`warn`/`fail`)
-   Writes **`metadata_final.tsv`**, which Augur uses for downsampling/refine and Auspice uses for coloring & metadata display.

**Join key (important):**

-   Nextclade's `seqName` mirrors your FASTA header. We **split at the first `|`** and use the **left token** (the GISAID ID).\
    This is robust whether your headers had a pipe or were already just `EPI_ISL_...`.

**Why Python *in Docker*?**\
Reliable TSV parsing (quotes, embedded newlines) with **no local installs**. The `nextstrain/base` image has Python preinstalled; we mount your current folder into `/data` so files read/write locally.

```
docker run --rm -i -v "$PWD":/data -w /data nextstrain/base \
  python3 - <<'PY'
import csv

meta_in  = "meta_clean.tsv"
clade_in = "nextclade.tsv"
outp     = "metadata_final.tsv"
missed_log = "nextclade_unmatched_ids.txt"

clade_map = {}
with open(clade_in, newline='') as f:
    r = csv.DictReader(f, delimiter="\t")
    for row in r:
        acc = row['seqName'].split('|')[0]  # safe even if no '|'
        clade_map[acc] = {
          'nextclade_clade': row.get('clade',''),
          'qc.overallStatus': row.get('qc.overallStatus','')
        }

missed = set()
with open(meta_in, newline='') as f, open(outp, "w", newline='') as g:
    r = csv.DictReader(f, delimiter="\t")
    cols_out = ["strain","Isolate_Id","Isolate_Name","Subtype","Clade","nextclade_clade",
                "Host","region","country","division","location","Collection_Date",
                "date","year","month","qc.overallStatus"]
    w = csv.DictWriter(g, delimiter="\t", fieldnames=cols_out); w.writeheader()
    for row in r:
        iso = row["Isolate_Id"]
        nc  = clade_map.get(iso)
        if not nc: missed.add(iso); nc = {'nextclade_clade':'','qc.overallStatus':''}
        w.writerow({
          "strain": iso, "Isolate_Id": iso,
          "Isolate_Name": row["Isolate_Name"], "Subtype": row["Subtype"],
          "Clade": row["Clade"], "nextclade_clade": nc['nextclade_clade'],
          "Host": row["Host"], "region": row["region"], "country": row["country"],
          "division": row["division"], "location": row["location"],
          "Collection_Date": row["Collection_Date"], "date": row["date"],
          "year": row["year"], "month": row["month"],
          "qc.overallStatus": nc["qc.overallStatus"]
        })

with open(missed_log,"w") as h:
    for m in sorted(missed): h.write(m+"\n")

print(f"Wrote {outp}. Unmatched in Nextclade: {len(missed)} (see {missed_log})")
PY
```

✅ **Checkpoints**

-   **File exists:** `ls -lh metadata_final.tsv`
-   **Row counts look sane:** should be similar to `meta_clean.tsv`

```
echo "meta_clean rows:" $(tail -n +2 meta_clean.tsv | wc -l)
echo "metadata_final rows:" $(tail -n +2 metadata_final.tsv | wc -l)
```
-   **Columns & order (strain first):**
```
head -1 metadata_final.tsv | tr '\t' '\n' | nl
head -3 metadata_final.tsv
```
-   **`Unmatched IDs (ideally 0):**

```
wc -l nextclade_unmatched_ids.txt
sed -n '1,10p' nextclade_unmatched_ids.txt
```

**Common causes of "unmatched" (and quick fixes):**

-   Ran Nextclade on a **different FASTA** than the normalized + deduped one.\
    → Re-run Nextclade on `gisaid_ha.acc.uniq.fasta`.
-   **Whitespace** or invisible characters in IDs.\
    → We call `.strip()`; also re-check your normalization step (1.3.1).
-   Sequences filtered out **before** Nextclade.\
    → Either ignore (fields will be blank) or re-run Nextclade on the current set.

**Optional tweaks (if you want pre-curation):**

-   You can later filter by `qc.overallStatus` in `augur filter`, or add a simple condition in this script to exclude `fail` sequences before writing `metadata_final.tsv`.

  
* * * * *

### 2.5 Augur: downsample, align, tree, refine

**Goal:** take your full dataset and produce a tidy, time-scaled tree that's fast to view in Auspice.\
**Strategy:** Downsample to keep the tree readable (e.g., **≤10 sequences per clade per year** --- tune this to taste), then align, infer a tree, and refine it in time using collection dates.

---

#### a) Index (speed up downstream filtering)

`augur index` makes a lightweight index over your FASTA so `augur filter` runs faster and uses less memory.

```
docker run -it --rm -v "$PWD":/data -w /data nextstrain/base \
  augur index \
    --sequences gisaid_ha.acc.uniq.fasta \
    --output raw.idx
```

✅ **Checkpoint:** `raw.idx` exists (small file).

**Why this matters:** For larger datasets the index prevents repeated FASTA scans.

---

#### b) Filter (downsample to a manageable, representative set)

We select a subset stratified by **clade** and **year** (10 per stratum).\
The join key is `strain` (first column of your metadata) which must match the **FASTA headers** (we normalized these earlier).


```
docker run -it --rm -v "$PWD":/data -w /data nextstrain/base \
  augur filter \
    --metadata metadata_final.tsv \
    --sequences gisaid_ha.acc.uniq.fasta \
    --sequence-index raw.idx \
    --metadata-id-columns strain \
    --group-by nextclade_clade year \
    --sequences-per-group 10 \
    --output-sequences curated.fasta \
    --output-metadata curated.tsv
```

✅ **Checkpoints**

-   Files exist: `curated.fasta`, `curated.tsv`
-   Count looks sane:
```
echo "Curated seqs:" $(grep -c '^>' curated.fasta)
echo "Curated rows:" $(tail -n +2 curated.tsv | wc -l)
```

-   If the count is **0 or very small**, you may have:
    -   Very sparse clade/year combinations → reduce grouping (e.g., `--group-by year`)
    -   Strict QC you want to apply → add `--query "qc.overallStatus in ['pass','warn']"`\
        (or run QC filtering earlier when building metadata)

**Notes**

-   Reproducibility: some Augur versions support `--subsample-seed` (or `--seed`) on `filter`. If your `nextstrain/base` lacks it, switch to `nextstrain/cli:latest`.
-   You can group by other fields (e.g., `country`), but ensure those columns exist in `metadata_final.tsv`.

---


#### c) Normalise South African province names (recommended for maps)

Auspice expects canonical province names. This harmonises common variants (e.g., *Guateng*, *Province of ...*, different casings) so the **division** map layer renders.

Run this on the **downsampled** metadata (`curated.tsv`) to produce `curated_geo.tsv`:

```
docker run --rm -i -v "$PWD":/data -w /data nextstrain/base \
  python3 - <<'PY'
import csv, re

infile  = "curated.tsv"      # or "metadata_final.tsv" if fixing earlier
outfile = "curated_geo.tsv"

canon = {
  "eastern cape":"Eastern Cape",
  "free state":"Free State",
  "gauteng":"Gauteng",
  "guateng":"Gauteng",
  "guanteng":"Gauteng",
  "?gauteng":"Gauteng",
  "gauteng province":"Gauteng",
  "kwazulu natal":"KwaZulu-Natal",
  "kwazulu-natal":"KwaZulu-Natal",
  "kwa-zulu natal":"KwaZulu-Natal",
  "province of kwazulu-natal":"KwaZulu-Natal",
  "kzn":"KwaZulu-Natal",
  "limpopo":"Limpopo",
  "mpumalanga":"Mpumalanga",
  "mpumalanga province":"Mpumalanga",
  "north west":"North West",
  "province of north-west":"North West",
  "northern cape":"Northern Cape",
  "western cape":"Western Cape",
  "province of the western cape":"Western Cape",
  "province of eastern cape":"Eastern Cape",
  "western cape ":"Western Cape",
  "western  cape":"Western Cape"
}

def norm_div(s):
  if not s: return s
  t = s.strip()
  t = re.sub(r'^(province of\s+)', '', t, flags=re.I)  # drop "Province of" prefix
  t = re.sub(r'[^\w\s\-]', '', t)                      # remove stray punctuation like '?'
  k = t.lower().strip()
  return canon.get(k, t)  # fall back to cleaned original if unknown

with open(infile, newline='') as f, open(outfile, "w", newline='') as g:
  r = csv.DictReader(f, delimiter="\t")
  w = csv.DictWriter(g, delimiter="\t", fieldnames=r.fieldnames)
  w.writeheader()
  for row in r:
    row["division"] = norm_div(row.get("division",""))
    # we keep 'location' untouched; we'll drop city-level mapping in export unless lat/longs are supplied
    loc = row.get("location","") or ""
    row["location"] = loc.strip()
    w.writerow(row)

print(f"Wrote {outfile}")
PY
```

✅ **Checkpoint:** `curated_geo.tsv` exists.

Eyeball unique province names:
```
awk -F'\t' 'NR==1{for(i=1;i<=NF;i++) if($i=="division"){c=i;break;}; next} {print $c}' \
  curated_geo.tsv | sort -u | sed -n '1,15p'
```

---

#### d) Align (multiple sequence alignment)

Build an MSA for tree inference. (If you output an aligned FASTA from Nextclade earlier, you can reuse that and skip this step.)

```
docker run -it --rm -v "$PWD":/data -w /data nextstrain/base \
  augur align \
    --sequences curated.fasta \
    --output aligned.fasta \
    --fill-gaps
```

✅ **Checkpoint:** `aligned.fasta` exists and has the same number of sequences as `curated.fasta`.

**Alternative:** If you generated `/data/nextclade_aligned.fasta` in the Nextclade step, you can:
```
cp nextclade_aligned.fasta aligned.fasta
```
...and skip `augur align`.

---

#### e) Tree (phylogenetic inference)

Infer a maximum-likelihood tree from the alignment.

```
docker run -it --rm -v "$PWD":/data -w /data nextstrain/base \
  augur tree \
    --alignment aligned.fasta \
    --output tree_raw.nwk \
    --nthreads 2
```

✅ **Checkpoint:** `tree_raw.nwk` exists.

**Notes**

-   `--nthreads` speeds up tree building on multi-core machines.
-   For very small sets, this is quick; for larger, consider increasing threads or further downsampling.

---

#### f) Refine (time-scale the tree + attach metadata)

Refine does date inference (using `date`/`year`/`month` from your metadata) and writes a time-scaled tree.

```
docker run -it --rm -v "$PWD":/data -w /data nextstrain/base \
  augur refine \
    --tree tree_raw.nwk \
    --alignment aligned.fasta \
    --metadata curated_geo.tsv \
    --output-tree tree.nwk \
    --timetree
```

✅ **Checkpoint:** tree.nwk exists (final time-scaled tree).

```
echo "Curated seqs:" $(grep -c '^>' curated.fasta)
```

This sets you up for export & visualization. Next we’ll run augur export v2 and view in Auspice.

---

### Common gotchas (and fast fixes)

-   **"Sequence ids must be unique."**\
    Re-check header normalization & de-dup (Section 1.3.1) and ensure `--metadata-id-columns strain` matches the **FASTA headers**.

-   **Downsampling removed everything.**\
    Try fewer grouping fields (e.g., `--group-by nextclade_clade` **only**) or increase `--sequences-per-group`.

-   **Dates missing / bad** → time tree looks odd.\
    Ensure `date`/`year`/`month` exist (Section 1.3.3). If many are missing, refine has little temporal signal.

-   **Province map not rendering later in Auspice.**\
    Make sure you ran the **province normaliser** above and use `curated_geo.tsv` for refine and export.

-   **Reproducible subsampling**\
    Use a seed on `augur filter` if your Augur build supports it (or switch to `nextstrain/cli:latest`).


* * * * *


### 2.6 Export to Auspice & view

Make a directory for the auspice data.

```
mkdir -p auspice
```

```
docker run -it --rm -v "$PWD":/data -w /data nextstrain/base \
  augur export v2 \
    --tree tree.nwk \
    --metadata curated_geo.tsv \
    --color-by-metadata nextclade_clade Clade year month country division Host \
    --metadata-columns date \
    --geo-resolutions country division \
    --title "H3N2 South Africa (2018–2025)" \
    --maintainers "Workshop Team <team@example.org>" \
    --output auspice/gisaid-h3n2-ha.json
```

```
docker run -it --rm \
  -v "$PWD":/data -w /data \
  -p 4010:4000 \
  nextstrain/base \
  nextstrain view --host 0.0.0.0 auspice/
```

Open: `http://localhost:4010/gisaid-h3n2-ha`

✅ **Checkpoint:** In Auspice, try **Color by**: `nextclade_clade`, `year`, `country`.


* * * * *



### 2.7 (Optional) Add **your own sequences** to the tree

**Prepare your files**

-   `myseqs.fasta` (headers like `>MYSEQ_001`, `>MYSEQ_002`, ...)

-   `my_meta.tsv` (same columns as `metadata_final.tsv`; **must** have `strain` = FASTA header)

Minimal template (copy to `my_meta.tsv` and edit):

```
strain	Isolate_Id	Isolate_Name	Subtype	Clade	nextclade_clade	Host	region	country	division	location	Collection_Date	date	year	month	qc.overallStatus
MYSEQ_001	MYSEQ_001	A/South_Africa/Seq1/2023	H3N2		Human	Africa	South Africa	Western Cape	Cape Town	2023-05-15	2023-05-15	2023	05	pass
MYSEQ_002	MYSEQ_002	A/South_Africa/Seq2/2024	H3N2		Human	Africa	South Africa	Gauteng	Johannesburg	2024-02-01	2024-02-01	2024	02	pass
```

> Better: run Nextclade on `myseqs.fasta` too and fill `nextclade_clade`/`qc.overallStatus` from its TSV, but it's optional for the demo.

**Combine with GISAID**

Combine FASTA

```
cat gisaid_ha.acc.uniq.fasta myseqs.fasta > combined.fasta
```

Combine metadata (header from GISAID metadata_final.tsv)

```
(head -n 1 metadata_final.tsv && tail -n +2 metadata_final.tsv && tail -n +2 my_meta.tsv) \
  > combined_meta.tsv
```

**Re-run Augur on the combined set**

```
docker run -it --rm -v "$PWD":/data -w /data nextstrain/base \
  augur index --sequences combined.fasta --output raw.idx
```

```
docker run -it --rm -v "$PWD":/data -w /data nextstrain/base \
  augur filter \
    --metadata combined_meta.tsv \
    --sequences combined.fasta \
    --sequence-index raw.idx \
    --metadata-id-columns strain \
    --group-by nextclade_clade year \
    --sequences-per-group 10 \
    --output-sequences curated.fasta \
    --output-metadata curated.tsv

# (align, tree, refine, export, view) — same as Steps 5–6
```

✅ **Checkpoint:** your `MYSEQ_*` tips appear in Auspice alongside GISAID references.


* * * * *
