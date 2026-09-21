# UK tax rates, as JSON

Income tax bands for England, Wales and Scotland, National Insurance
thresholds and all five student loan plans — one file per tax year, free to
use commercially with attribution.

Every UK payroll project starts by hard-coding the bands into a constants
file, and every April that file is quietly wrong. This is that file, kept
current and versioned.

**Browse it:** <https://hadidevlabx.github.io/uk-tax-rates/>

## Use it

```bash
curl https://hadidevlabx.github.io/uk-tax-rates/data/latest.json
```

Pin to a tax year when a calculation needs to stay reproducible:

```js
const r = await fetch(
  "https://cdn.jsdelivr.net/gh/HadiDevLabx/uk-tax-rates@v1.0.1/data/2026-27.json"
).then((r) => r.json());

r.incomeTax.personalAllowance                        // 12570
r.incomeTax.bands.scotland                           // 6 bands
r.studentLoans.find((p) => p.key === "plan5").threshold  // 25000
```

### Which URL to use

| URL | CORS | Type | Use for |
| --- | --- | --- | --- |
| `cdn.jsdelivr.net/gh/HadiDevLabx/uk-tax-rates@v1.0.1/data/…` | ✅ | `application/json` | **Pinned.** Served `immutable`, cached a year. Reproducible calculations |
| `cdn.jsdelivr.net/gh/HadiDevLabx/uk-tax-rates@main/data/…` | ✅ | `application/json` | Follows the repo. 12-hour edge cache |
| `raw.githubusercontent.com/HadiDevLabx/uk-tax-rates/main/data/…` | ✅ | `text/plain` | No CDN, no cache delay |
| `hadidevlabx.github.io/uk-tax-rates/data/…` | ❌ | `application/json` | Server-side and `curl` only |

GitHub Pages sends no `Access-Control-Allow-Origin` header, so a cross-origin
`fetch()` against `hadidevlabx.github.io` is blocked by the browser. Every
other row sends it.

Pin to a `@tag` if a calculation has to give the same answer next year — those
URLs are served `immutable`. A `@main` URL is fine for a live lookup but takes
up to twelve hours to pick up a change at the edge.

Server-side — Node, Python, curl, anything that is not a browser — every row
works, because CORS is a browser rule and not a server one.

```python
import requests
r = requests.get(
    "https://hadidevlabx.github.io/uk-tax-rates/data/2026-27.json"
).json()
r["nationalInsurance"]["employee"]["primaryThreshold"]  # 12570
```

## Files

| Tax year | File |
| --- | --- |
| 2026/27 *(latest)* | [`data/2026-27.json`](data/2026-27.json) |
| 2025/26 | [`data/2025-26.json`](data/2025-26.json) |
| 2024/25 | [`data/2024-25.json`](data/2024-25.json) |
| Always newest | [`data/latest.json`](data/latest.json) |
| Index of years | [`data/index.json`](data/index.json) |

## What each file contains

| Key | Contains |
| --- | --- |
| `incomeTax` | Personal allowance, the £100,000 taper and its rate, bands by region, standard tax code, Blind Person's Allowance |
| `nationalInsurance` | Employee thresholds and rates including the category B reduced rate, plus employer figures |
| `studentLoans` | All five plans — 1, 2, 4, 5 and Postgraduate — with thresholds and rates |
| `dividends`, `selfEmployed` | Dividend allowance and rates; Class 2 and Class 4 NI |
| `statutoryPayments` | Statutory Maternity Pay and Statutory Sick Pay |
| `childBenefit`, `marriageAllowance`, `taxFreeChildcare`, `redundancy`, `companyCarBik` | The thresholds each of these turns on |

## Each block, applied

Reading a threshold is easy; knowing what it does to a real salary is the part
that takes a while. Each row is the same figure from this dataset worked
through on a live calculator — useful for sanity-checking your own
implementation against a reference.

