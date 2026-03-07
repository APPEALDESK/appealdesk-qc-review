# Batch94 Post-SANITIZE Rerun — 2026-03-07

## CCP-0307-SANITIZE Context

Deployed commit `31d41da` to production. Three files changed:
- `sanitizeTemplateData.js` (new, 159 lines) — input sanitizer at template boundary
- `packetValidator.js` — 11 critical checks, 10 warnings, COMP_COUNT_MATCH deleted, intentionalOverride removed
- `generatePacketCore.js` — 504 lines deleted (retry loops, QG fallback-to-lite, intentionalOverride all gone; sanitizer calls at DU/LP/Full insertion points; DOWNGRADE_LITE no longer destroys comps)

## Top-Line Numbers

| Metric | Count | % |
|--------|-------|---|
| **PASS** | **88** | **94%** |
| FAIL | 6 | 6% |
| TIMEOUT | 0 | 0% |
| Total | 94 | |
| Total time | 1440s (24 min) | |

### Status Breakdown

| Status | Count |
|--------|-------|
| do_not_appeal | 45 |
| completed | 43 |
| quality_gate_failed | 3 |
| refund_pending | 2 |
| failed | 1 |

### Packet Type Breakdown (passing only)

| Type | Count |
|------|-------|
| do_not_appeal | 45 |
| full | 30 |
| assessment | 7 |
| lite | 6 |

### Data Source Breakdown (passing only)

| Source | Count |
|--------|-------|
| reapi | 61 |
| reapi+quantarium | 22 |
| quantarium | 5 |

### Processing Time (passing only)

- Average: 58s
- Min: 46s
- Max: 123s

## Comparison to Batch94 Pre-SANITIZE

| Metric | Pre-SANITIZE (0306-1, 50 addr) | Post-SANITIZE (0307-1, 94 addr) |
|--------|-------------------------------|----------------------------------|
| Pass rate | 49/50 (98%) | 88/94 (94%) |
| Addresses tested | 50 | 94 |
| Avg processing time | ~60s | 58s |
| Timeouts | 0 | 0 |

Note: Direct comparison is limited — address sets differ (50 vs 94 addresses, different selections). The pre-SANITIZE run used a different, smaller address set. The 6 failures in this run are all legitimate gates (agricultural, commercial, AVM divergence), not routing issues.

## Non-Delivering Orders (6 total)

### Refund — Correct Property Type Gates (3)

| # | Address | State | Status | Failure Reason |
|---|---------|-------|--------|----------------|
| 33 | 4435 Georgetown Rd, Lexington, KY 40511 | KY | refund_pending | agricultural_property |
| 45 | 2478 N Farm Rd 231, Strafford, MO 65757 | MO | refund_pending | agricultural_property |
| 58 | 134 Hannold Blvd, Woodbury, NJ 08096 | NJ | failed | Commercial property type: Public Works. Auto-refund issued. |

These 3 are correct gates — non-residential properties that should not receive packets.

### Quality Gate Failed — AVM_COMP_DIVERGENCE (3)

| # | Address | State | Status | Failure Reason |
|---|---------|-------|--------|----------------|
| 44 | 2261 Maple Trl, Merrifield, MN 56465 | MN | quality_gate_failed | AVM_COMP_DIVERGENCE: Comp value ($163,271) is <50% of AVM ($475,000) -- ratio 34% |
| 79 | 244 Hiwassee Dr, Jacksboro, TN 37757 | TN | quality_gate_failed | AVM_COMP_DIVERGENCE: Comp value ($468,747) is <50% of AVM ($1,522,000) -- ratio 31% |
| 81 | 1109 Lindale St, Houston, TX 77022 | TX | quality_gate_failed | AVM_COMP_DIVERGENCE: Comp value ($75,148) is <50% of AVM ($269,000) -- ratio 28% |

These 3 hit the AVM_COMP_DIVERGENCE quality gate — comp median value is less than 50% of the AVM, indicating the comps found don't match the subject property's value tier. All three have large divergence ratios (28-34%).

## Lite Packets (6 total) — Why Each Is Lite

All 6 lite packets are genuine data gaps or zero-comp scenarios that correctly routed to lite:

| # | Address | State | Gate Reason | Why Lite |
|---|---------|-------|-------------|----------|
| 15 | 66 Nugent Loop, Clayton, DE 19938 | DE | provider_data_gap | Insufficient property data from providers |
| 36 | 4109 Pisciotta St, Alexandria, LA 71302 | LA | provider_data_gap | Insufficient property data from providers |
| 41 | 15830 22 Mile Rd, Big Rapids, MI 49307 | MI | provider_data_gap | Insufficient property data from providers |
| 43 | 740 Evergreen Dr, Saint Charles, MN 55972 | MN | provider_data_gap | Insufficient property data from providers |
| 52 | 720 Park Ave, Navassa, NC 28451 | NC | provider_data_gap | Insufficient property data from providers |
| 90 | 9540 Cliffs Trl, Boulder Junction, WI 54512 | WI | provider_data_gap | Insufficient property data from providers |

All 6 are provider_data_gap — genuine cases where REAPI/Quantarium lack sufficient data for the property. These are rural or small-town addresses where provider coverage is thin. No routing issues detected.
