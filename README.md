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
  "https://taxbands.pages.dev/data/2026-27.json"
).then((r) => r.json());

r.incomeTax.personalAllowance                        // 12570
r.incomeTax.bands.scotland                           // 6 bands
r.studentLoans.find((p) => p.key === "plan5").threshold  // 25000
```

### Fetching from a browser

GitHub Pages sends no `Access-Control-Allow-Origin` header, so a cross-origin
`fetch()` against `hadidevlabx.github.io` is blocked by the browser. Use one
of these instead — same files, same bytes:

| URL | CORS | Content-Type |
| --- | --- | --- |
| `https://taxbands.pages.dev/data/…` | ✅ | `application/json` |
| `https://raw.githubusercontent.com/HadiDevLabx/uk-tax-rates/main/data/…` | ✅ | `text/plain` |
| `https://hadidevlabx.github.io/uk-tax-rates/data/…` | ❌ | `application/json` |

Server-side (Node, Python, curl) any of the three is fine — CORS is a browser
rule, not a server one.

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