| Field | Worked example |
| --- | --- |
| `incomeTax.bands` | [Income tax calculator](https://truetakehome.co.uk/income-tax-calculator/) |
| `incomeTax.personalAllowance` | [Take-home pay calculator](https://truetakehome.co.uk/) |
| `incomeTax.paTaperStart` | [100k tax trap calculator](https://truetakehome.co.uk/100k-tax-trap-calculator/) — the 60% marginal band |
| `incomeTax.paTaperRate` | [Adjusted net income calculator](https://truetakehome.co.uk/adjusted-net-income-calculator/) |
| `incomeTax.bands.scotland` | [Scottish income tax calculator](https://truetakehome.co.uk/scottish-income-tax-calculator/) — all six bands |
| `incomeTax.standardTaxCode` | [Tax code checker](https://truetakehome.co.uk/tax-code-checker/) |
| `nationalInsurance.employee` | [National Insurance calculator](https://truetakehome.co.uk/national-insurance-calculator/) |
| `nationalInsurance.employer` | [Employer cost calculator](https://truetakehome.co.uk/employer-cost-calculator/) |
| `studentLoans` | [Student loan repayment calculator](https://truetakehome.co.uk/student-loan-repayment-calculator/) — all five plans |
| `dividends` | [Dividend tax calculator](https://truetakehome.co.uk/dividend-tax-calculator/) |
| `selfEmployed` | [Self-employed tax calculator](https://truetakehome.co.uk/self-employed-tax-calculator/) |
| `childBenefit` | [Child Benefit calculator](https://truetakehome.co.uk/child-benefit-calculator/) — the HICBC taper |
| `marriageAllowance` | [Marriage Allowance calculator](https://truetakehome.co.uk/marriage-allowance-calculator/) |
| `taxFreeChildcare` | [Tax-Free Childcare calculator](https://truetakehome.co.uk/tax-free-childcare-calculator/) |
| `redundancy` | [Redundancy pay calculator](https://truetakehome.co.uk/redundancy-pay-calculator/) |
| `statutoryPayments.maternity` | [Maternity pay calculator](https://truetakehome.co.uk/maternity-pay-calculator/) |
| `statutoryPayments.sickPay` | [Statutory Sick Pay calculator](https://truetakehome.co.uk/statutory-sick-pay-calculator/) |
| `companyCarBik` | [Company car tax calculator](https://truetakehome.co.uk/company-car-tax-calculator/) |

## Scotland is not England with different numbers

Scotland has **six** income tax bands against England's three, starting with a
19% starter rate and topping out at 48%. National Insurance is reserved, so it
does not change. Code that models the UK as one band set is wrong for roughly
8% of UK taxpayers — `bands` is keyed by region for exactly this reason.

## Provenance

Each file carries `meta.primarySource` — the gov.uk or gov.scot page the
figures were read from — and `meta.reviewedOn`, the date that was last
checked.

The figures are generated from the rate tables behind
[True Take-Home](https://truetakehome.co.uk/), a free UK take-home pay
calculator, whose build fails rather than serve a stale tax year. That is what
keeps this from drifting past an April.

For the same figures applied to a salary rather than served as constants, the
calculator is at [truetakehome.co.uk](https://truetakehome.co.uk/), and
[truetakehome.co.uk/rates.json](https://truetakehome.co.uk/rates.json) serves
the current year live from the same source.

## Caveat on the non-year-versioned blocks

`incomeTax`, `nationalInsurance` and `studentLoans` are the figures for the
tax year named in the file. The remaining blocks — statutory payments,
childcare, redundancy, BIK — are **current values** and are not yet
year-versioned. `meta.note` says so in every file. Check `reviewedOn` before
using them for a historical calculation.

## Licence

[CC BY 4.0](LICENSE). Free for commercial use. The attribution line is the
only condition:

```text
Data: True Take-Home (https://truetakehome.co.uk)
```

**No warranty.** These are published figures transcribed carefully and
checked, not tax advice. If you are shipping payroll, verify against HMRC's
own tables first.

## Corrections

Found a figure that is wrong, or a tax year worth adding? Open an issue with
the gov.uk page that shows it and it will be fixed.
