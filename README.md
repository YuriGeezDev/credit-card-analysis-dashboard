# Credit Card Default Dashboard

A portfolio/demo recreation of a delinquency-tracking Power BI dashboard, rebuilt on top of a **public dataset** — no real company data involved. Same overall layout (KPI cards, slicers, ranking charts, detail table), adapted to a different domain and data source.

## Data source

- **Dataset:** Default of Credit Card Clients Dataset
- **Origin:** UCI Machine Learning Repository, donated by I-Cheng Yeh (2009); mirrored on Kaggle at `kaggle.com/datasets/uciml/default-of-credit-card-clients-dataset`
- **License:** CC BY 4.0 (Creative Commons Attribution 4.0 International). Please verify on the Kaggle page before redistributing, in case the license has changed since this was last checked.
- **Content:** 30,000 credit card clients in Taiwan, with payment history from April to September 2005 (`PAY_0`/`PAY_2`.../`PAY_6`, `BILL_AMT1-6`, `PAY_AMT1-6`) and a ground-truth flag for "will default next month" (`default payment next month`).
- **Local file:** `data/default_of_credit_card_clients.csv`

## How to open

1. Open `credit-card-default-dashboard.pbip` in Power BI Desktop.
2. On first open, Desktop will generate its own local cache files (`.pbi/`) — expected, already covered by `.gitignore`.
3. The CSV path is driven by a Power Query parameter, `ProjectFolder` (the folder this repo was cloned/extracted into). Set it once via **Transform Data → Manage Parameters → ProjectFolder**, pointing to the full local path of this folder (e.g. `C:\Users\you\Desktop\credit-card-default-dashboard`). No need to touch the Advanced Editor.

## What was adapted from the original dashboard

The public dataset has a very different shape from the original source system (one row per client, with 6 months "wide" across columns, instead of one row per financial transaction). Instead of a straight source swap, some concepts were remapped:

| Original concept | Equivalent in this demo |
|---|---|
| Overdue invoice/bill | Each month (`PAY_0`..`PAY_6`) where a client's payment status is > 0 becomes one row of "overdue payment" in `OverduePayments` |
| Salesperson/department segment | `EducationLevel` (from the dataset's `EDUCATION` column) — the public dataset has no sales department, so the closest available categorical dimension was used to keep the "segment slicer" pattern |
| Partner/client (ID + name) | Client (`ID`), with a synthetic name ("Client 000123") — the dataset is anonymized, there's no real name |
| Commission / post-default repeat purchases | **Removed.** The dataset has no repeated transactions per client over time, so there's no way to simulate "bought again after defaulting" without fabricating data that isn't in the source |
| *(did not exist in the original)* | **Actual Default Rate (%)** — bonus: uses the dataset's real ground-truth flag over all 30,000 clients, a signal the original source system never had (an ERP doesn't predict who will default, it only records who already has) |

## Important caveats (this data is an approximation — don't over-extrapolate)

- `DaysLate` is approximate: the dataset only reports how many **months** a payment was late (a payment-status code), not an exact due date. One month late was assumed to equal 30 days.
- `OutstandingAmount` = max(bill amount for the month − amount paid that month, 0). This is a reasonable simplification for demo purposes, not an accounting reconciliation (the exact semantics of `BILL_AMT`/`PAY_AMT` in the original dataset are subtler than a simple invoice).
- Monetary values are in **NT$** (New Taiwan Dollar), the dataset's original currency.
- The **"Actual Default Rate (%)"** card is calculated over the full population of 30,000 clients; the other cards (Total Outstanding, Delinquent Clients, Max Days Late) are scoped to the **Top 50** by outstanding amount. These are different population scopes shown side by side on purpose — they are not directly comparable.

## Project structure

```
data/
  default_of_credit_card_clients.csv      ← public CSV (Kaggle/UCI)
credit-card-default-dashboard.pbip
credit-card-default-dashboard.Report/         ← report layer (pages, visuals)
credit-card-default-dashboard.SemanticModel/  ← data model (tables, relationships, measures)
```

## Data model

- `Clients` — raw import of the CSV (30,000 rows, one per client)
- `OverduePayments` — unpivoted view of `Clients`: one row per client per month where they were actually late (payment status > 0)
- `Top50Clients` — top 50 clients by total outstanding amount (computed from `OverduePayments`)
- `Top50OverduePayments` — top 50 individual overdue records (client-month rows) by outstanding amount
- `DimClient` / `DimEducation` — supporting dimensions for the slicers (client names and education level)

<img width="1115" height="587" alt="image" src="https://github.com/user-attachments/assets/584750c4-e3f0-474d-aa5a-d7fe63801e29" />


## Credits

Original dataset: Yeh, I-Cheng (2009). *Default of Credit Card Clients Dataset*. UCI Machine Learning Repository. Published on Kaggle by user `uciml` under CC BY 4.0 license.
