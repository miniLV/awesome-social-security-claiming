# Awesome Social Security Claiming Strategies [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of high-quality resources, formulas, research papers, and open-source tools for optimizing Social Security retirement benefit claiming decisions.

**Financial Disclaimer**: This repository is for educational and informational purposes only. Social Security rules are complex and change frequently. Always consult a licensed financial planner or Social Security Administration representative before making claiming decisions. Nothing here constitutes financial or legal advice.

_Last reviewed: April 2026 | Contributions welcome — see [CONTRIBUTING.md](CONTRIBUTING.md)_

---

## Why This Matters

The Social Security claiming decision is arguably the **highest-stakes single financial choice** most Americans make in retirement. Claiming at 62 vs. 70 can mean a lifetime income difference of **$100,000–$250,000+**, depending on longevity, spousal situation, and survivor benefit strategy.

Yet 57% of Americans claim before their Full Retirement Age (FRA), often forfeiting tens of thousands of dollars in guaranteed, inflation-adjusted income.

Key figures (SSA 2025 data):
- ~70 million Americans receive Social Security benefits
- Average retired worker benefit (2026): ~$1,975/month
- Delayed claiming from 62→70 increases monthly benefit by **~77%**
- Break-even age vs. early claiming: typically **78–82** (highly individual)

---

## Contents

