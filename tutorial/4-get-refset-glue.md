## 4. Command-line approach with Flu-GLUE

 

### 4.1. Download the isolate table


* * * * *

### 4.2. Filter to select isolates

```
# Step 1: initial filter
awk -F'\t' -v rc="$REC_COL" -v hc="$HOST_COL" -v yc="$YEAR_COL" -v cc="$CTRY_COL" -v sc="$SEG4_COL" \
  'NR>1 && $rc ~ /^H3N2/ && $hc=="Homo sapiens" && $yc>=2015 && $yc<=2024 && $sc!="" {print $yc "\t" $cc "\t" $sc}' \
  iav_nuccore_isolates.tsv | sort -u > H3N2_candidates.tsv

# Step 2: exclude USA
awk -F'\t' '$2 != "USA"' H3N2_candidates.tsv > H3N2_candidates_noUSA.tsv

```

Now you can continue with subsampling or direct accession extraction from `H3N2_candidates_noUSA.tsv`.

* * * * *

### 4.3. Fetch HA sequences

```
: > H3N2_HA.fasta
awk 'NR%200==1{printf "%s",$0; next} NR%200>1{printf ",%s",$0} NR%200==0{print ""} END{if(NR%200) print ""}' \
  H3N2_HA_accessions.txt \
| while IFS= read -r batch; do
    efetch -db nucleotide -id "$batch" -format fasta >> H3N2_HA.fasta
    sleep 0.34   # be polite to NCBI (export NCBI_API_KEY=... to go faster)
  done

# Quick check
grep -c '^>' H3N2_HA.fasta
head -n 2 H3N2_HA.fasta
```

* * * * *
      
### 4.4. QA using NextClade

```
# Nextclade
docker run --rm -it -v "$PWD":/data -w /data nextstrain/nextclade:latest run \
  --input-dataset flu/h3n2/ha \
  --output-dir nextclade_H3N2 \
  H3N2_HA.fasta
```

* * * * *

### 4.5. Augur: tree + refine

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

### 4.6. View in Auspice

```
# Export for Auspice + view
mkdir -p auspice
docker run --rm -it -v "$PWD":/data -w /data nextstrain/base augur export v2 \
  --tree H3N2.refined.tree.nwk \
  --metadata iav_nuccore_isolates.tsv \
  --node-data H3N2.refined.node.json \
  --output auspice/H3N2.json

docker run -it --rm -v "$PWD":/data -w /data -p 4000:4000 nextstrain/base \
  auspice view --datasetDir auspice --host 0.0.0.0
# open http://localhost:4000
```

Then open:

👉 <http://localhost:4000>[](http://localhost:4000)

* * * * *
