# Main Street Deal Analyzer

A Python tool for people buying a small business. Give it a listing's numbers and it tells you what the business is worth, what you can afford to pay, how an SBA loan would be structured, what you'd take home, where the risks are, and what to check before you sign.

No dependencies beyond Python 3.9+. Runs offline. Reports are single HTML files you can open, print, or email.

## What you get

- **Normalized earnings** — rebuilds Seller's Discretionary Earnings (SDE) and adjusted EBITDA from the P&L and add-backs, with trends and a weighted valuation basis.
- **Valuation from five angles** — SDE multiple, adjusted-EBITDA multiple, discounted cash flow, tangible assets, and debt capacity, drawn as a "football field" against the asking price.
- **Price guidance** — a fair-value estimate, a walk-away price, an opening offer, and a plain-English verdict (Priced attractively → Within range → Negotiable → Stretched → Overpriced).
- **SBA 7(a) financing** — sources & uses, loan schedules, the 10% equity-injection rule, full-standby seller notes, and debt-service coverage (DSCR) the way a lender looks at it.
- **Cash to you** — year-by-year free cash flow after your salary, debt and taxes; payback, cash-on-cash, IRR and multiple on your cash, including the exit.
- **Stress tests** — coverage if earnings fall after the handover, and returns across price and exit multiples.
- **Risk review** — a business-quality grade (A–F) from customer concentration, trend, stability, owner dependence, add-back quality, recurring revenue and durability, plus specific red flags.
- **Structuring ideas & a diligence checklist** — what to negotiate, and a prioritized list of documents to request, driven by the deal's own flags.

## Quick start

```bash
cd mainstreet-deal-analyzer

# 60-second screen of a listing (asks a few questions)
python -m mainstreet quick

# Full analysis of an example, with an HTML report opened in your browser
python -m mainstreet analyze examples/hvac_services.json --html --open

# Start your own deal file, edit it, then analyze it
python -m mainstreet new my-deal.json
python -m mainstreet analyze my-deal.json --html

# Rank several deals side by side
python -m mainstreet compare examples/*.json --html

# Plan offers: opening, walk-away, lender max — with coverage and returns at each
python -m mainstreet offer examples/laundromat.json
```

Optional: `pip install -e .` adds a `mainstreet` command so you can type `mainstreet analyze my-deal.json`.

### Useful options

| Option | What it does |
|---|---|
| `--price 1.4M` | Analyze at your offer price instead of the asking price |
| `--set key=value` | Try a what-if without editing the file, e.g. `--set financing.seller_note_pct=0.10` or `--set assumptions.revenue_growth=0` (repeatable) |
| `--html [PATH]` | Write the HTML report (default `<deal>-report.html`) |
| `--json PATH` | Write every number as JSON for your own spreadsheet or code |
| `--brief` | Shorter terminal output |
| `offer --dscr 1.35` | Plan offers against a stricter lender |

## The deal file

A deal is a JSON file. Money can be written `1250000` or `"$1.25M"` or `"450k"`; percents as `0.10` or `"10%"`. Any key starting with `_` is a comment. `python -m mainstreet new` writes a filled-in template, and the `examples/` folder has three complete deals.

If you only have the listing summary, a period can be as short as `{"period": "2025", "revenue": 1300000, "sde": 320000}`. The report will flag that the add-backs haven't been checked.

**Top level**

| Field | Meaning |
|---|---|
| `name`, `industry`, `location`, `description`, `listing_url` | Identification. Run `python -m mainstreet industries` for industry keys (aliases like `hvac` or `marketing agency` work). |
| `asking_price` | Required. |
| `inventory_value`, `inventory_included` | Inventory, and whether it's in the asking price. |
| `ffe_value` | Furniture, fixtures & equipment. |
| `real_estate_value` | Only if real estate is included in the price. |
| `financials` | List of periods (years or TTM), oldest first. |

**Each period in `financials`**

