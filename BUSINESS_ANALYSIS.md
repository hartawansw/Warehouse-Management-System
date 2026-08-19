# Business Analysis — Warehouse Management System (WMS)

## 1. Business Problem

Warehouse teams at small-to-mid-sized companies routinely operate stock control through a mix of paper counts, disconnected Excel sheets, and verbal handoffs. This creates several recurring, costly problems:

- **No single source of truth.** System quantity, physical count, and in/out movement live in separate files that are rarely reconciled in real time, so stock discrepancies are discovered late — often only during a formal audit.
- **No audit trail.** Excel-based counting has no built-in log of who counted what, when, or what changed between counts — making variance investigation slow and often inconclusive.
- **No access control.** Anyone with the file can edit system quantities, categories, or historical records, with no separation between "can view," "can transact," and "can administer."
- **High tooling cost for small operations.** Commercial WMS/ERP modules (SAP WM, Oracle WMS, etc.) require licensing, IT infrastructure, and implementation budget that many mid-sized warehouses — especially 3PLs, distributors, or single-site manufacturers — can't justify for their scale.
- **Slow, manual reporting.** Producing a stock opname variance report or an inventory value summary for management typically means hours of manual Excel work, usually the night before a meeting.

This gap — between "too manual" (Excel/paper) and "too expensive/complex" (enterprise WMS) — is where this project sits.

## 2. Who This Is For

- Warehouse or SCM leads running periodic stock opname cycles who need discipline and traceability without an ERP rollout.
- Small distributors or 3PLs with a handful of staff who need role separation (counting staff vs. supervisors vs. admin) but no budget for licensed software.
- Any operation currently doing cycle counts in Excel that wants a lightweight digital step-up before (or instead of) a full WMS/ERP investment.

## 3. Solution & How It Addresses the Problem

| Problem | How the WMS addresses it |
|---|---|
| No single source of truth | All product master data, counts, and in/out transactions live in one connected dataset — Rekap Stok and SOH always reflect the latest state. |
| No audit trail | Every stock opname count and every IN/OUT transaction is timestamped and attributed to an officer; multiple cycles are preserved independently rather than overwritten. |
| No access control | Role-based access (Viewer / Warehouse / Admin) restricts who can transact, who can administer, and who can only view — enforced on every page. |
| High tooling cost | Zero licensing cost, zero hosting cost, zero IT infrastructure — a single HTML file that runs in any browser. |
| Slow manual reporting | Dashboards, variance reports, and CSV/Excel exports are generated instantly from live data, not rebuilt by hand each cycle. |

## 4. Operational Impact (Qualitative)

Based on direct warehouse operations experience, the categories of impact this kind of tool typically delivers are:

- **Faster discrepancy detection** — variances surface as soon as a count is entered, not weeks later during a formal audit.
- **Reduced reconciliation effort** — Rekap Stok auto-calculates stock on hand, total IN/OUT, and inventory value per SKU instead of requiring a manual VLOOKUP-heavy spreadsheet rebuild.
- **Clearer accountability** — because counts and transactions are tied to an officer and a timestamp, disputes over "who changed what" are answerable from the data itself.
- **Lower onboarding friction for new staff** — a browser-based scan-and-search interface is faster to train on than a shared Excel template with hidden formulas.
- **Management-ready reporting on demand** — accuracy rate, variance by category, and inventory value are available at any time, not just when someone has time to build the report.

*(Note: this is a personal/independent project rather than a company-commissioned system, so the figures above are described as directional operational benefits based on the author's warehouse management experience — not audited before/after metrics from a specific deployment.)*

## 5. Why This Matters as a Portfolio Piece

This project demonstrates the intersection of two skill sets that are usually siloed:

1. **Domain expertise** — over 6 years of hands-on warehouse and supply chain operations (SAP WM, Oracle, Odoo across multiple companies), meaning the requirements behind this tool come from lived operational pain points, not a generic tutorial spec.
2. **Technical execution** — the ability to translate that domain knowledge directly into a working, role-secured, data-driven application — the same bridging skill required in an IT Business Analyst role, but carried all the way through to a shipped product rather than stopping at a requirements document.

## 6. Limitations & Honest Scope

- This is a client-side tool using browser `localStorage`, not a multi-user real-time database — it's best suited to a single device/workstation or a manually-synced small team (via the JSON export/import backup feature), not a large multi-site operation.
- It has not been deployed in a production warehouse environment; it is an independent proof-of-concept built to demonstrate both the operational understanding and the technical capability.
- No automated testing suite is included — a natural next step if this were to be extended toward production use.
