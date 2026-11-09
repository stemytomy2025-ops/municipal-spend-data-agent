# Spend Benchmark Agent


A Claude agent answers procurement questions over Umeå kommun's 2025 supplier invoices (383,902 invoice lines, about SEK 5.9 billion): spend by account, supplier, department and month, plus invoice-size benchmarks. Every figure comes from a validated tool, states the aggregation used and the number of rows behind it, and the tool calls are shown.

Ask questions in plain English; a Claude agent answers by calling validated Python tools over
the real **Umeå kommun supplier invoices 2025, monthly spend by department, supplier and account** dataset (CC0 (licence field in the opendata.umea.se catalog metadata)). Every number in an answer comes from a tool
result, and the app shows which tools ran, with their inputs and outputs.

## Example questions

- Which accounts did Umeå kommun spend the most on in 2025, and how many invoice lines are behind each figure?
- For food (Livsmedel), which suppliers got the most spend in 2025, and what share did the largest one have?
- How does food (Livsmedel) spend compare across departments in 2025?
- How did rent (Lokalhyra) spend develop month by month in 2025?

## How it works

| Tool | Operation | What it does |
|---|---|---|
| `spend_by_account` | aggregate | Rank account categories (konto_text) by spend or invoice-line count, optionally within one department, month or supplier. Data: Umeå kommun supplier invoices 2025 summed per month (datum, YYYYMM), department (forvaltning), supplier (leverantor) and account text (konto_text). belopp is SEK excluding VAT; antal_rader is the number of invoice lines. Use agg='sum' for totals: 'mean' and 'median' describe month-department-supplier-account cells, not invoices. The 'rows' in the result are cells, so sum antal_rader to count invoice lines. |
| `spend_by_supplier` | aggregate | Rank suppliers (leverantor) by spend or invoice-line count, optionally within one account, department or month. Returns the top 50; use it to find exact supplier names and to measure supplier concentration in an account. Data: Umeå kommun supplier invoices 2025 summed per month (datum, YYYYMM), department (forvaltning), supplier (leverantor) and account text (konto_text). belopp is SEK excluding VAT; antal_rader is the number of invoice lines. Use agg='sum' for totals: 'mean' and 'median' describe month-department-supplier-account cells, not invoices. The 'rows' in the result are cells, so sum antal_rader to count invoice lines. |
| `spend_by_department` | aggregate | Compare departments (forvaltning) on spend or invoice-line count, optionally within one account, supplier or month. This is the peer benchmark: one department against the others. Data: Umeå kommun supplier invoices 2025 summed per month (datum, YYYYMM), department (forvaltning), supplier (leverantor) and account text (konto_text). belopp is SEK excluding VAT; antal_rader is the number of invoice lines. Use agg='sum' for totals: 'mean' and 'median' describe month-department-supplier-account cells, not invoices. The 'rows' in the result are cells, so sum antal_rader to count invoice lines. |
| `spend_by_month` | aggregate | Monthly spend or invoice-line count for 2025 (group = datum, YYYYMM), optionally within one department, account or supplier. Results are sorted by value, not by month. Data: Umeå kommun supplier invoices 2025 summed per month (datum, YYYYMM), department (forvaltning), supplier (leverantor) and account text (konto_text). belopp is SEK excluding VAT; antal_rader is the number of invoice lines. Use agg='sum' for totals: 'mean' and 'median' describe month-department-supplier-account cells, not invoices. The 'rows' in the result are cells, so sum antal_rader to count invoice lines. |
| `invoice_size_by_account` | aggregate | Invoice-line size benchmark per account (konto_text): median, mean, min, max, sum or count of belopp, optionally within one department. Compare median with mean to show skew, and report the 'rows' behind each figure. Data: individual supplier invoice lines of Umeå kommun for September 2025 only. belopp is SEK excluding VAT per line; negative values are credits. |
| `find_invoice_lines` | filter_rows | List individual invoice lines, filtered by department, account or supplier and sorted by any returned column (for example the largest lines by belopp). Use it to show the evidence behind a figure. Data: individual supplier invoice lines of Umeå kommun for September 2025 only. belopp is SEK excluding VAT per line; negative values are credits. |

- The model (`claude-opus-5-5`) only chooses tools and arguments. Arguments are validated against
  the real data (allowed metrics, filter values, date ranges, feature bounds); it never writes
  or runs code.
- Each question is limited to 6 model turns and 8 tool calls.
- See [docs/architecture.md](docs/architecture.md) for details.

## Quickstart

Requires Python 3.11+.

```bash
python -m venv .venv
```

Activate it (`.venv\Scripts\Activate.ps1` on Windows, `source .venv/bin/activate` elsewhere),
then:

```bash
pip install -e ".[dev]"
```

```bash
python scripts/fetch_data.py
```

```bash
streamlit run app.py
```

For the chat tab, set `ANTHROPIC_API_KEY` in your environment or in
`.streamlit/secrets.toml` (see `.streamlit/secrets.toml.example`). The tool runner, evaluation
and data tabs work without a key.

## Tests and evaluation

```bash
pytest
```

Tests run offline on SYNTHETIC fixtures with a fake Anthropic client. After downloading the
data, the real-data evals run too.

```bash
python scripts/pin_expectations.py
```

```bash
python scripts/run_eval.py
```

`pin_expectations.py` recomputes the expected answers from the raw data with pyarrow and plain
Python, independently of the tool code. `run_eval.py --agent` also runs agent-level evals
(calls the API, costs tokens).

## Data and attribution

Umeå kommun supplier invoices 2025, monthly spend by department, supplier and account, https://opendata.umea.se/explore/dataset/leverantorsfakturor-2025/information/ (licence: CC0 (licence field in the opendata.umea.se catalog metadata)). Full details, evidence and limitations:
[docs/data_sources.md](docs/data_sources.md). The data is downloaded at setup time and is not
committed to this repository.

## Client view

The app opens in a client view with three tabs: **Ask the assistant**, **About this project**
([docs/about.md](docs/about.md)) and **Data & sources**. Set `"show_developer_tabs": true` in
`app_config.json` to bring back **Run a tool** and **Evaluation**, which take JSON arguments
and need no API key. With the developer tabs hidden, every visitor question uses API tokens.

## Deployment

The app runs on Streamlit Community Cloud: `requirements.txt` installs the package, and the
API key goes into the app's **Settings → Secrets**. The data is downloaded and validated
automatically on first start; **Download / refresh data** in the sidebar fetches it again.