| Field | Meaning |
|---|---|
| `period` (or `year`), `revenue` | Required. |
| `cogs` | Cost of goods sold. |
| `operating_expenses` | All other expenses **except** owner pay, depreciation, amortization, interest and income tax. |
| `owner_compensation` | What the owner paid themselves (salary/draws). |
| `depreciation`, `amortization`, `interest`, `taxes` | Non-cash and financing items; added back automatically. |
| `add_backs` | `{ "owner_vehicle": 9000, ... }` — personal or one-time costs inside operating expenses. |
| `tax_return_revenue` | Revenue on the tax return, to catch books-vs-return gaps. |

**`operations`** (leave out anything you don't know — it's scored neutral and added to the diligence list): `employees`, `years_in_business`, `top_customer_pct`, `top5_customers_pct`, `recurring_revenue_pct`, `owner_hours_per_week`, `owner_holds_key_relationships`, `has_general_manager`, `documented_processes` (`none`/`partial`/`full`), `lease_years_remaining` (including options), `licenses_transferable`, `market_manager_salary`, `seller_transition_months`.

**`financing`** (defaults model a typical SBA acquisition)

| Field | Default | Meaning |
|---|---|---|
| `loan_type` | `sba_7a` | `sba_7a`, `conventional`, or `cash` |
| `buyer_equity_pct` | 0.10 | Your cash, as a share of total project cost |
| `seller_note_pct` | 0.05 | Seller financing, as a share of price |
| `seller_note_full_standby` | true | No payments until the senior loan is repaid |
| `seller_note_standby_years` | loan term | Override the standby period |
| `seller_note_amortization_years`, `seller_note_rate` | 5, 0.06 | Payoff after standby |
| `senior_rate`, `senior_term_years` | 0.0975, 10 | Prime (7.00%) + 2.75%, 10 years |
| `closing_costs_pct` | 0.035 | Lender, guarantee, legal and diligence fees |
| `working_capital` | 0 | Cash you put into the business at closing |

**`assumptions`**

| Field | Default | Meaning |
|---|---|---|
| `valuation_basis` | `weighted` | `weighted` (3/2/1, newest heaviest), `latest`, or `average` |
| `buyer_salary` | 90,000 | What you need to draw to live — lenders subtract it before sizing debt |
| `revenue_growth` | 0.02 | Annual growth after year 1 |
| `maintenance_capex_pct` | 0.01 | Share of revenue reinvested each year |
| `tax_rate` | 0.25 | Combined rate on business profit |
| `hold_years`, `exit_multiple` | 7, industry mid | When and at what SDE multiple you sell |
| `exit_costs_pct`, `capital_gains_rate` | 0.08, 0.20 | Costs and tax when you sell |
| `discount_rate` | 0.25 | Required return for the DCF |
| `target_dscr` | 1.25 | The debt coverage a lender wants |

**`valuation`** (optional): `sde_multiples: [low, mid, high]` and `ebitda_multiples: [low, mid, high]` from real comps, plus a `comps_note`. Use these whenever you have them — the built-in industry table is a rough starting point.

## How the numbers are built

**SDE** = revenue − COGS − operating expenses (i.e. EBITDA after owner pay) + owner pay + add-backs. **Adjusted EBITDA** = SDE − a market salary for a manager: what the business earns if you don't work in it. The valuation basis is a weighted average of the periods (newest counts most) unless you choose otherwise.

**Valuation methods**
- *SDE multiple* — the industry range (or your comps) × basis SDE.
- *Adjusted-EBITDA multiple* — size-banded multiples; a cross-check for larger businesses.
- *DCF* — free cash flow to an owner who pays a manager (adj. EBITDA − capex − tax), plus an exit at the SDE multiple, discounted at your required return ± 5 points.
- *Tangible assets* — liquidation to stated value of equipment, inventory and real estate: the floor.
- *Debt capacity* — the highest price at which cash flow after your salary and upkeep covers loan payments at the target DSCR.

