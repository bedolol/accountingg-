# accounting/SOURCES.md

Every number used in the accounting spec is recorded here with where it came from. Nothing goes into the spec or code unless it has an entry below and the source was opened.

Status key: **VERIFIED** = value read from the primary source by us. **OPEN** = not yet sourced.

---

## 1. PUE (power usage effectiveness)

**Status: VERIFIED (values below). Re-check the CSV yourself before freezing (see "Still to do").**

### Primary sources (both opened on Mon Nov 2, 2026)

1. AWS, "AWS Cloud" page, Amazon Sustainability, section "Power usage effectiveness" and the table "AWS Region PUE and WUE":
   https://sustainability.aboutamazon.com/products-services/aws-cloud
   The page links a downloadable CSV of the same data:
   https://sustainability.aboutamazon.com/content/dam/sustainability-marketing-site/12-the-cloud/aws-pue-wue.csv
   (not downloaded by us yet)
2. AWS, "Power Usage Effectiveness (PUE) Methodology", last updated Dec 3, 2024:
   https://sustainability.aboutamazon.com/pue-methodology.pdf

### What the source says (definition)

- PUE = total facility energy (kWh) / IT energy (kWh).
- Measured with power meters, following ISO/IEC 30134-2:2016 and EN 50600-4-2.
- Values on the page are average PUE for the data centers of a region over Jan 1 to Dec 31 of each year.

### Values for the four proposed regions (not yet confirmed by the team)

| Our region | AWS table name | PUE 2022 | PUE 2023 | PUE 2024 | PUE 2025 |
|---|---|---|---|---|---|
| us-east-1 | U.S. East (Northern Virginia) | 1.16 | 1.15 | 1.15 | 1.15 |
| us-west-1 | U.S. West (Northern California) | 1.17 | 1.17 | 1.18 | 1.18 |
| eu-central-1 | Europe (Frankfurt) | 1.32 | 1.33 | 1.35 | 1.24 |
| eu-north-1 | Europe (Stockholm) | 1.12 | 1.12 | 1.10 | 1.09 |

Other reference points from the same page (2025): global average 1.14, North America 1.14, Europe 1.11.

The mapping from region code to the AWS table name is my reading (us-east-1 = N. Virginia, us-west-1 = N. California, eu-central-1 = Frankfurt, eu-north-1 = Stockholm). Confirm it against the AWS region list.

### Proposed choice (for the team to confirm)

- Use the **per-region PUE for the most recent full year (2025)** as a constant per region, e.g. us-east-1 = 1.15, eu-north-1 = 1.09.
- Store the value actually used in every log record (`pue_used`), so results can be recomputed.
- Report a sensitivity check using the single global value (1.14) to show the conclusions do not hinge on the regional PUE.

Reasons: a single global PUE would hide a real difference between regions (1.09 in Stockholm versus 1.24 in Frankfurt and 1.18 in N. California), and that difference is on the same order as some of the carbon differences we are trying to measure.

### Is the Caribou value (1.11) right for us?

No, not as a default.

- 1.11 is not the value for any of our four regions in the AWS table. It does equal AWS's **Europe-wide** average for 2022 to 2025, but I have not opened Caribou, so I do not know where Caribou got its number. Do not assume they are the same source.
- Using 1.11 everywhere would understate us-east-1 and us-west-1 and overstate eu-north-1.

### Limits you must state in the report

- AWS publishes PUE per region as a facility-level annual average across all workloads in the data centers. **I found no EC2-specific or instance-type-specific PUE.** PUE is a facility metric, so it is applied the same way to any service, but this is an assumption we make, not something AWS states for EC2.
- It is an annual average, so it does not vary by hour or by load. This matches the brief's validity threat ("PUE is an assumed value").
- Year-to-year changes can be large (Frankfurt 1.35 in 2024 versus 1.24 in 2025), so name the year used.
- The values are AWS's own reported figures, not independently audited by us.

### Still to do

- Download the CSV and commit a dated snapshot, so the numbers are frozen in the repo.
- Decide which year matches the test period once the protocol is frozen (week 2).
- If the region set changes, add the new regions' rows here.

---

## 2. EC2 power coefficients

**Status: OPEN.** Not yet sourced. Waiting for the instance types to be agreed with the spot person.

## 3. Transfer energy / carbon (best and worst case)

**Status: OPEN.** Not yet sourced.
