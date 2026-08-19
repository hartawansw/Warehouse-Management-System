# Warehouse Management System (WMS)

A single-file, browser-based Warehouse Management System built to run Stock Opname (cycle counting), Stock In/Out, and real-time inventory reporting — with zero backend, zero install, and zero hosting cost.

Built by **Hartawan Sampoerna Wijaya** — Warehouse & Supply Chain professional turned self-taught developer, based on real operational needs from managing physical warehouses.

---

## Why this exists

Most small-to-mid warehouse teams either overpay for enterprise WMS licenses or run stock control through scattered Excel files with no audit trail, no role separation, and no live dashboard. This project is a working alternative: a single HTML file that any warehouse staff can open in a browser — no server, no database setup, no IT ticket required — and get role-based stock counting, in/out tracking, and management-ready reporting out of the box.

## Features

- **Role-Based Access Control** — three access levels (Viewer, Warehouse, Admin) gated by password, each with a distinct permission set enforced on every page (see the in-app Role Permissions matrix).
- **Product Master Data** — add products manually or bulk-upload via CSV, with material code, description, system quantity, category/sub-category, location, and unit price.
- **Stock Opname (Cycle Counting)** — create multiple, independently tracked counting cycles, each with its own history, so counts from different periods never overwrite each other.
- **Barcode / QR Scanning** — scan-mode input for stock counting and stock in/out, plus a built-in QR code generator per product for printable warehouse labels.
- **Stock In/Out Tracking** — dedicated forms for goods received (IN) and goods issued (OUT), with a searchable, filterable transaction history and CSV export.
- **Stock Recap (Rekap Stok)** — auto-calculated stock-on-hand per material, total IN, total OUT, inventory value (unit price × SOH), and discrepancy flags, exportable to CSV and Excel.
- **Stock on Hand Lookup (SOH)** — instant per-material stock check by scan or manual search.
- **Live Dashboards & Analytics** — Chart.js-powered visualizations: category progress, status distribution, variance by category, and in/out movement, with drill-down stat cards (Total Items, Counted, Pending, Discrepancies).
- **Category Management** — admin-configurable category/sub-category tree (e.g. SAP → T Block, PLC, Duct, Ferrule), with CSV bulk upload.
- **Data Backup & Restore** — full data export/import as JSON so the entire dataset can be moved between devices without a server.
- **Offline-Capable** — the entire app can be downloaded as a single HTML file and run fully offline.

## Tech Stack

| Layer | Technology |
|---|---|
| UI / Logic | Vanilla JavaScript, HTML5, CSS3 (no framework) |
| Charts | [Chart.js](https://www.chartjs.org/) |
| CSV parsing | [PapaParse](https://www.papaparse.com/) |
| QR codes | [qrcodejs](https://github.com/davidshimjs/qrcodejs) |
| Data persistence | Browser `localStorage` (JSON) |
| Fonts | DM Sans / DM Mono (Google Fonts) |

No backend, no build step, no package manager — the entire application is one `.html` file (4,600+ lines) that runs by double-clicking it or opening it in any modern browser.

## Getting Started

1. Download `WMS.html` from this repository.
2. Open it in any modern browser (Chrome, Edge, Firefox).
3. Log in with one of the three role passwords (set your own in the Admin panel on first run).
4. Upload your product list via CSV (template downloadable in-app), or add products manually.
5. Start a new Stock Opname cycle and begin counting.

No installation, no server, no internet connection required after the first load.

## Project Documentation

For a deeper look at the system design, data model, and role/permission matrix, see [`PROJECT_DOCUMENTATION.md`](./PROJECT_DOCUMENTATION.md).

For the business case — the operational problem this solves and the impact of solving it — see [`BUSINESS_ANALYSIS.md`](./BUSINESS_ANALYSIS.md).

## Background

This project was built independently, outside of formal job responsibilities, by a warehouse operations professional who taught himself to code in order to close a real gap: the lack of an affordable, easy-to-deploy stock counting and inventory tracking tool for warehouse teams without dedicated IT/ERP budgets.

## Author

**Hartawan Sampoerna Wijaya**
Warehouse & Supply Chain Professional | IT Business Analyst
📧 hrtwn26@gmail.com
