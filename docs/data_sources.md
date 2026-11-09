# Data sources

## Umeå kommun supplier invoices 2025, monthly spend by department, supplier and account

- Landing page: https://opendata.umea.se/explore/dataset/leverantorsfakturor-2025/information/
- Licence: **CC0 (licence field in the opendata.umea.se catalog metadata)**
- Attribution: Umeå kommun supplier invoices 2025, monthly spend by department, supplier and account, https://opendata.umea.se/explore/dataset/leverantorsfakturor-2025/information/ (licence: CC0 (licence field in the opendata.umea.se catalog metadata))
- Download method: `{'kind': 'http_csv', 'url': 'https://opendata.umea.se/api/explore/v2.1/catalog/datasets/leverantorsfakturor-2025/exports/csv?delimiter=%2C&select=datum,forvaltning,leverantor,konto_text,sum(belopp)%20as%20belopp,count(*)%20as%20antal_rader&group_by=datum,forvaltning,leverantor,konto_text', 'max_mb': 300}`
- Inspected on 2026-11-09 (sha256 of the inspected sample `052dd63124d86f94…`)

| Column | Validated type |
|---|---|
| `datum` | integer |
| `forvaltning` | string |
| `leverantor` | string |
| `konto_text` | string |
| `belopp` | float |
| `antal_rader` | integer |

### Limitations

- Automated inspection: Snapshot inspected 2026-11-09; the source may change after this date.
- Automated inspection: Profiling stopped at 50000 rows.
- Server-side aggregation of the 383,902 invoice lines of 2025 (catalog record count) by month, department, supplier and account text; belopp is the summed amount in SEK excluding VAT, antal_rader the number of invoice lines.
- Some supplier names are masked with asterisks; the catalog description of an earlier year says some data is hidden under GDPR.
- datum is the posting month as an integer YYYYMM, not a date.

## Umeå kommun supplier invoice lines, September 2025

- Landing page: https://opendata.umea.se/explore/dataset/leverantorsfakturor-2025/information/
- Licence: **CC0 (licence field in the opendata.umea.se catalog metadata)**
- Attribution: Umeå kommun supplier invoice lines, September 2025, https://opendata.umea.se/explore/dataset/leverantorsfakturor-2025/information/ (licence: CC0 (licence field in the opendata.umea.se catalog metadata))
- Download method: `{'kind': 'http_csv', 'url': 'https://opendata.umea.se/api/explore/v2.1/catalog/datasets/leverantorsfakturor-2025/exports/csv?delimiter=%2C&where=datum%3D%22202509%22', 'max_mb': 300}`
- Inspected on 2026-11-09 (sha256 of the inspected sample `ad442f76a2940079…`)

| Column | Validated type |
|---|---|
| `datum` | integer |
| `forvaltning` | string |
| `verifikationsnummer` | integer |
| `leverantor` | string |
| `konto_text` | string |
| `belopp` | float |

### Limitations

- Automated inspection: Snapshot inspected 2026-11-09; the source may change after this date.
- One month only (September 2025): invoice-level figures say nothing about other months.
- Some supplier names are masked with asterisks; the catalog description of an earlier year says some data is hidden under GDPR.

## Alternatives considered and rejected

- **City of Helsinki procurements, September 2026**: access failed on 2026-11-09: network error: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1081).
- **Leverandørregnskap Stavanger kommune (supplier ledger)**: The CSV holds only a header row (146 bytes); the opencom.no page marks the dataset Utgått/Ended and says accounting datasets are on hold.
