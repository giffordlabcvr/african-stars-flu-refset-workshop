How curated reference sets improve alignment
============================================

**What goes wrong with un-curated inputs**

-   **Non-homologous mix**: different segments, partial fragments (e.g., HA1 only vs full HA), or mismatched start/stop → aligner forces bogus gaps to "explain" apples vs oranges.

-   **Low-quality sequences**: many Ns, frameshifts, primer/adaptor tails, lab-passage artifacts → gap pile-ups and spurious indels.

-   **Redundancy & imbalance**: hundreds of near-duplicates from one outbreak dominate the scoring and "pull" the alignment, hiding true diversity.

-   **Over-divergence without "bridges"**: if distant clades are included with few intermediates, progressive alignment can misplace blocks or over-gapp variable loops.

-   **Orientation/feature mismatches**: mixed sense, UTRs included for some, CDS-only for others → inconsistent coordinates and codon frames.

**What a curated reference set does (mechanisms)**

-   **Guarantees homology**: one segment/region per set, consistent trimming (e.g., HA CDS only), correct strand/orientation.

-   **Anchors coordinates**: picking a well-annotated reference sequence (the "seed") pins positions, so features (sites, glycosylation motifs) line up across samples.

-   **Removes noise before alignment**: exclude low-QC sequences, mask primer tails/low-complexity ends → far fewer spurious gaps.

-   **Balances diversity**: include exemplars across clades/time/regions (not just duplicates) so the aligner "sees" real evolutionary signal, not sampling bias.

-   **Preserves codon structure** (when aligning coding regions): trimming to the same CDS and keeping frame makes downstream dN/dS, mutation calling, and site numbering reliable.

-   **Speeds & stabilizes the algorithm**: fewer, better, more even sequences reduce pathological local optima and runtime.


* * * * *
