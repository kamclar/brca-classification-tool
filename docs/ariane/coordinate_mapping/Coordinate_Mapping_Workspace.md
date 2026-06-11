# BRCA Coordinate Mapping Workspace

## Goal

Replace bulk per-variant online coordinate resolution with local transcript to
genome mapping for SNVs in the BRCA1 and BRCA2 ENIGMA reference transcripts.

Reference transcripts:

- BRCA1: NM_007294.4 / ENST00000357654.9
- BRCA2: NM_000059.4 / ENST00000380152.8

  
The old/current resolver calls external services per variant:

- VariantValidator
- Mutalyzer fallback

That is acceptable for small validation batches, but too slow and fragile for
the full SNV scan.

## Local Mapping Plan

For coding SNVs, coordinates can be derived locally from transcript exon/CDS
structure:

1. Load exon and CDS coordinates for the reference transcript on GRCh37 and
   GRCh38.
2. Build a CDS-position to genomic-position map.
3. For each SNV in `brca_snv_manifest.csv`, map `cds_pos` to genomic position.
4. Reverse-complement alleles for minus-strand transcripts where needed.
5. Validate a random sample against VariantValidator.
6. Write a scan-owned coordinate cache compatible with
   `variant_space_scan/precompute_coordinates.py`.

