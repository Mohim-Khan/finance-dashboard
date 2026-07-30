# Finance Transactions Dashboard

An executive-level financial analytics dashboard built in Power BI, analyzing 50,000+ banking transactions across 5,000 customers to surface transaction trends, channel performance, fraud risk, and customer segment behavior across 3 report pages.

**Tools used:** Power BI Desktop, Power Query (M), DAX

---

## 1. What problem am I solving?

A bank/fintech operations team needs a single view to answer:
- Where is transaction volume and value actually coming from — which customer segments, states, and channels drive the business?
- Is transaction performance healthy — what share of transactions succeed, fail, or stay pending, and why?
- Where is fraud risk concentrated, and how do risk scores differ between flagged and clean transactions?
- How is transaction volume trending month over month, and are there seasonal spikes worth planning around?

Without a live, filterable view, this kind of analysis lives in scattered spreadsheets and can't be sliced by year, customer segment, or metric on demand. This dashboard consolidates transaction and customer data into three report pages — an **Overview** for leadership-level KPIs and trends, a **Transactions** page for row-level drill-down, and a **Risk & Fraud** page dedicated to surfacing where fraud actually concentrates — with a dynamic metric switcher so viewers can toggle between Amount, Fees, Tax, and Transaction Count without needing separate charts for each.

## 2. How did I collect, clean, and analyze the data?

**Data sources**
- `customers.csv` — 5,000 customer records (demographics, occupation, segment, income, join date)
- `finance_transactions.csv` — 50,069 transaction records (date, channel, type, amount, fee, tax, status, fraud flag, risk score)

**Data model**
- `finance_transactions` and `customers` joined on `customer_id`
- A dedicated `Calendar table` for consistent time intelligence (Year/Month slicers, trend charts)
- A disconnected `Dynamic Metric` table feeding a field-parameter-style slicer, so a single set of visuals can switch between Total Amount, Total Fee, Total Tax, and Total Transactions

**Data cleaning (Power Query)**
The raw data had several real-world quality issues that needed fixing before the numbers could be trusted:

| Issue found | Fix applied |
|---|---|
| 69 fully duplicated `transaction_id` rows | Removed duplicates — inflated Total Amount by ~1.4% if left in |
| `channel` had inconsistent formatting: leading/trailing spaces (`" Net Banking"`, `"ATM "`) and a typo variant `"M@bile App"` (765 rows) alongside `"Mobile App"` | Trimmed whitespace and corrected the typo so channel-level totals aren't artificially split |
| `currency` had mixed casing (`INR`, `inr`, `inR`) | Standardized to uppercase |
| `fee_amount` had 24 missing values | Reviewed and handled as part of fee/tax total measures |
| 9 negative transaction amounts (Loan EMI, Deposit, Transfer, Card Payment, Bill Payment) | Converted to absolute value, treating them as data entry errors rather than valid reversals |
| `customers.csv` had a header typo (`fisrt_name`) | Corrected on load |

**DAX measures built:** Total Amount, Total Transactions, Average Transaction, Total Fee, Total Tax, plus the Dynamic Metric switch measure driving the KPI cards and charts from a single slicer. For the risk page: Fraud Rate %, Fraud Transaction Count, Avg Risk Score, and separate Fraud/Clean average risk score measures, plus a calculated `Risk Score Bucket` column (binned 0–20, 20–40, 40–60, 60–80, 80–100) to test whether the risk score actually separates fraud from clean transactions.

**Analysis / report design**
- **Overview Dashboard page:** KPI card row (Amount, Transactions, Average, Fee, Tax), monthly trend (area chart), status breakdown (donut), amount by customer segment and by state (bar charts), transaction-type breakdown table, gender split (donut), with Year, Dynamic Metric, Occupation, and Merchant Category slicers.
- **Transactions page:** Same KPI/slicer header for consistency, with a detailed transaction-level table (ID, date, customer name, type, status, gender, segment, state, amount, fee, tax) for row-level investigation.
- **Risk & Fraud page:** Added after noticing the model had `is_fraud` and `risk_score` fields that weren't being surfaced anywhere. KPI cards (Fraud Rate %, Fraud Transaction Count), a bar chart comparing average risk score for fraud vs. clean transactions, fraud rate by channel and by transaction type, and a 100%-stacked bar chart showing what share of each risk-score bucket is actually fraud — the chart that proves the risk score is doing real predictive work rather than being a decorative field.

## 3. What insights did my analysis actually uncover?

- **Retail dominates transaction value**, accounting for ~54% of total transaction amount (₹24.8 Cr of ₹45.5 Cr total across the cleaned dataset), more than Premium, SME, Corporate, and Wealth segments combined.
- **Loan EMI and Transfers are the two heaviest transaction types by value** — ₹13.1 Cr and ₹12.1 Cr respectively — together making up over half of total transaction value, while Fee Charges and Refunds are negligible by comparison.
- **Success rate sits at ~85.7%**, with Failed transactions at ~10.2% and Pending at ~4.1% — a healthy but not perfect completion rate worth investigating further at the channel level.
- **Fraud is rare but sharply distinguishable**: only ~1.3% of transactions are flagged as fraud, but flagged transactions carry a dramatically higher average risk score (~83) than clean ones (~36) — meaning the existing risk score is doing real predictive work, not just sitting there as a data field.
- **The risk score cleanly separates fraud from clean transactions at a threshold**: transactions scoring 0–60 are essentially 100% clean, while the 80–100 bucket is almost entirely fraud, with a small transition zone in the 60–80 range. This means a simple risk-score threshold could realistically support automated fraud pre-screening.
- **ATM has the highest fraud rate by channel** (~1.4%), while Auto Debit has the lowest (~1.1%) — a useful signal for where to prioritize fraud-monitoring resources. Fraud rate by transaction type, by contrast, is fairly flat across categories (roughly 0.9%–1.7%), suggesting fraud risk here is more a channel problem than a transaction-type problem.
- **Channel performance is remarkably even** — no single channel dominates transaction value (all sit within a tight ₹64–66 Cr band), but ATM and Auto Debit show the highest raw counts of failed transactions, worth flagging to ops for reliability review.
- **Maharashtra, Karnataka, and Gujarat are the top three states by transaction value**, suggesting these regions could be prioritized for relationship banking or fraud-monitoring resources.
- **July 2025 was the peak month** for transaction value in the dataset, useful as a reference point for seasonal capacity planning.

---

*Data used in this project is synthetic and was generated/sourced for portfolio and learning purposes only — it does not represent real customer or transaction data.*
