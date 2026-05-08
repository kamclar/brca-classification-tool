# gnomAD API for BRCA1/2 Variant Classification

## Overview

gnomAD (Genome Aggregation Database) provides allele frequency data from >140,000 exomes and >76,000 genomes. For BRCA1/2 classification according to ENIGMA VCEP guidelines, we need to query specific datasets.

## ENIGMA Requirements

ENIGMA VCEP v1.2 specifies:
- Use **gnomAD v2.1** (exomes) + **v3.1** (genomes)
- Use **non-cancer subset** (excludes individuals with cancer)
- Read depth must be **>= 25** at the variant position
- **PM2 is NOT applicable** for insertions, deletions, or delins

## API Endpoint

```
https://gnomad.broadinstitute.org/api
```

This is a GraphQL API. You send POST requests with a query and variables.

## Available Datasets

| Dataset ID | Description | Genome Build |
|------------|-------------|--------------|
| gnomad_r2_1 | v2.1 exomes + genomes | GRCh37 |
| gnomad_r2_1_non_cancer | v2.1 non-cancer subset | GRCh37 |
| gnomad_r3 | v3 genomes only | GRCh38 |
| gnomad_r4 | v4 exomes + genomes | GRCh38 |

For ENIGMA compliance, use `gnomad_r2_1_non_cancer`.

## Variant ID Format

```
chrom-pos-ref-alt
```

Examples:
- SNV: `17-41256138-A-C`
- Deletion: `13-32954022-CA-C`
- Insertion: `17-41209079-A-AT`

Note: BRCA1 is on the minus strand, so cDNA notation is reverse complement of genomic.

## GraphQL Query Example

```graphql
query GnomadVariant($variantId: String!, $dataset: DatasetId!) {
    variant(variantId: $variantId, dataset: $dataset) {
        variantId
        chrom
        pos
        ref
        alt
        exome {
            ac      # allele count
            an      # allele number  
            af      # allele frequency
            ac_hom  # homozygote count
            filters # QC filters
        }
        genome {
            ac
            an
            af
            ac_hom
            filters
        }
        flags
    }
}
```

## Python Implementation

```python
import requests

GNOMAD_API_URL = "https://gnomad.broadinstitute.org/api"

def query_gnomad(variant_id: str, dataset: str = "gnomad_r2_1_non_cancer"):
    query = """
    query GnomadVariant($variantId: String!, $dataset: DatasetId!) {
        variant(variantId: $variantId, dataset: $dataset) {
            variantId
            exome { ac an af filters }
            genome { ac an af filters }
        }
    }
    """
    
    response = requests.post(
        GNOMAD_API_URL,
        json={"query": query, "variables": {"variantId": variant_id, "dataset": dataset}},
        headers={"Content-Type": "application/json"},
        timeout=30
    )
    
    data = response.json()
    return data.get("data", {}).get("variant")
```

## ENIGMA Frequency Thresholds

| Criterion | Threshold | Points | Classification |
|-----------|-----------|--------|----------------|
| BA1 | AF > 0.1% (0.001) | Stand-alone | Benign |
| BS1_Strong | AF > 0.01% (0.0001) | -4 | Toward Benign |
| BS1_Supporting | AF > 0.002% (0.00002) | -1 | Toward Benign |
| PM2_Supporting | Absent, depth >= 25 | +1 | Toward Pathogenic |

## Important Notes

1. **Rate Limiting**: Add delays between API calls (0.5-1 second) to avoid being blocked.

2. **Variant Not Found**: If the API returns no data, the variant is absent from gnomAD. This triggers PM2_Supporting for SNVs.

3. **PM2 for Indels**: ENIGMA explicitly states "Do not apply for insertion, deletion or delins variants." The tool must block PM2 for these.

4. **Coordinate Conversion**: 
   - gnomAD v2.1 uses GRCh37
   - gnomAD v3/v4 uses GRCh38
   - HGVS c. notation is transcript-relative and doesn't change
   - Genomic coordinates require liftover between builds

5. **Coverage Check**: Ideally check that read depth >= 25 at the position. This requires a separate coverage query.

## Coverage Query (Optional)

```graphql
query Coverage($dataset: DatasetId!, $chrom: String!, $start: Int!, $stop: Int!) {
    region(dataset: $dataset, chrom: $chrom, start: $start, stop: $stop) {
        coverage {
            exome {
                mean
            }
            genome {
                mean
            }
        }
    }
}
```

## References

- gnomAD Browser: https://gnomad.broadinstitute.org
- gnomAD API: https://gnomad.broadinstitute.org/api
- ENIGMA VCEP: https://cspec.genome.network/cspec/ui/svi/doc/GN092
- gnomAD paper: https://doi.org/10.1038/s41586-020-2308-7
