# Architecture

```mermaid
flowchart LR
    U[User] --> UI[Streamlit app.py]
    UI --> A[Agent: bounded Claude loop]
    A -- tool_use --> R[Tool registry: Pydantic validation]
    R --> O[Operations: pandas / scikit-learn]
    O --> D[(validated parquet)]
    F[scripts/fetch_data.py] --> D
    R -- tool_result / is_error --> A
    A --> UI
```

| Component | File | Responsibility |
|---|---|---|
| Data layer | `src/spend_benchmark_agent/data.py` | Download from the documented source, hash, validate schema and row count, cache as parquet |
| Operations | `src/spend_benchmark_agent/operations.py` | Deterministic pandas/scikit-learn computations |
| Tools | `src/spend_benchmark_agent/tools.py` | Bind operations to inspected columns; Pydantic input models built from observed values and ranges |
| Agent | `src/spend_benchmark_agent/agent.py` | Manual Messages API loop on claude-opus-5-5; progress events; tool errors returned as is_error |
| UI | `app.py` | Streamlit chat with progress states, charts, a direct tool runner, evals and data provenance |
| Evaluation | `src/spend_benchmark_agent/evaluation.py` | Pinned tool-level checks and agent-level checks |

## Why an agent, and how it is bounded

Questions are open-ended (which tool, which filters, which date) and often need more than one step, so a small tool-using agent fits. It stays bounded: 6 model turns, 8 tool calls, validated inputs, and no code execution. Every number in an answer comes from a tool result.

- Manual Messages API loop: the app controls the budget and streams progress to the UI.
- Tool results go back in one user message per turn; failures are `is_error` results the
  model can correct.
- `stop_reason` handling: `refusal` (with server-side fallback enabled:
  True), `max_tokens`, and the turn limit all end the run with a clear
  status.

## Tool inventory

| Tool | Operation | Bindings |
|---|---|---|
| `spend_by_account` | aggregate | group=`konto_text`, metrics=`['belopp', 'antal_rader']`, filterable=`['forvaltning', 'datum', 'leverantor']`, eval_filters=`{'forvaltning': 'Teknisk nämnd'}`, eval_args=`{'agg': 'sum'}`, eval_question=`Which account had the highest total spend in Teknisk nämnd in 2025?` |
| `spend_by_supplier` | aggregate | group=`leverantor`, metrics=`['belopp', 'antal_rader']`, filterable=`['konto_text', 'forvaltning', 'datum']`, eval_filters=`{'konto_text': 'Livsmedel'}`, eval_args=`{'agg': 'sum'}`, eval_question=`Which supplier received the most food (Livsmedel) spend in 2025?` |
| `spend_by_department` | aggregate | group=`forvaltning`, metrics=`['belopp', 'antal_rader']`, filterable=`['konto_text', 'leverantor', 'datum']`, eval_filters=`{'konto_text': 'Livsmedel'}`, eval_args=`{'agg': 'sum'}`, eval_question=`Which department spent the most on food (Livsmedel) in 2025?` |
| `spend_by_month` | aggregate | group=`datum`, metrics=`['belopp', 'antal_rader']`, filterable=`['forvaltning', 'konto_text', 'leverantor']`, eval_filters=`{'konto_text': 'Lokalhyra'}`, eval_args=`{'agg': 'sum'}`, eval_question=`In which month of 2025 was rent (Lokalhyra) spend highest?` |
| `invoice_size_by_account` | aggregate | group=`konto_text`, metrics=`['belopp']`, filterable=`['forvaltning']`, eval_filters=`{'forvaltning': 'Äldrenämnd'}`, eval_args=`{'agg': 'median'}`, eval_question=`Which account has the highest median invoice line in Äldrenämnd in September 2025?` |
| `find_invoice_lines` | filter_rows | columns=`['forvaltning', 'leverantor', 'konto_text', 'belopp', 'verifikationsnummer']`, filterable=`['forvaltning', 'konto_text', 'leverantor']`, eval_filters=`{'konto_text': 'Livsmedel'}`, eval_question=`How many food (Livsmedel) invoice lines are there in September 2025?` |

Input models are generated from the loaded data, so the JSON schema the model sees lists real
values and ranges, and invalid arguments are rejected before any computation.

## Data flow and validation

1. `fetch_data.py` downloads from the documented source, records SHA-256 hashes in a manifest,
   validates column presence and types and the minimum row count, and writes parquet.
2. The app and scripts load only that validated file. If it is missing, they say so and offer
   the download; they never substitute invented data.

## Evaluation

- Tool-level: expected values pinned by an independent implementation (pyarrow + plain
  Python).
- Agent-level (needs a key): the expected tool is called and the pinned value appears in the
  answer.

## Trade-offs

- Generic, configurable operations instead of bespoke code per tool: easier to validate and
  test, at the cost of some domain-specific nuance.
- Fixed tools instead of model-generated code: safer and reproducible, but questions outside
  the tool set are declined rather than improvised.

