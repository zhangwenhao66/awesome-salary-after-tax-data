# Contributing to Awesome Salary After-Tax Data

Thank you for contributing! This repository serves developers building payroll tools, researchers studying wage inequality, and individuals comparing take-home pay across states.

---

## What We Welcome

### Data Contributions (Highest Value)

- **State tax bracket updates** — State legislatures frequently change income tax rates and brackets. Contributions must cite the official state tax authority (e.g., california.gov/ftb, tax.ny.gov) and the effective tax year.
- **New state JSON entries** — Adding missing or incomplete state data to the tax rate JSON schema with full bracket tables.
- **BLS occupational wage data updates** — BLS releases annual OEWS data each November. Updating the occupation salary benchmark table each year is welcome.
- **Local income tax additions** — New city/county income taxes are occasionally enacted. Add them to the Local Tax Reference section with the official municipal source.
- **Correction of calculation errors** — If any after-tax estimate in the salary milestone tables is incorrect, open an issue immediately with your verified calculation.

### Code Contributions

- New language implementations of the after-tax pay calculation (TypeScript, R, Julia, Go)
- Improved precision in state tax calculations (graduated brackets vs simplified rates)
- Pre-tax deduction handling improvements

### Documentation

- Corrections to state tax treatment descriptions
- New open dataset sources from official government agencies
- Additional paycheck frequency conversion formulas

---

## What We Don't Accept

- Tax avoidance strategies or misrepresentation of income
- Salary data from non-authoritative sources (not BLS, SSA, or peer-reviewed research)
- State tax data that is estimated rather than from the official state tax authority
- Salary milestone estimates without showing the underlying calculation methodology

---

## Data Quality Standards

### State Tax Data

Each state entry must include:
- Source: Official state tax authority URL
- Tax year the data applies to
- Whether the rate is a flat rate or graduated bracket
- Standard deduction or exemption amounts, if applicable
- Any notable local taxes (NYC, Philly, etc.)

### Salary Estimates

All salary milestone estimates must:
- State the filing status (default: Single)
- State the assumption about pre-tax deductions (default: none)
- Show the breakdown: federal + state + FICA separately
- Be reproducible using the provided calculation formula

### Code Examples

New code implementations must:
- Produce the same results as the Python reference implementation for identical inputs
- Include at least 3 test cases spanning different income levels
- Handle edge cases: 0% state rate, states with no income tax, high earners above SS wage base

---

## How to Contribute

### For State Tax Rate Updates

1. Check the current README state data against the official state tax authority website
2. If data is outdated, note the old value, new value, effective date, and source URL
3. Open a PR with the changes and citation

### For Dataset Links

```markdown
- [Dataset Name](https://official-url.gov) — What it contains, update frequency, format (CSV/Excel/API), and why it's useful for after-tax salary analysis.
```

### For Code

1. Fork the repository
2. Branch: `git checkout -b add/typescript-implementation`
3. Add your implementation with full test cases
4. Submit PR with sample output for 3 salary levels across 3 states

---

## Updating Annual Data

Each November/December (after BLS and IRS releases):
1. Update BLS occupation salary table (BLS OEWS released November each year)
2. Update FICA wage base for new year (SSA announces in October)
3. Update federal tax brackets (IRS announces in October/November via Rev. Proc.)
4. Flag any state tax law changes enacted that year

---

## Questions?

Open an issue with the `question` label. For disputed state tax calculations, include the specific state, filing status, income level, and what result you're seeing vs. what you expect.
