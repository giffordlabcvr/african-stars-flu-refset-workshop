## 4. Command-line approach with Flu-GLUE

<img src="../images/glue.png" align="right" alt="" width="180"/>

### 4.0. Install EDirect (macOS/Linux/WSL) - One-off Install 

Script (official):

```
sh -c "$(curl -fsSL https://ftp.ncbi.nlm.nih.gov/entrez/entrezdirect/install-edirect.sh)"
export PATH="$PATH:$HOME/edirect"
```


### 4.1. Download the isolate table

```
wget https://raw.githubusercontent.com/giffordlabcvr/Flu-GLUE/refs/heads/main/tabular/extension/iav_nuccore_isolates.tsv -O iav_isolates.tsv
```

### 4.2. Extract HA accessions

```
awk -F'\t' '$3 == "H3N2"' iav_isolates.tsv > iav_isolates_H3N2.tsv
```
    -   Column 3 = `rec_serotype`.
    -   You can replace `"H3N2"` with `"H1N1"` etc. for other gene sets.

### 4.3. Extract HA accessions

```
cut -f4,12,13,14,15 iav_isolates_H3N2.tsv | cut -f12 > H3N2_HA_accessions.txt
```
    -   Here you're pulling `segment4_accession`.
    -   (Better: use `awk -F'\t' '{print $13}'` since segment4 is column 13 in your example file --- we'll double-check column indices before finalizing.)

      
### 4.4. Fetch Sequences
```
# Join every 200 IDs into a comma-separated batch and fetch
: > H3N2_HA.fasta
awk 'NR%200==1{printf "%s",$0; next} NR%200>1{printf ",%s",$0} NR%200==0{print ""} END{if(NR%200) print ""}' \
  H3N2_HA_accessions.txt \
| while IFS= read -r batch; do
    efetch -db nucleotide -id "$batch" -format fasta >> H3N2_HA.fasta
    sleep 0.34   # be polite to NCBI; set NCBI_API_KEY to go faster
  done

# Quick check
grep -c '^>' H3N2_HA.fasta

```

(using `ncbi-entrez-direct` package)

### 4.5. Run through Nextclade

```
nextclade run \
  --input-dataset flu/h3n2/ha \
  --output-dir nextclade_H3N2 \
  H3N2_HA.fasta
```

This produces:

-   `nextclade_H3N2/nextclade.aligned.fasta` (aligned sequences)
-   `nextclade_H3N2/nextclade.tsv` (QC + clades)

### 4.6. Build a tree with Augur & view in Auspice

```
augur tree \
  --alignment nextclade_H3N2/nextclade.aligned.fasta \
  --output-tree H3N2.tree.nwk
```

Optionally refine with metadata:

```
augur refine \
  --tree H3N2.tree.nwk \
  --alignment nextclade_H3N2/nextclade.aligned.fasta \
  --metadata iav_isolates_H3N2.tsv \
  --output-tree H3N2.refined.tree.nwk \
  --output-node-data H3N2.refined.node.json
```

### 4.7. Export to Auspice JSON

```
augur export v2 \
  --tree H3N2.refined.tree.nwk \
  --metadata iav_isolates_H3N2.tsv \
  --node-data H3N2.refined.node.json \
  --output auspice/H3N2.json
```


### 4.8. Export to Auspice JSON

Run inside a Nextstrain Docker container (or local install):

```
docker run -it --rm \
  -v "$PWD":/data -w /data \
  -p 4000:4000 nextstrain/base \
  auspice view --datasetDir auspice --host 0.0.0.0
```

Then open:

👉 <http://localhost:4000>[](http://localhost:4000)

* * * * *
