# Student handout

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

* * * * *
