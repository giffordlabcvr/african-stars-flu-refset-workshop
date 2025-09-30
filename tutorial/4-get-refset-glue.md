## 3 Command-line approach with data downloaded from **GISAID** (web)

<img src="../images/glue.png" align="right" alt="" width="180"/>

### 1. Download the isolate table

```
wget https://raw.githubusercontent.com/giffordlabcvr/Flu-GLUE/refs/heads/main/tabular/extension/iav_nuccore_isolates.tsv -O iav_isolates.tsv
```

### 2. Extract HA accessions

```
awk -F'\t' '$3 == "H3N2"' iav_isolates.tsv > iav_isolates_H3N2.tsv
```
    -   Column 3 = `rec_serotype`.
    -   You can replace `"H3N2"` with `"H1N1"` etc. for other gene sets.

### 3. Extract HA accessions

```
cut -f4,12,13,14,15 iav_isolates_H3N2.tsv | cut -f12 > H3N2_HA_accessions.txt
```
    -   Here you're pulling `segment4_accession`.
    -   (Better: use `awk -F'\t' '{print $13}'` since segment4 is column 13 in your example file --- we'll double-check column indices before finalizing.)

      
### 4. Extract HA accessions

```
cat H3N2_HA_accessions.txt | \
  while read acc; do
    efetch -db nucleotide -id "$acc" -format fasta >> H3N2_HA.fasta
  done
```

(using `ncbi-entrez-direct` package)

### 5. Run through Nextclade

```
nextclade run \
  --input-dataset flu/h3n2/ha \
  --output-dir nextclade_H3N2 \
  H3N2_HA.fasta
```

This produces:

-   `nextclade_H3N2/nextclade.aligned.fasta` (aligned sequences)
-   `nextclade_H3N2/nextclade.tsv` (QC + clades)

### 6. Build a tree with Augur & view in Auspice

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

### 7. Export to Auspice JSON

```
augur export v2 \
  --tree H3N2.refined.tree.nwk \
  --metadata iav_isolates_H3N2.tsv \
  --node-data H3N2.refined.node.json \
  --output auspice/H3N2.json
```


### 8. Export to Auspice JSON

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
