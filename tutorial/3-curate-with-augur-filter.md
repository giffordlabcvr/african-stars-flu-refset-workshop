# Step 3 — Build & visualize (20 min, hands-on)

## 4.1 Align to reference

```
augur align \
  --sequences curated.fasta \
  --reference-sequence tutorial_data/reference/reference_h3n2.fasta \
  --output aligned.fasta \
  --fill-gaps
```

## 4.1 ML tree then time-refine

```
augur tree \
  --alignment aligned.fasta \
  --output tree_raw.nwk \
  --nthreads 2

augur refine \
  --tree tree_raw.nwk \
  --alignment aligned.fasta \
  --metadata curated.tsv \
  --output-tree tree.nwk \
  --output-node-data branch_lengths.json \
  --timetree
```

## 4.3 (Optional) Traits/annotations

```
augur traits \
  --tree tree.nwk \
  --metadata curated.tsv \
  --output-node-data traits.json \
  --columns clade year month country
```

## 4.4 Export to Auspice & view

```
mkdir -p auspice
augur export v2 \
  --tree tree.nwk \
  --metadata curated.tsv \
  --node-data branch_lengths.json traits.json \
  --output auspice/sa-h3n2-2018-2025.json

nextstrain view auspice/
# open http://localhost:4000
```

* * * * *


**Things to try:**

-   Colour by **clade**, then **year**.
-   Toggle **Tips only** vs full tree.
-   Search for a specific strain name.
-   If anchors were added, ask what context they provide.
  

* * * * *