**Price guidance.** The SDE range is positioned by the business-quality score (an A business sits near the top, an F near the bottom). Fair value is two-thirds that risk-adjusted market value and one-third DCF. The **walk-away price** is the lower of fair value and debt capacity; the **opening offer** is about 12% below it. The verdict compares your price with the walk-away price: within it is *Within range*, up to 12% over is *Negotiable* (asking prices usually sit above where deals close), up to 30% over is *Stretched*, beyond that *Overpriced*.

**Financing.** Total project = price + working capital + closing costs. Your cash is `buyer_equity_pct` of that, the seller note is `seller_note_pct` of the price, and the senior loan covers the rest. For SBA loans the tool checks the rules in SOP 50 10 8 (effective June 1, 2025): a 10% minimum equity injection for a change of ownership, with a seller note counting toward it only if it's on full standby for the life of the loan and only for up to half; a $5M 7(a) maximum; 10-year terms without real estate; and the prime + 3% rate cap on loans over $350K.

**DSCR** is measured the lender's way: at today's earnings (no growth), over the senior loan's term, as (SDE − your salary − maintenance capex) ÷ total debt payments. The tightest year is what counts.

**Returns.** Year 1 uses basis earnings as a transition year; growth starts in year 2. Taxes assume an asset purchase, with the price (less inventory and real estate) amortized over 15 years, and losses carried forward. At exit the business sells at the exit multiple of final-year SDE, less selling costs, remaining debt (including any accrued seller-note interest) and capital-gains tax; working capital comes back to you. IRR and multiple are on your cash only — your salary is pay, not return.

**Risk score.** Eight factors are scored 0–100 and weighted: customer concentration (15), earnings trend (15), stability (10), owner dependence (15), add-back quality (10), recurring revenue (10), durability — age, lease, licenses (10) — and debt coverage (15). The *business-quality* grade leaves out debt coverage, because that depends on the price, not the company. Grades: A ≥ 80, B ≥ 68, C ≥ 55, D ≥ 42, F below.

## Keeping it current

Rates and rules live in `mainstreet/constants.py`, dated **October 2026** (prime 7.00%, SBA SOP 50 10 8, BizBuySell's Q2 2026 average of 2.7× SDE). Update `PRIME_RATE` when the Fed moves, and check SBA rules with your lender — they change. Industry multiples live in `mainstreet/industries.py`; override them per deal with real comps.

## Use it from Python

```python
from mainstreet import load_deal, analyze
from mainstreet.report_html import render_html

deal = load_deal("examples/laundromat.json")
a = analyze(deal, price=640_000)
print(a.guidance.verdict, a.guidance.walk_away, a.dscr_min, a.returns.irr)
open("laundromat.html", "w", encoding="utf-8").write(render_html(a))
```

`analyze()` returns a `DealAnalysis` with `earnings`, `methods`, `guidance`, `structure`, `projections`, `returns`, `risk`, `dscr_grid`, `irr_grid` and `structuring`; `a.to_json()` serializes all of it.

## Tests

```bash
python -m unittest discover -s tests
```

`tests/test_mainstreet.py` checks the math, inputs, reports and commands. `tests/test_leakage.py` checks that nothing leaks where it shouldn't:

- **Into the valuation** — the asking price, the price you analyze, and your financing terms don't move what the business is worth (they only move what you can afford).
- **From the future** — growth and exit assumptions don't change historical earnings or what a lender would finance today.
- **Between deals** — results don't depend on analysis order, editing one result doesn't affect the next, and inputs are never modified.
- **Out of your machine** — every command runs with the network blocked, reports load nothing external and contain no local file paths, `_comment` fields never appear in output, deal text is HTML-escaped, and listing links must be `http(s)`.
- **Resources** — memory and open file handles stay flat across repeated runs.

## Limits worth knowing

This is a screening and negotiation tool, not an appraisal, a quality-of-earnings review, or advice. It's only as good as the numbers you enter — the seller's add-backs especially. Taxes are simplified (no depreciation recapture, state differences, or entity-level detail), the tool assumes an asset purchase, and lenders differ in how they calculate coverage (many also count your personal debts). Use it to decide which deals deserve your time and what to offer, then verify everything with a CPA, an attorney and your lender.
