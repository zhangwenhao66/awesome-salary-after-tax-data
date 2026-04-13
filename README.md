# Awesome Salary After-Tax Data [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated collection of open datasets, official wage sources, calculation formulas, and developer tools for computing after-tax take-home pay across all 50 US states — including federal income tax, FICA, and state/local tax breakdowns for common salary levels.

**Last Updated:** April 2026 | Tax Year 2026

> **Disclaimer:** All calculations are estimates based on standard deductions and assumptions. Actual take-home pay depends on individual circumstances including filing status, pre-tax deductions, and local taxes. For precise calculations consult a payroll professional or CPA.

---

## Contents

- [Official Wage & Salary Data Sources](#official-wage--salary-data-sources)
- [After-Tax Calculation Formula](#after-tax-calculation-formula)
- [2026 Federal Tax Parameters](#2026-federal-tax-parameters)
- [All-50-States Tax Rate Reference](#all-50-states-tax-rate-reference)
- [Salary Milestone Estimates (All 50 States)](#salary-milestone-estimates-all-50-states)
- [Occupation Salary Benchmarks](#occupation-salary-benchmarks)
- [Open Datasets (CSV/JSON)](#open-datasets-csvjson)
- [Open-Source Libraries & APIs](#open-source-libraries--apis)
- [Paycheck Frequency Conversions](#paycheck-frequency-conversions)
- [Pre-Tax Deductions Impact](#pre-tax-deductions-impact)
- [Local Tax Reference](#local-tax-reference)
- [Contributing Data](#contributing-data)

---

## Official Wage & Salary Data Sources

- [BLS Occupational Employment and Wage Statistics (OEWS)](https://www.bls.gov/oes/) — Annual salary data by occupation and state; the most authoritative US wage dataset.
- [BLS National Compensation Survey](https://www.bls.gov/ncs/) — Wages and benefits by industry and occupation.
- [Census Bureau American Community Survey (ACS) PUMS](https://www.census.gov/programs-surveys/acs/microdata/access.html) — Individual-level earnings microdata (anonymized).
- [SSA Wage Statistics](https://www.ssa.gov/cgi-bin/netcomp.cgi) — Annual wage distribution from Social Security records; shows median and percentiles.
- [IRS Statistics of Income (SOI)](https://www.irs.gov/statistics/soi-tax-stats-individual-statistical-tables-by-size-of-adjusted-gross-income) — Actual reported AGI distributions.
- [Glassdoor Research](https://www.glassdoor.com/research/) — Real salary reports by role and location (employer-reported + user-submitted).
- [Levels.fyi](https://www.levels.fyi/comp.html) — Tech industry compensation data (base + equity + bonus).

---

## After-Tax Calculation Formula

### Complete After-Tax Pay Model

```python
def after_tax_pay(
    gross_annual: float,
    state: str,
    filing_status: str = "single",
    pre_tax_401k: float = 0,
    pre_tax_hsa: float = 0,
    pre_tax_fsa: float = 0,
    annual_property_not_applicable: bool = True
) -> dict:
    """
    Estimate annual after-tax take-home pay.
    Assumes standard deduction, no itemizing, no dependents.

    Args:
        gross_annual: Gross annual salary
        state: State abbreviation (e.g., "CA", "TX", "NY")
        filing_status: "single" | "married_jointly" | "head_of_household"
        pre_tax_401k: Annual 401(k) contribution (reduces federal + state taxable income)
        pre_tax_hsa: Annual HSA contribution (reduces all income taxes + FICA)
        pre_tax_fsa: Annual FSA contribution (reduces federal + state taxable income)
    """
    # Step 1: Pre-tax deductions
    hsa_deduction = min(pre_tax_hsa, 4_300)  # 2026 individual HSA limit
    fsa_deduction = min(pre_tax_fsa, 3_300)  # 2026 health FSA limit
    k401_deduction = min(pre_tax_401k, 23_500)  # 2026 401(k) limit

    # FICA base: gross - HSA (HSA reduces FICA; 401k/FSA do not for most plans)
    fica_base = gross_annual - hsa_deduction

    # Step 2: FICA
    SS_WAGE_BASE = 176_100
    ss_tax = min(fica_base, SS_WAGE_BASE) * 0.062
    medicare_base_tax = fica_base * 0.0145
    addl_medicare = max(0, fica_base - 200_000) * 0.009
    total_fica = ss_tax + medicare_base_tax + addl_medicare

    # Step 3: Federal income tax base
    federal_taxable_base = gross_annual - hsa_deduction - k401_deduction - fsa_deduction
    standard_deductions = {"single": 15_000, "married_jointly": 30_000, "head_of_household": 22_500}
    std_ded = standard_deductions.get(filing_status, 15_000)
    federal_taxable = max(0, federal_taxable_base - std_ded)

    # Step 4: Federal income tax (single brackets 2026)
    brackets = [
        (11_925, 0.10), (48_475, 0.12), (103_350, 0.22),
        (197_300, 0.24), (250_525, 0.32), (626_350, 0.35), (float('inf'), 0.37)
    ]
    federal_tax = 0
    prev = 0
    for limit, rate in brackets:
        if federal_taxable <= prev:
            break
        federal_tax += (min(federal_taxable, limit) - prev) * rate
        prev = limit

    # Step 5: State income tax (simplified — see STATE_TAX_RATES below)
    state_tax = estimate_state_tax(gross_annual - k401_deduction - fsa_deduction, state, filing_status)

    total_deductions = total_fica + federal_tax + state_tax
    take_home = gross_annual - total_deductions

    return {
        "gross_annual": gross_annual,
        "federal_income_tax": round(federal_tax, 0),
        "social_security_tax": round(ss_tax, 0),
        "medicare_tax": round(medicare_base_tax + addl_medicare, 0),
        "state_income_tax": round(state_tax, 0),
        "total_tax": round(total_deductions, 0),
        "take_home_annual": round(take_home, 0),
        "take_home_monthly": round(take_home / 12, 0),
        "take_home_biweekly": round(take_home / 26, 0),
        "effective_rate": round(total_deductions / gross_annual * 100, 1),
    }
```

---

## 2026 Federal Tax Parameters

### Tax Brackets (Single)

| Taxable Income Range | Rate |
|---|---|
| $0 – $11,925 | 10% |
| $11,925 – $48,475 | 12% |
| $48,475 – $103,350 | 22% |
| $103,350 – $197,300 | 24% |
| $197,300 – $250,525 | 32% |
| $250,525 – $626,350 | 35% |
| Over $626,350 | 37% |

### Tax Brackets (Married Filing Jointly)

| Taxable Income Range | Rate |
|---|---|
| $0 – $23,850 | 10% |
| $23,850 – $96,950 | 12% |
| $96,950 – $206,700 | 22% |
| $206,700 – $394,600 | 24% |
| $394,600 – $501,050 | 32% |
| $501,050 – $751,600 | 35% |
| Over $751,600 | 37% |

### FICA (2026)

| Tax | Rate | Wage Base |
|---|---|---|
| Social Security | 6.2% | $176,100 |
| Medicare | 1.45% | No cap |
| Additional Medicare | 0.9% | Above $200,000 (single) |

> Source: IRS Rev. Proc. 2025-38; SSA COLA announcement October 2025

---

## All-50-States Tax Rate Reference

### States with No Income Tax

| State | Notes |
|---|---|
| AK — Alaska | No income or sales tax |
| FL — Florida | No income tax |
| NV — Nevada | No income tax |
| NH — New Hampshire | No wage income tax (investment income taxed at 3% until 2027) |
| SD — South Dakota | No income tax |
| TN — Tennessee | No income tax |
| TX — Texas | No income tax; high property taxes |
| WA — Washington | No income tax (7% capital gains tax on gains > $262,000) |
| WY — Wyoming | No income tax |

### State Income Tax Top Rates (2026)

| State | Top Marginal Rate | Notes |
|---|---|---|
| CA | 13.3% | On income over $1M; SDI: 1.1% (no cap) |
| HI | 11.0% | On income over $400,000 |
| NJ | 10.75% | On income over $1M |
| OR | 9.9% | On income over $125,000 (single) |
| MN | 9.85% | On income over $183,340 (single) |
| DC | 10.75% | District of Columbia |
| NY | 10.9% | + NYC: 3.876% local for city residents |
| VT | 8.75% | On income over $213,150 |
| IA | 8.53% | Flat rate reform underway |
| WI | 7.65% | On income over $405,550 |
| ME | 7.15% | On income over $59,450 |
| ID | 5.8% | Flat rate (2023 reform) |
| CO | 4.4% | Flat rate |
| IL | 4.95% | Flat rate |
| MI | 4.25% | Flat rate |
| IN | 3.15% | Flat rate |
| PA | 3.07% | Flat rate + local EIT (1–3%) |
| ND | 2.5% | Flat rate (reduced 2024) |
| AZ | 2.5% | Flat rate (Prop 208 surcharge blocked) |

> **Note:** Many states use graduated brackets. Rates shown are top marginal rates. See each state's tax authority for complete bracket tables.

---

## Salary Milestone Estimates (All 50 States)

Estimates below assume: Single filer, standard deduction, no pre-tax deductions, no dependents, 2026 tax year.

### $75,000 Gross Salary — After-Tax Estimates

| State | Federal Tax | State Tax | FICA | Take-Home | Effective Rate |
|---|---|---|---|---|---|
| TX (no state tax) | $9,918 | $0 | $5,738 | $59,344 | 20.9% |
| FL (no state tax) | $9,918 | $0 | $5,738 | $59,344 | 20.9% |
| CA | $9,918 | $4,193 | $5,738 | $55,151 | 26.5% |
| NY | $9,918 | $3,891 | $5,738 | $55,453 | 26.1% |
| WA (no state tax) | $9,918 | $0 | $5,738 | $59,344 | 20.9% |
| IL | $9,918 | $2,888 | $5,738 | $56,456 | 24.7% |
| PA | $9,918 | $1,848 | $5,738 | $57,496 | 23.3% |

### $100,000 Gross Salary — After-Tax Estimates

| State | Federal Tax | State Tax | FICA | Take-Home | Effective Rate |
|---|---|---|---|---|---|
| TX / FL / WA | $14,818 | $0 | $7,650 | $77,532 | 22.5% |
| CA | $14,818 | $6,601 | $7,650 | $70,931 | 29.1% |
| NY | $14,818 | $5,853 | $7,650 | $71,679 | 28.3% |
| NY + NYC | $14,818 | $5,853 | $7,650 | $68,767 | 31.2% |
| IL | $14,818 | $4,028 | $7,650 | $73,504 | 26.5% |
| PA | $14,818 | $2,477 | $7,650 | $75,055 | 24.9% |

### $150,000 Gross Salary — After-Tax Estimates

| State | Federal Tax | State Tax | FICA | Take-Home | Effective Rate |
|---|---|---|---|---|---|
| TX / FL / WA | $26,718 | $0 | $10,114 | $113,168 | 24.6% |
| CA | $26,718 | $12,012 | $10,114 | $101,156 | 32.6% |
| NY | $26,718 | $10,109 | $10,114 | $103,059 | 31.3% |
| IL | $26,718 | $6,178 | $10,114 | $106,990 | 28.7% |
| CO | $26,718 | $5,940 | $10,114 | $107,228 | 28.5% |

> For interactive after-tax calculations with full state breakdowns, [TaxLogic.cc's US Salary Tax Calculator](https://taxlogic.cc/tools/tax/us-salary-tax-calculator) lets you adjust filing status, pre-tax deductions, and compares all 50 states side by side.

---

## Occupation Salary Benchmarks

### BLS Median Annual Wages by Occupation (2024 OEWS Data)

| Occupation | Median Annual | Source |
|---|---|---|
| Software Developer | $130,160 | BLS SOC 15-1252 |
| Registered Nurse | $86,070 | BLS SOC 29-1141 |
| Accountant/Auditor | $79,880 | BLS SOC 13-2011 |
| Teacher (Elementary) | $64,380 | BLS SOC 25-2021 |
| Police Officer | $70,380 | BLS SOC 33-3051 |
| Plumber | $61,550 | BLS SOC 47-2152 |
| Dental Hygienist | $83,450 | BLS SOC 29-2021 |
| Dentist (general) | $175,080 | BLS SOC 29-1021 |
| Lawyer | $145,760 | BLS SOC 23-1011 |
| Physical Therapist | $99,710 | BLS SOC 29-1123 |
| Social Worker | $58,380 | BLS SOC 21-1022 |
| Electrician | $61,590 | BLS SOC 47-2111 |
| Financial Analyst | $99,890 | BLS SOC 13-2051 |

> Source: [BLS OEWS May 2024 National Estimates](https://www.bls.gov/oes/current/oes_nat.htm)

---

## Open Datasets (CSV/JSON)

### Available from Official Sources

```bash
# BLS OEWS State Wage Data — Free download
# https://www.bls.gov/oes/tables.htm
# Format: Excel/CSV, updated annually (May data released November)
# Contains: mean, median, percentile wages by state × occupation

# Census ACS PUMS — Microdata
# https://www.census.gov/programs-surveys/acs/microdata/access.html
# Format: CSV, updated annually
# Contains: Individual earnings, taxes paid, employment status (anonymized)

# SSA Wage Statistics — Publicly available
# https://www.ssa.gov/cgi-bin/netcomp.cgi
# Format: HTML table (scraped to CSV)
# Contains: Wage percentiles, number of workers at each wage level
```

### State Tax Rate Dataset (JSON)

```json
{
  "tax_year": 2026,
  "states": {
    "CA": {
      "type": "graduated",
      "top_rate": 0.133,
      "brackets_single": [
        {"min": 0, "max": 10412, "rate": 0.01},
        {"min": 10412, "max": 24684, "rate": 0.02},
        {"min": 24684, "max": 38959, "rate": 0.04},
        {"min": 38959, "max": 54081, "rate": 0.06},
        {"min": 54081, "max": 68350, "rate": 0.08},
        {"min": 68350, "max": 349137, "rate": 0.093},
        {"min": 349137, "max": 418961, "rate": 0.103},
        {"min": 418961, "max": 698274, "rate": 0.113},
        {"min": 698274, "max": 1000000, "rate": 0.123},
        {"min": 1000000, "max": null, "rate": 0.133}
      ],
      "standard_deduction_single": 5202,
      "additional_taxes": ["SDI: 1.1% no cap"]
    },
    "TX": {
      "type": "none",
      "top_rate": 0,
      "notes": "No state income tax"
    },
    "NY": {
      "type": "graduated",
      "top_rate": 0.109,
      "local_taxes": {
        "NYC": {"type": "graduated", "top_rate": 0.03876}
      }
    }
  }
}
```

---

## Open-Source Libraries & APIs

- [OpenFisca-US](https://github.com/PolicyEngine/policyengine-us) — Python microsimulation of complete US tax-benefit system; handles all 50 states.
- [Tax-Calculator (PSL)](https://github.com/PSLmodels/Tax-Calculator) — Federal income tax model with microdata input; used by CBO and academic researchers.
- [taxee](https://github.com/taxee/taxee-ruby) — Ruby gem for payroll tax calculations; federal + state.
- [payroll (npm)](https://www.npmjs.com/search?q=payroll+tax) — Various Node.js payroll calculation packages.

### Payroll API Services

- [Symmetry Paycheck Calculator API](https://www.symmetry.com/) — Commercial API for precise payroll tax calculations (used by ADP, Gusto).
- [PaycheckCity API](https://www.paycheckcity.com/api/) — Commercial payroll calculator API.

---

## Paycheck Frequency Conversions

```python
def frequency_breakdown(annual_gross: float, annual_take_home: float) -> dict:
    """Convert annual figures to all common paycheck frequencies."""
    return {
        "annual":    {"gross": annual_gross,          "net": annual_take_home},
        "monthly":   {"gross": annual_gross / 12,     "net": annual_take_home / 12},
        "semimonthly":{"gross": annual_gross / 24,    "net": annual_take_home / 24},
        "biweekly":  {"gross": annual_gross / 26,     "net": annual_take_home / 26},
        "weekly":    {"gross": annual_gross / 52,     "net": annual_take_home / 52},
        "daily":     {"gross": annual_gross / 260,    "net": annual_take_home / 260},
        "hourly_40": {"gross": annual_gross / 2080,   "net": annual_take_home / 2080},
    }

# Note: Biweekly (26) ≠ Semimonthly (24)
# Biweekly: paid every 2 weeks → 26 paychecks/year
# Semimonthly: paid twice/month → exactly 24 paychecks/year
```

---

## Pre-Tax Deductions Impact

### How Pre-Tax Benefits Reduce Take-Home vs. Taxes Saved

```python
def pretax_benefit_value(
    gross: float,
    contribution: float,
    marginal_federal_rate: float,
    state_rate: float,
    reduces_fica: bool = False
) -> dict:
    """
    Calculate actual tax savings from a pre-tax contribution.

    reduces_fica: True for HSA (reduces FICA base); False for 401k/FSA (FICA not reduced for most plans)
    """
    federal_savings = contribution * marginal_federal_rate
    state_savings = contribution * state_rate
    fica_savings = contribution * 0.0765 if reduces_fica else 0

    total_savings = federal_savings + state_savings + fica_savings
    net_cost = contribution - total_savings  # Actual reduction in take-home

    return {
        "gross_contribution": contribution,
        "federal_tax_savings": round(federal_savings, 2),
        "state_tax_savings": round(state_savings, 2),
        "fica_savings": round(fica_savings, 2),
        "total_tax_savings": round(total_savings, 2),
        "net_cost_to_paycheck": round(net_cost, 2),
        "effective_discount": round(total_savings / contribution * 100, 1),
    }

# Example: $500/month 401k contribution, 22% federal, 5% state
result = pretax_benefit_value(
    gross=100_000,
    contribution=6_000,  # Annual
    marginal_federal_rate=0.22,
    state_rate=0.05,
    reduces_fica=False
)
# Total tax savings: $1,620 — paycheck only reduced by $4,380, not $6,000
```

---

## Local Tax Reference

### Cities/Counties with Local Income Taxes

| Location | Rate | Notes |
|---|---|---|
| New York City, NY | 3.076%–3.876% | Graduated; residents only |
| Yonkers, NY | 16.75% of state tax | Residents only |
| Philadelphia, PA | 3.75% (residents) / 3.44% (nonresidents) | Earned income |
| Pittsburgh, PA | 3.0% | |
| Columbus, OH | 2.5% | Ohio has 550+ municipal income taxes |
| Cleveland, OH | 2.5% | |
| Detroit, MI | 2.4% (residents) / 1.2% (nonresidents) | |
| Kansas City, MO | 1.0% | Residents + nonresidents who work in KC |
| St. Louis, MO | 1.0% | Earnings tax |
| Denver, CO | $5.75/month (flat) | Occupational Privilege Tax |
| San Francisco, CA | 0.38%–0.691% | Payroll Expense Tax (paid by employer) |

---

## Contributing Data

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on adding state tax bracket data, BLS occupational wage updates, or correcting calculation formulas.

---

## Repo Info

**GitHub Topics:** `salary`, `after-tax`, `payroll`, `us-tax`, `income-tax`, `personal-finance`, `open-data`, `awesome-list`

**About:** A curated collection of after-tax salary datasets, formulas, and official wage sources for all 50 US states — for individuals, developers building payroll tools, and researchers analyzing wage inequality.
