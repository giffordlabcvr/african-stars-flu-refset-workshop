## 3. Command-line approach with Flu-GLUE

 

### 3.1. Download the isolate table


* * * * *

### 3.2. Map your TSV column names to positions (once)

**Why:** The AWK filters below expect column indices. We'll quickly map the header names to indices so the rest is deterministic.

```
# Inspect header with numbered columns\
head -1 iav_nuccore_isolates.tsv | tr '\t' '\n' | nl
```

Now set environment variables to the **column numbers** (from the left, starting at 1):

```
# Example names; adjust numbers to match YOUR header indices
export REC_COL=1            # column containing subtype/record info (e.g., "H3N2 ...")
export HOST_COL=2           # host (e.g., "Homo sapiens")
export YEAR_COL=3           # collection year (e.g., 2018)
export CTRY_COL=4           # country (e.g., "USA")
export SEG4_COL=5           # segment4_accession (HA nucleotide accession)
```
✅ **Checkpoint:** echo them back

```
echo $REC_COL $HOST_COL $YEAR_COL $CTRY_COL $SEG4_COL
```

### 3.3. Select H3N2 human candidates with HA accession present (2015--2024)

**What:** Keep header; then keep rows where:

-   subtype/record **starts with `H3N2`**
-   **host = Homo sapiens**
-   **year between 2015 and 2024**
-   **segment4_accession present (non-empty)**


```
awk -F'\t' -v rc="$REC_COL" -v hc="$HOST_COL" -v yc="$YEAR_COL" -v sc="$SEG4_COL" \
  'NR==1 || (NR>1 && $rc ~ /^H3N2/ && $hc=="Homo sapiens" && $yc>=2015 && $yc<=2024 && $sc!="")' \
  iav_nuccore_isolates.tsv > H3N2_candidates.tsv
```

✅ **Checkpoint:**


```
wc -l H3N2_candidates.tsv
head -3 H3N2_candidates.tsv
```

### 3.4. Exclude USA records (keep header)

```
awk -F'\t' -v CTRY_COL="$CTRY_COL" 'NR==1 || $CTRY_COL != "USA"' \
  H3N2_candidates.tsv > H3N2_candidates_noUSA.tsv
```

✅ **Checkpoint:**

```
grep -c $'\tUSA\t' H3N2_candidates_noUSA.tsv   # ideally 0
```

### 3.5. Extract HA (segment 4) accessions (unique)

```
awk -F'\t' -v sc="$SEG4_COL" 'NR>1 && $sc!="" {print $sc}' H3N2_candidates_noUSA.tsv \
  | sort -u > H3N2_HA_accessions.txt
```

✅ **Checkpoint:**

```
wc -l H3N2_HA_accessions.txt
sed -n '1,5p' H3N2_HA_accessions.txt
```

* * * * *

### 3.6. Fetch HA sequences from NCBI (batched efetch)

**Why batching:** URLs have length limits; batching **200 IDs** per request is safe & fast.

```
: > H3N2_HA.fasta
awk 'NR%200==1{printf "%s",$0; next} NR%200>1{printf ",%s",$0} NR%200==0{print ""} END{if(NR%200) print ""}' \
  H3N2_HA_accessions.txt \
| while IFS= read -r batch; do
    efetch -db nucleotide -id "$batch" -format fasta >> H3N2_HA.fasta
    sleep 0.34   # be polite to NCBI (export NCBI_API_KEY=... to go faster)
  done
```

✅ Checkpoint:

```
grep -c '^>' H3N2_HA.fasta
head -n 2 H3N2_HA.fasta
```

* * * * *
      
### 3.7. Normalize FASTA headers to ACCESSION.VERSION only

**Why:** Some efetch headers contain extra text; we keep just the first whitespace-separated token so names are predictable.

```
awk '
  /^>/ {
    split($0, a, " ");   # keep only >ACCESSION.VERSION
    print a[1];
    next
  }
  { print }
' H3N2_HA.raw.fasta > H3N2_HA.fasta
```

✅ Checkpoint:

```
head -4 H3N2_HA.fasta
```

* * * * *


### 3.7. Build a sequence metadata table with explicit `seq_id`

We'll **prepend a `seq_id` column** that equals the **segment4_accession**, preserving all original columns. This keeps a clean join key for tools that expect `seq_id`.

```
awk -F'\t' -v OFS='\t' -v sc="$SEG4_COL" '
  NR==1 { print "seq_id", $0; next }
  { print $sc, $0 }
' H3N2_candidates_noUSA.tsv > H3N2_HA.metadata.tsv
```

✅ **Checkpoint:**

```
head -1 H3N2_HA.metadata.tsv | tr '\t' '\n' | nl
sed -n '2,4p' H3N2_HA.metadata.tsv
```

### 3.8. Nextclade: fetch H3N2 HA dataset & run

Pull image (once)


```
docker pull nextstrain/nextclade:latest
```


```
# Nextclade
docker run --rm -it -v "$PWD":/data -w /data nextstrain/nextclade:latest run \
  --input-dataset flu/h3n2/ha \
  --output-dir nextclade_H3N2 \
  H3N2_HA.fasta
```

* * * * *

### 3.5. Augur: tree + refine

```
docker run --rm -it -v "$PWD":/data -w /data nextstrain/base augur tree \
  --alignment nextclade_H3N2/nextclade.aligned.fasta \
  --output-tree H3N2.tree.nwk
```

```
docker run --rm -it -v "$PWD":/data -w /data nextstrain/base augur refine \
  --tree H3N2.tree.nwk \
  --alignment nextclade_H3N2/nextclade.aligned.fasta \
  --metadata iav_nuccore_isolates.tsv \
  --output-tree H3N2.refined.tree.nwk \
  --output-node-data H3N2.refined.node.json
```

* * * * *

### 3.6. View in Auspice

```
# Export for Auspice + view
mkdir -p auspice
docker run --rm -it -v "$PWD":/data -w /data nextstrain/base augur export v2 \
  --tree H3N2.refined.tree.nwk \
  --metadata iav_nuccore_isolates.tsv \
  --node-data H3N2.refined.node.json \
  --output auspice/H3N2.json
```

```
docker run -it --rm -v "$PWD":/data -w /data -p 4000:4000 nextstrain/base \
  auspice view --datasetDir auspice --host 0.0.0.0
# open http://localhost:4000
```

Then open:

👉 <http://localhost:4000>[](http://localhost:4000)

* * * * *