- [Official Government Resources](#official-government-resources)
- [Break-Even Analysis Formulas](#break-even-analysis-formulas)
- [Spousal Benefit Strategies](#spousal-benefit-strategies)
- [Survivor Benefit Math](#survivor-benefit-math)
- [Windfall Elimination Provision (WEP) & GPO](#windfall-elimination-provision-wep--gpo)
- [Taxation of Benefits](#taxation-of-benefits)
- [Open-Source Tools & Calculators](#open-source-tools--calculators)
- [Academic Research & Papers](#academic-research--papers)
- [Books & Long-Form Guides](#books--long-form-guides)

---

## Official Government Resources

- [SSA.gov — Retirement Benefits](https://www.ssa.gov/benefits/retirement/) — Official benefit overview, FRA tables, earnings test rules
- [my Social Security Portal](https://www.ssa.gov/myaccount/) — View your personal earnings record and benefit estimates (essential starting point)
- [SSA Retirement Estimator](https://www.ssa.gov/benefits/retirement/estimator.html) — Official benefit estimator based on actual earnings history
- [SSA Publication: When to Start Receiving Retirement Benefits](https://www.ssa.gov/pubs/EN-05-10147.pdf) — Clear overview of the 62/FRA/70 tradeoffs
- [SSA COLA History](https://www.ssa.gov/oact/cola/colaseries.html) — Historical Cost-of-Living Adjustments (1975–present)
- [2026 Social Security Fact Sheet](https://www.ssa.gov/news/press/factsheets/colafacts2026.pdf) — Current year benefit amounts, taxable wage base, earnings limits

---

## Break-Even Analysis Formulas

### Core Break-Even Concept

Break-even is the age at which **total lifetime benefits from delayed claiming equal total lifetime benefits from early claiming**.

```python
# Break-even age calculator
# Assumes no investment of early benefits (pure income comparison)

def break_even_age(
    early_monthly: float,     # Monthly benefit at early age (e.g., 62)
    delayed_monthly: float,   # Monthly benefit at delayed age (e.g., 70)
    early_start_age: int = 62,
    delayed_start_age: int = 70,
) -> float:
    """
    Returns the break-even age.

    Example:
        Early claim at 62: $1,400/month
        Delayed claim at 70: $2,480/month
        Foregone early benefits: 8 years × 12 × $1,400 = $134,400
        Monthly difference: $2,480 - $1,400 = $1,080
        Months to break even: $134,400 / $1,080 = 124.4 months ≈ 10.4 years
        Break-even age: 70 + 10.4 = ~80.4
    """
    foregone_benefits = (delayed_start_age - early_start_age) * 12 * early_monthly
    monthly_gain = delayed_monthly - early_monthly
    if monthly_gain <= 0:
        raise ValueError("Delayed monthly benefit must exceed early monthly benefit")
    months_to_breakeven = foregone_benefits / monthly_gain
    return delayed_start_age + (months_to_breakeven / 12)

# Example
print(f"Break-even age: {break_even_age(1400, 2480, 62, 70):.1f}")
# Output: Break-even age: 80.4
```

### Early Claiming Reduction Schedule

Benefits claimed before FRA are permanently reduced. The reduction rate depends on how many months early:

| Months Before FRA | Reduction per Month | Example (FRA 67) |
|-------------------|--------------------|--------------------|
| First 36 months   | 5/9 of 1% (≈ 0.556%) | Months 37–60 before 67 |
| Beyond 36 months  | 5/12 of 1% (≈ 0.417%) | Months 1–36 before 67 |

```python
def early_reduction_factor(fra_age: int, claim_age_months: int) -> float:
    """
    Calculate reduction factor for early claiming.
    fra_age: Full Retirement Age in years (66, 67, etc.)
    claim_age_months: Claiming age in total months (e.g., 62*12 = 744)
    """
    fra_months = fra_age * 12
    months_early = fra_months - claim_age_months
    if months_early <= 0:
        return 1.0  # No reduction at or after FRA
    reduction = 0.0
    first_36 = min(months_early, 36)
    reduction += first_36 * (5 / 9 / 100)
    beyond_36 = max(0, months_early - 36)
    reduction += beyond_36 * (5 / 12 / 100)
    return 1.0 - reduction

# FRA = 67, claiming at 62 (60 months early)
factor = early_reduction_factor(67, 62 * 12)
print(f"Benefit factor at 62 with FRA=67: {factor:.4f}")  # ≈ 0.7000 (30% reduction)
```

### Delayed Retirement Credits (DRC)

Each year of delay past FRA (up to age 70) earns **8% per year** (for those born 1943 or later):

```python
def delayed_credit_factor(fra_age: int, claim_age: int) -> float:
    """
    Calculate the delayed retirement credit multiplier.
    Maximum delay credit is earned up to age 70.
    """
    years_delayed = max(0, min(claim_age, 70) - fra_age)
    return 1.0 + (years_delayed * 0.08)

# FRA=67, claiming at 70
print(f"Delayed credit at 70 (FRA=67): {delayed_credit_factor(67, 70):.2f}x")  # 1.24x
```

---

## Spousal Benefit Strategies

### Spousal Benefit Basics

A spouse can claim **up to 50%** of the higher earner's PIA (Primary Insurance Amount) at the higher earner's FRA — but **not** delayed credits. The spousal benefit does not increase beyond 50% of the worker's FRA amount, even if the worker delays to 70.

Key rules:
- Spousal benefit requires the primary worker to have filed for their own benefit
- Spousal benefit is reduced if claimed before the spouse's own FRA
- Ex-spouse benefits: available if marriage lasted 10+ years, divorced 2+ years, and both are 62+

```python
def spousal_benefit(
    worker_pia: float,        # Worker's Primary Insurance Amount
    spouse_own_pia: float,    # Spouse's own earned benefit at their FRA
    spouse_fra: int,
    spouse_claim_age: int,
) -> float:
    """
    Calculate the spousal benefit.
    Returns the HIGHER of own benefit or 50% of worker's PIA,
    adjusted for early claiming.
    """
    max_spousal = worker_pia * 0.50
    # Reduce spousal portion for early claiming (separate calculation)
    months_early = (spouse_fra - spouse_claim_age) * 12
    if months_early > 0:
        # Spousal reduction: 25/36 of 1% per month for first 36, 5/12% beyond
        first_36 = min(months_early, 36)
        beyond_36 = max(0, months_early - 36)
        reduction = (first_36 * 25/36/100) + (beyond_36 * 5/12/100)
        spousal_at_claim_age = max_spousal * (1 - reduction)
    else:
        spousal_at_claim_age = max_spousal
    # Spouse receives the GREATER of their own benefit or the spousal benefit
    # (SSA calls the difference the "excess spousal benefit")
    return max(spouse_own_pia, spousal_at_claim_age)
```

### File-and-Suspend (Historical Context)

The "file and suspend" strategy was eliminated by the Bipartisan Budget Act of 2015. Understanding its elimination helps clarify current rules:
- **Before April 2016**: A worker could file and immediately suspend, allowing a spouse to claim spousal benefits while the worker earned delayed credits.
- **After April 2016**: Suspending your benefit suspends all benefits based on your record (including spousal).

### Restricted Application (Still Available for Pre-1954 Birth Years)

Workers born **before January 2, 1954** can still file a "restricted application" for spousal benefits only at FRA, allowing their own benefit to grow with delayed credits. This group is now 72+ (largely past decision point as of 2026), but understanding it is useful for advising older clients.

---

## Survivor Benefit Math

Survivor benefits are often **more valuable** than spousal benefits and are frequently overlooked:

- Surviving spouse can receive **100%** of the deceased worker's benefit (including any delayed credits earned up to the time of death)
- This makes delayed claiming by the higher earner a **life insurance strategy** for the lower-earning spouse

```python
def survivor_benefit_value(
    high_earner_benefit_at_death: float,  # Includes delayed credits at time of death
    survivor_own_benefit: float,
    survivor_life_expectancy_years: float,
) -> dict:
    """
    Estimate the value of survivor benefits vs. own benefit.
    """
    survivor_receives = max(high_earner_benefit_at_death, survivor_own_benefit)
    years_of_survivor_benefit = survivor_life_expectancy_years
    lifetime_survivor_value = survivor_receives * 12 * years_of_survivor_benefit
    return {
        "monthly_survivor_benefit": survivor_receives,
        "estimated_lifetime_value": lifetime_survivor_value,
    }

# Example: High earner dies at 75 with $3,200/month benefit
# Low earner has own benefit of $900/month, expected to live to 88 (13 more years)
result = survivor_benefit_value(3200, 900, 13)
print(f"Monthly survivor benefit: ${result['monthly_survivor_benefit']:,.0f}")
print(f"Estimated lifetime survivor value: ${result['estimated_lifetime_value']:,.0f}")
# Output: $3,200/month | $499,200 lifetime
```

**Key insight**: If the higher earner delays to 70 and achieves a $3,200 benefit instead of $2,100 at 62, the survivor's monthly income increases by $1,100/month — potentially for 15–20 years.

---

## Windfall Elimination Provision (WEP) & GPO

Applies to workers who receive a pension from a job **not covered by Social Security** (e.g., many state/local government employees, some foreign pensions).

### WEP Formula

WEP reduces the Social Security benefit formula for those with fewer than 30 years of "substantial" Social Security-covered earnings:

```python
# 2026 WEP maximum reduction: $587/month (indexed annually)
# Substantial earnings threshold 2026: $31,275/year

def wep_reduction(
    years_substantial_ss_earnings: int,
    primary_insurance_amount: float,
    max_wep_reduction_2026: float = 587.0,
) -> float:
    """
    Estimate WEP reduction to Social Security benefit.
    Reduction phases out with 21-29 years of substantial earnings;
    fully eliminated at 30+ years.
    """
    if years_substantial_ss_earnings >= 30:
        return 0.0
    elif years_substantial_ss_earnings >= 21:
        phase_out_factor = 1.0 - ((years_substantial_ss_earnings - 20) * 0.10)
        return max_wep_reduction_2026 * phase_out_factor
    else:
        return max_wep_reduction_2026

# Teacher with 25 years SS-covered work (side jobs, etc.)
wep = wep_reduction(25, 1800)
print(f"WEP reduction: ${wep:.0f}/month")  # $293.50/month reduction
```

- [SSA WEP Factsheet](https://www.ssa.gov/pubs/EN-05-10045.pdf)
- [WEP Online Calculator (SSA)](https://www.ssa.gov/benefits/retirement/planner/wep-out.html)

---

## Taxation of Benefits

Up to 85% of Social Security benefits may be taxable depending on "combined income" (AGI + non-taxable interest + ½ of SS benefits):

| Combined Income (Single) | % of Benefits Taxable |
|--------------------------|----------------------|
| Below $25,000            | 0%                   |
| $25,000 – $34,000        | Up to 50%            |
| Above $34,000            | Up to 85%            |

| Combined Income (MFJ)    | % of Benefits Taxable |
|--------------------------|----------------------|
| Below $32,000            | 0%                   |
| $32,000 – $44,000        | Up to 50%            |
| Above $44,000            | Up to 85%            |

Note: These thresholds are **not indexed for inflation** and have not changed since 1993, meaning more retirees pay tax on benefits each year.

```python
def taxable_ss_benefits(
    agi: float,
    non_taxable_interest: float,
    annual_ss_benefit: float,
    filing_status: str = "single",  # "single" or "mfj"
) -> float:
    """
    Estimate taxable portion of Social Security benefits.
    Returns the taxable amount (not the tax itself).
    """
    combined_income = agi + non_taxable_interest + (annual_ss_benefit * 0.5)
    thresholds = {
        "single": (25000, 34000),
        "mfj": (32000, 44000),
    }
    low, high = thresholds[filing_status]
    if combined_income <= low:
        return 0.0
    elif combined_income <= high:
        return min(annual_ss_benefit * 0.50, (combined_income - low) * 0.50)
    else:
        tier1 = min(annual_ss_benefit * 0.50, (high - low) * 0.50)
        tier2 = min(annual_ss_benefit * 0.85 - tier1, (combined_income - high) * 0.85)
        return min(annual_ss_benefit * 0.85, tier1 + tier2)
```

---

## Open-Source Tools & Calculators

- [opensocialsecurity.com](https://opensocialsecurity.com/) — Free, open-source optimizer by Mike Piper (author of "Social Security Made Simple"); maximizes household lifetime benefits with Roth conversion coordination
- [Maximize My Social Security (Kotlikoff)](https://maximizemysocialsecurity.com/) — Commercial but highly regarded academic-backed tool ($50 one-time)
- [SSA ANYPIA Calculator](https://www.ssa.gov/oact/anypia/anypia.html) — Official SSA tool for computing exact PIA from earnings history
- [i-ORP (Optimal Retirement Planner)](http://www.i-orp.com/) — Integrates SS claiming with Roth conversion, RMD, and tax optimization
- [Bogleheads Social Security Spreadsheet](https://www.bogleheads.org/wiki/Social_Security) — Community-maintained spreadsheet for break-even and spousal analysis

---

## Academic Research & Papers

- **Shoven & Slavov (2014)** — ["Does It Pay to Delay Social Security?"](https://www.nber.org/papers/w17666) — NBER Working Paper; foundational break-even analysis showing delay is beneficial for most married couples
- **Munnell & Sass (Boston College CRR)** — ["The Government's Redesigned Reverse Mortgage Program"](https://crr.bc.edu/briefs/the-social-security-claiming-decision/) — Comprehensive claiming research; finds average American leaves $182,000 on the table by claiming early
- **Meyer & Reichenstein (2010)** — ["Social Security: When to Start Benefits and How to Minimize Longevity Risk"](https://www.cfapubs.org/doi/abs/10.2469/faj.v66.n4.6) — CFA Institute analysis on optimal delay strategies
- **SSA Actuarial Studies** — [SSA Actuarial Publications](https://www.ssa.gov/OACT/NOTES/actstudies.html) — Longevity tables and mortality projections used in break-even analysis

---

## Books & Long-Form Guides

- **"Social Security Made Simple" — Mike Piper** — Best single-volume plain-English guide; updated annually; highly recommended as first read
- **"Get What's Yours" — Laurence Kotlikoff, Philip Moeller, Paul Solman** — Comprehensive strategies for complex situations (divorced spouses, government workers, disabled)
- **Bogleheads Wiki: Social Security** — [wiki.bogleheads.org/wiki/Social_Security](https://www.bogleheads.org/wiki/Social_Security) — Community-maintained, peer-reviewed, no ads
- **SSA Publication 05-10035** — [Understanding the Benefits](https://www.ssa.gov/pubs/EN-05-10035.pdf) — Official 60-page comprehensive guide

---

## Related Tools

For salary-to-net-pay calculations and broader financial planning context, [TaxLogic.cc](https://taxlogic.cc) offers free after-tax income calculators covering all 50 US states.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). PRs adding new calculators, corrected formulas, or updated official resources are welcome. Please cite primary sources (IRS publications, SSA.gov, peer-reviewed research).

---

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](https://creativecommons.org/publicdomain/zero/1.0/)
