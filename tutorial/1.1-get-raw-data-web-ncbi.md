## 1.1: Get a raw dataset - web-based approach: NCBI Nucleotide

### Step 1: Go to **[NCBI Nucleotide](https://www.ncbi.nlm.nih.gov/nuccore)**.

* * * * *

### Step 2: Enter search string

See **[here](https://www.ncbi.nlm.nih.gov/books/NBK49540/)** for a query syntax reference.

Make a note how many entries are returned with each of the following queries (expected answers are provided at the bottom of this page): 

**a) Get all influenza A virus entries**:

```
Alphainfluenzavirus[Organism]
```
**b) Focus on subtype H3N2**:

```
Alphainfluenzavirus[Organism] AND H3N2[All Fields]
```

**c) Filter by sequence length (1500–1900 bp)**:

```
Alphainfluenzavirus[Organism] AND 1500:1900[SLEN]
```
**d) Add a geographic filter: South Africa**:

```
Alphainfluenzavirus[Organism] AND 1500:1900[SLEN] AND "South Africa"[All Fields]
```

**e) Add gene (HA), subtype (H3N2), length, and date range (2018–2025)**:

```
Alphainfluenzavirus[Organism] AND HA[Gene] AND H3N2[All fields] AND 1500:1900[SLEN] AND ("2018/01/01"[PDAT] : "2025/09/15"[PDAT])
```

**f) Combine length + geography only (South Africa)**

```
Alphainfluenzavirus[Organism] AND 1500:1900[SLEN] AND "South Africa"[All Fields]
```

**g) Add everything: HA, H3N2, length, date range, South Africa**

```
Alphainfluenzavirus[Organism] AND HA[Gene] AND H3N2[All fields] AND 1500:1900[SLEN] AND ("2018/01/01"[PDAT] : "2025/09/15"[PDAT]) AND "South Africa"[All Fields]
```
* * * * *

### Step 3: Send to → File → FASTA.

**Click**: 'Send to' (near top right) -> File -> Coding Sequences -> FASTA Nucleotide -> Create File

**Click**: 'Send to' (near top right) -> File -> Complete record -> File -> Select Format: FASTA -> Create File

Note the differences in the FASTA header structures used in each file.

Go to **[next section](https://github.com/giffordlabcvr/african-stars-flu-refset-workshop/blob/main/tutorial/1.2-get-raw-data-cli-ncbi.md)**

* * * * *

| Query # | Title | Number of Results |
| --- | --- | --- |
| 1 | Get all influenza A virus entries | 1,433,165 |
| 2 | Focus on subtype H3N2 | 495,818 |
| 3 | Filter by sequence length (1500--1900 bp) | 303,829 |
| 4 | Add a geographic filter: South Africa | 71,059 |
| 5 | Add gene (HA), subtype (H3N2), length, and date range (2018--2025) | 46,514 |
| 6 | Combine length + geography only (South Africa) | 2,509 |
| 7 | Add everything: HA, H3N2, length, date range, South Africa | 23 |

* * * * *
