## What this is

A question-answering assistant for procurement spend. You ask in plain English, an AI agent
works out which calculations to run, runs them on real invoice data, and explains the result:
what the number is, how it was calculated and how many records are behind it.

## The data

**Umeå kommun**, a Swedish municipality, publishes its incoming supplier invoices as open data
on [opendata.umea.se](https://opendata.umea.se/explore/dataset/leverantorsfakturor-2025/information/).
This app uses the 2025 dataset in two forms:

- **The 2025 spend cube.** All **383,902 invoice lines** of 2025, about **SEK 5.9 billion**
  excluding VAT, summed per month, department, supplier and account. That gives 52,673 rows
  across 17 departments, about 4,300 supplier names and 224 account categories.
- **September 2025 invoice lines.** The **33,878 individual invoice lines** of one month
  (about SEK 510 million), used for questions about what a typical invoice looks like.

Each record says which department paid, which supplier was paid, which account the cost was
booked on (for example *Livsmedel*, food, or *Lokalhyra*, rent of premises) and the amount in
SEK excluding VAT. Department and account names are in Swedish, as published.

## What you can ask

- **Where the money goes:** "Which accounts did Umeå kommun spend the most on in 2025, and how
  many invoice lines are behind each figure?"
- **Supplier concentration:** "For food (Livsmedel), which suppliers got the most spend in 2025,
  and what share did the largest one have?"
- **Benchmark one department against the others:** "How does food (Livsmedel) spend compare
  across departments in 2025?"
- **Development over the year:** "How did rent (Lokalhyra) spend develop month by month in
  2025?"
- **What is typical:** "What is a typical food (Livsmedel) invoice line in September 2025:
  median versus mean, and why do they differ?"
- **The evidence:** "Show the five largest invoice lines for IT-Licens & databasavgift in
  September 2025."

## How the agent works

The assistant is a Claude model (Anthropic) connected to six data tools. For each question it:

1. **Plans.** It decides which tools answer the question and with which filters (department,
   account, supplier, month).
2. **Acts, in several steps if needed.** It can chain tool calls: for example, rank the
   suppliers in an account, then compare departments for the largest one, then list the
   invoice lines behind it.
3. **Recovers from mistakes.** Every request is validated before it runs. If the agent asks
   for a department or account that is not in the data, the tool says so and lists real
   values, and the agent corrects itself.
4. **Explains.** It answers in plain language and says which aggregation it used (a total, a
   median, a count) and how many rows the figure rests on.

Click **"How this answer was computed"** under any answer to see every tool call the agent made
and what each one returned.

### Built-in safeguards

- **No made-up numbers.** The agent is told to take every figure from a tool result. The tools
  are ordinary, tested Python code running on the downloaded data.
- **Bounded.** At most 6 model turns and 8 tool calls per question.
- **No code execution.** The agent can only choose among the six tools and fill in settings
  that are checked against the real data.
- **Tool output is data.** The agent is instructed to treat text in tool results as data to
  report on, not as instructions. This is an instruction to the model, not a technical barrier.

## The tools

| Tool | In plain words |
|---|---|
| `spend_by_account` | Ranks account categories by spend or number of invoice lines |
| `spend_by_supplier` | Ranks suppliers, for example inside one account (top 50) |
| `spend_by_department` | Compares departments, for example on one account |
| `spend_by_month` | Spend per month of 2025 |
| `invoice_size_by_account` | Median, mean, largest or number of invoice lines per account, September 2025 |
| `find_invoice_lines` | Lists individual September 2025 invoice lines, for example the largest |

## How it is checked

- **6 of 6 tool-level checks pass** on the real data (run on 9 November 2026). Each expected
  answer is recomputed separately from the tool code before the comparison.
- **68 automated tests pass** (1 skipped) without network access or an API key, using small
  SYNTHETIC fixtures and a stand-in for the Claude client.
- On every download the data is checked against the expected columns, types and minimum row
  counts, and a SHA-256 hash of each file is recorded.
- The agent's free-text answers have not been scored automatically.

## Limits

- One buyer and one year: this is not a benchmark across organisations.
- Invoice-level questions cover September 2025 only.
- Totals include everything booked through supplier invoices, such as construction contracts,
  grants and pension payments, unless a question narrows it to an account.
- Filters are exact matches. There is no fuzzy supplier search and no date range.
- Some supplier names are masked with asterisks in the published data. An earlier year's
  dataset description says some data is hidden under GDPR.
- Supplier rankings return the top 50.

## Data credit

Supplier invoices 2025 (*Leverantörsfakturor 2025*), published by Umeå kommun on
opendata.umea.se. The catalogue metadata lists the licence as CC0. This project is not
affiliated with or endorsed by Umeå kommun.
