# Saving a Spot

**How long should a paused PostgreSQL CDC pipeline keep its replication slot before a default guard drops it?** An inverse-CDF Monte Carlo decision prototype.

- **Live demo:** https://shriyashekatkar25-byte.github.io/saving-a-spot/
- **APM Lab gallery:** https://shriyashekatkar25-byte.github.io/apm-lab/
- Built by Shriya Shekatkar · UCLA · Stats 102C Monte Carlo Methods

## The launch that inspired it

On Nov 15, 2026, Databricks released a Beta that ingests PostgreSQL change data through one integrated CDC pipeline, with no separate ingestion gateway ([release notes](https://docs.databricks.com/aws/en/release-notes/product/2026/november)). The [setup guide](https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/postgresql-integrated-pipeline) says the pipeline is continuous by default and can be switched to triggered mode on a schedule (60 minutes is the suggested starting interval). It also says slot health is reported only while the pipeline runs, and it recommends a scheduled cleanup notebook so a stopped or deleted pipeline can't leave behind a slot that fills the database disk. The [PostgreSQL connector FAQ](https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/postgresql-faq) notes that a replication slot can make WAL files pile up on the source, and that a slot outliving its retention limits forces a full refresh.

## The PM decision

Should every integrated pipeline get a **default slot guard** that drops the replication slot after **H hours** of absence, and what should H be?

- A short guard protects the customer's production disk, but forces more full re-snapshots.
- A long guard avoids re-snapshots, but lets high-write, small-disk sources fill up.

The prototype simulates pipeline-months, scores 12 guard settings on the same random draws, and writes a director memo for whichever setting minimizes expected cost while checking four guardrails (disk-full ≤ 2 per 1,000 pipeline-months, refreshes ≤ 60, re-snapshot hours ≤ 400, zero data loss). With the default inputs it lands on **18 hours** and points out that the remaining disk-full tail needs a disk-aware check rather than a shorter timer.

## What it shows about product judgment

| Databricks APM skill | Where it shows up |
|---|---|
| Customer problem framing | The DBA's disk and the data engineer's freshness are both customers; the decision card names who gets hurt, and how. |
| Prototyping | A working, seeded simulator with sliders for guard, trigger interval, WAL rate, cost ratio and sample size. |
| SQL / Python | Section 05 fits per-cause outage means and a joint rate × disk table from (illustrative) telemetry, then scores guards in NumPy. |
| Data-driven decisions | Common random numbers, guardrails with thresholds, an expected-cost curve, and a recommendation that updates as inputs change. |
| Clear writing | A memo that rewrites itself, plus a one-sentence VP summary. |

## Methods (Stats 102C)

| Piece of the model | Method |
|---|---|
| Hours a pipeline stays away (per cause) | Ch3 inverse CDF, exponential: `T = −μ·ln(1−U)` |
| Stops per pipeline-month | Ch3 inverse CDF, geometric: `K = ⌊ln U / ln(1−p)⌋` |
| Customer write-rate tier × free-disk tier | Ch3 discrete sampling from a two-variable table |
| Stop cause (pause / credentials / network / forgotten) | Ch3 discrete sampling by partitioning [0,1] |
| Rates per 1,000 pipeline-months, cost curve | Ch1 Monte Carlo averages with a seeded PRNG (mulberry32) |

All inputs are synthetic and chosen to be plausible, not measured.

## Design baseline

Four pins from Shriya's Pinterest home feed, each redrawn from scratch in CSS/SVG/canvas. No pin images are used.

- **Hero** ← ["some sunday magic for ya"](https://www.pinterest.com/pin/917397386620657704/) by kira (credited to Palesora): saturated yellow (#F7CB05), a black condensed grotesque repeated row after row, with the last line in bold.
- **Cards & buttons** ← [angel-wings birthday cake](https://www.pinterest.com/pin/1006343479248199311/) by Ашхен: frosting pink (#EBB0BC / #F7D6DD), cream bead piping borders, script headings, and a small redrawn pair of wings.
- **KPI badges** ← ["LUCKY" condensed type](https://www.pinterest.com/pin/4608097310563621184/) from HiFashion Shop: white field, ultra-condensed black numerals stretched vertically.
- **Simulation room** ← [vintage ladybug "saving you a spot"](https://www.pinterest.com/pin/281615782945027255/) by Veronika: cream print paper (#F2EBD0), tomato red (#D8452C), olive wash (#B4C27A), graphite linework, and spotted ladybug dots. The pin's "saving a spot" caption is also where the title came from.

---

Unofficial concept prototype by Shriya Shekatkar. All data is simulated. Not affiliated with any company.
