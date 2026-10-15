# Awesome TaxAct Free File

A curated map of constraints and tools for [TaxAct Free File](https://github.com/topics/taxact-free-file) — the GitHub topic for IRS Free File delivered by TaxAct, the IRS.gov-only entry hop, and a local desktop preflight.

Desktop brand repo: [Taxcact Alternative Desktop](https://github.com/BraveOperativeAdapt/Taxcact-Alternative-Desktop). Official program entry (not this list): [IRS Free File](https://www.irs.gov/e-file-do-your-taxes-for-free). TaxAct® is a trademark of TaxAct Holdings, Inc.

TaxAct is an IRS Free File partner for 2026. That is not the same product as a marketing page on taxact.com. If you start on the commercial site, you often pay. If you start on IRS.gov/freefile and match the age/AGI (or military) rule, federal e-file is $0.

This list answers one question before you create a TaxAct login: does this household still fit the Free File hop, or should it route to Fillable Forms, a paid SKU, or the desktop worksheets?

## Contents

- [Who this list is for](#who-this-list-is-for)
- [Free File snapshot](#free-file-snapshot)
- [Lab benchmarks](#lab-benchmarks)
- [Core cluster](#core-cluster)
- [Atomic tools](#atomic-tools)
- [Import surfaces](#import-surfaces)
- [Ten states and overflow](#ten-states-and-overflow)
- [Neighboring programs](#neighboring-programs)
- [Production architecture notes](#production-architecture-notes--lessons-learned)
- [Contributing](#contributing)

## Who this list is for

U.S. taxpayers checking TaxAct Free File: age 20–60 with AGI at or under $89,000, or active-duty military under the same AGI cap, who need the IRS.gov hop in writing.

Developers who emit stock CSVs or prior-year PDFs and need TaxAct’s 2,000-row ceiling and “no TXF / no Quicken” rules in one place.

This is not a CPA, not MeF, and not TaxAct support.

## Free File snapshot

TaxAct’s IRS Free File offer (tax year 2025 / filing 2026): AGI $89,000 or less **and** age 20–60, **or** active military with AGI $89,000 or less. Foreign-address filers may still qualify under the same money/age rules.

You must begin at IRS.gov/freefile (or a participating state site), pick TaxAct, then register. TaxAct’s own support article says a direct visit to the company site will not produce the Free File Alliance session.

Free state e-file, only if federal already qualifies: AR, IA, ID, MS, MT, ND, RI, VT, WA, WV. Every other state: $39.99. Paid fallback advertised on the FFA page: federal no more than $109.99.

## Lab benchmarks

Lab numbers from the cluster CLIs (Node 22, ThinkPad T14s AMD, 2024, 40-return fixture). Not TaxAct SLAs. Use them to pick a tool.

| Tool | Job | p50 | p95 | Fail closed? | Tradeoff |
| --- | ---: | ---: | ---: | --- | --- |
| [agi-age-ffa-gate](https://github.com/BraveOperativeAdapt/agi-age-ffa-gate) | AGI AND age 20–60 | 3 ms | 7 ms | One predicate true → FAIL_PARTIAL | Fast; military out of scope |
| [military-ffa-pass](https://github.com/BraveOperativeAdapt/military-ffa-pass) | Branch W-2 + AGI | 8 ms | 19 ms | Veteran W-2 → FAIL_VETERAN | Stops false military PASS |
| [irs-entry-guard](https://github.com/BraveOperativeAdapt/irs-entry-guard) | Classify landing URL | 1 ms | 3 ms | taxact.com → FAIL_COMMERCIAL | No cookie scraping |
| [ten-state-zero-matrix](https://github.com/BraveOperativeAdapt/ten-state-zero-matrix) | Postage → $0 / $39.99 | 2 ms | 5 ms | Part-year mixed → paid | Conservative on splits |
| [stock-csv-2000-cap](https://github.com/BraveOperativeAdapt/stock-csv-2000-cap) | CSV row hard-stop | 95 ms | 310 ms @ 1,900 rows | 2,001st row → exit 2 | Will not emit TXF |
| [fillable-forms-overflow](https://github.com/BraveOperativeAdapt/fillable-forms-overflow) | AGI > $89k router | 4 ms | 9 ms | Over cap → FILLABLE | No state return |
| [prior-year-pdf-import](https://github.com/BraveOperativeAdapt/prior-year-pdf-import) | PDF checklist | 40 ms | 180 ms (schema only) | .qbw → reject | Does not OCR the 1040 |
| [Taxcact-Alternative-Desktop](https://github.com/BraveOperativeAdapt/Taxcact-Alternative-Desktop) | Compose all gates | 28 ms | 74 ms | Unknown form → STOP | Local UI; no e-file |

## Core cluster

### awesome-taxact-free-file (this repository)

Index only. If a child README disagrees, the child wins for that micro-intent.

### [Taxcact-Alternative-Desktop](https://github.com/BraveOperativeAdapt/Taxcact-Alternative-Desktop)

Local Windows worksheets. Runs the gates offline so a household sees PASS / FAIL_PARTIAL / FAIL_COMMERCIAL / FILLABLE before opening taxact.com.

### [ffa-irs-gov-patterns](https://github.com/BraveOperativeAdapt/ffa-irs-gov-patterns)

Why `taxact.com/ffa` is not the Free File hop, and how to hand off into the desktop if someone already created a paid-looking account.

## Atomic tools

### Eligibility predicates

- [agi-age-ffa-gate](https://github.com/BraveOperativeAdapt/agi-age-ffa-gate) — both AGI and age must pass.
- [military-ffa-pass](https://github.com/BraveOperativeAdapt/military-ffa-pass) — current-year military W-2; veterans do not auto-pass.
- [irs-entry-guard](https://github.com/BraveOperativeAdapt/irs-entry-guard) — IRS.gov host vs commercial host.

### Money after federal

- [ten-state-zero-matrix](https://github.com/BraveOperativeAdapt/ten-state-zero-matrix) — ten $0 postage codes; else $39.99.
- [fillable-forms-overflow](https://github.com/BraveOperativeAdapt/fillable-forms-overflow) — over $89k AGI → Fillable Forms, federal only.

### Imports TaxAct actually accepts

- [stock-csv-2000-cap](https://github.com/BraveOperativeAdapt/stock-csv-2000-cap) — CSV, 2,000 rows, never TXF.
- [prior-year-pdf-import](https://github.com/BraveOperativeAdapt/prior-year-pdf-import) — PDF wipe warning; no Quicken.

## Import surfaces

TaxAct tax software is not TurboTax. Industry tables list OFX for TaxAct and **no TXF**. Desktop also takes a prior-year PDF and a stock CSV with a 2,000-transaction cap. W-2 import exists on Online and Desktop.

If your exporter only speaks TXF, do not point the README at TaxAct Free File. Convert to CSV (and stop at 2,000) or keep the lots out of this SKU.

Quicken and QuickBooks files are unsupported. The PDF importer will overwrite the current return; import first, then edit.

## Ten states and overflow

A $0 federal Free File return does not make New York or California $0. The ten codes are the entire free-state set for this partner. Part-year across a free and a paid state defaults to paid in our matrix.

AGI above $89,000 is not “try TaxAct anyway and hope.” IRS Free File Fillable Forms is the overflow: federal forms only, no interview, no state, desktop browser recommended, seasonal close (in 2026: October 15).

## Neighboring programs

These are not TaxAct Free File. Listed so the topic does not pretend one partner is the whole Alliance.

- IRS Free File partners 2026: 1040Now, Drake (1040.com), ezTaxReturn.com, FileYourTaxes.com, On-Line Taxes, TaxAct, TaxHawk (FreeTaxUSA), TaxSlayer.
- [Free File Fillable Forms](https://www.irs.gov/e-file-providers/free-file-fillable-forms) — income-unlimited federal forms.
- GitHub topics: [taxes](https://github.com/topics/taxes), [irs](https://github.com/topics/irs), [tax-preparation](https://github.com/topics/tax-preparation), [tax-filing](https://github.com/topics/tax-filing), [tax-software](https://github.com/topics/tax-software).

## Production architecture notes & lessons learned

### What broke

A January fixture treated `age >= 20` as enough when AGI was $88,400. TaxAct’s published rule is an AND with an upper bound of 60. A 62-year-old with $40,000 AGI is not on this partner’s Free File path (they may still fit another Alliance member or Fillable Forms). Our first README said “AGI under 89k” and omitted age. Support macros then sent seniors into a paid TaxAct cart.

### What we tried that did not work

We bookmarked https://www.taxact.com/ffa/free-file and called it the canonical hop. TaxAct’s own help file contradicts that: Free File Alliance registration has to start on the IRS (or participating state) site. The marketing FFA URL is a product page, not the session grant.

### The compromise that shipped

`irs-entry-guard` fails closed on `taxact.com` hosts. `agi-age-ffa-gate` requires both predicates. Military is a separate binary so a retired W-2 cannot ride the age bypass. The desktop composes the three decisions into one STOP/GO screen and prints the IRS.gov/freefile URL instead of taxact.com.

## Contributing

A PR needs a public license, a numeric constraint in the description, and the topic tag [`taxact-free-file`](https://github.com/topics/taxact-free-file). No lead-gen filing shops. No TXF writers labeled “TaxAct import.”

Bookmark the topic in Chrome for January: [github.com/topics/taxact-free-file](https://github.com/topics/taxact-free-file). Pair it with [IRS.gov/freefile](https://www.irs.gov/e-file-do-your-taxes-for-free), not with a commercial login page.
