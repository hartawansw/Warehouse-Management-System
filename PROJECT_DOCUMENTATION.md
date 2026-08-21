# Project Documentation — Warehouse Management System (WMS)

## 1. Overview

This document describes the technical design, data model, and functional modules of the WMS application. It is intended for anyone evaluating the codebase — recruiters, engineers, or future contributors.

- **Type:** Single-page, single-file web application
- **File:** `WMS.html` (~5,000 lines — HTML, CSS, and JavaScript in one file)
- **Architecture:** Client-side only. No server, no API, no database engine.
- **Persistence:** Browser `localStorage`, serialized as JSON
- **Deployment model:** Static file — can be opened locally, hosted on any static file host (e.g. GitHub Pages), or downloaded and run fully offline

## 2. System Architecture

```
┌─────────────────────────────────────────────┐
│                 WMS.html                     │
│                                               │
│  ┌───────────┐   ┌───────────┐  ┌──────────┐ │
│  │   HTML     │  │    CSS     │  │   JS      │ │
│  │  (layout,  │  │ (design    │  │ (state,   │ │
│  │  pages,    │  │  tokens,   │  │  logic,   │ │
│  │  modals)   │  │  themes)   │  │  render)  │ │
│  └───────────┘   └───────────┘  └──────────┘ │
│                                               │
│              JS Global State                 │
│         (products, cycles, IO log,           │
│    categories, notes, roles, activity log)   │
│                                               │
│              localStorage (JSON)             │
│         key: wms data blob, persisted        │
│         on every mutation                    │
└─────────────────────────────────────────────┘
         │                        │                    │
         ▼                        ▼                    ▼
   Chart.js (dashboards)   PapaParse (CSV I/O)   BarcodeDetector API
   qrcodejs (QR labels)                          + jsQR (camera fallback)
```

There is no build pipeline. All three layers (structure, style, behavior) live in one file, loaded via CDN dependencies (Chart.js, PapaParse, qrcodejs, jsQR, Google Fonts) plus inline application code.

## 3. Access Control Model

The application implements three roles, gated by a password prompt at load time. Role state is held in a single `USER_ROLE` variable for the session and re-checked on every page navigation via a permission table (`navRules`) mapping each page to the roles allowed to view it.

| Role | Can view | Can write | Can administer |
|---|---|---|---|
| **Viewer** | Stock Dashboard, Stock Aging, Stock Opname, Rekap Stok, SOH, In/Out history | — | — |
| **Warehouse** | All Viewer pages + Product Master, Quick Notes, Activity Log | Add/edit products, record In/Out, run Stock Opname, add Quick Notes | — |
| **Admin** | All pages | All Warehouse permissions | Manage passwords, categories, full backup/restore, reset all data |

This is enforced in two places: (1) navigation is blocked at the page-routing level for disallowed roles, and (2) destructive actions (e.g. import backup, reset all data) carry an explicit role check before executing.

## 4. Core Modules

### 4.1 Product Master
Central product catalog: material code, description, system quantity, category, sub-category, warehouse location, and unit price (IDR). Supports manual entry, CSV bulk upload (with a downloadable template), search/filter, and per-product QR code generation.

### 4.2 Stock Opname (Cycle Counting)
The counting engine is built around **cycles** rather than a single running count — each Stock Opname cycle is created, run, and closed independently, preserving its own history. This lets a warehouse compare counts across periods (e.g. monthly cycle counts) without one count overwriting another.

Within an active cycle, staff scan or search a material and log system quantity vs. actual counted quantity; variances are calculated automatically.

### 4.3 Stock In/Out
Two dedicated flows — Barang Masuk (IN) and Barang Keluar (OUT) — each with scan/search input, a confirmation step, and a combined, filterable transaction history (filter by material or transaction type) with CSV export.

### 4.4 Rekap Stok (Stock Recap)
A computed, per-material rollup: total IN, total OUT, current stock on hand, unit price, and total inventory value (unit price × SOH), plus a discrepancy flag. Exportable to CSV and Excel. Backed by stat cards for Total SKU, Total Qty on Hand, Total Inventory Value, and Discrepancy Items — each clickable for drill-down.

### 4.5 Stock on Hand (SOH) Lookup
A fast, standalone lookup: scan or type a material code to see its current stock level, unit price, total value, and status — alongside a live table of all stock for reference. Supports camera-based barcode/QR scanning as an input method (see 4.11).

### 4.6 Dashboards & Reporting
Two dashboard layers, both powered by Chart.js:
- **Stock Dashboard** — overall inventory health: category-level stock charts, in/out movement, and a replenishment warning table driven by per-SKU reorder points (see 4.9).
- **Stock Opname Reports** — cycle-specific analytics: total variance, accuracy rate, items requiring review, inventory value, variance-by-category bar chart, discrepancy detail table, and in/out movement chart. Reports are scoped to the active/latest cycle and exportable (report CSV, all-transactions CSV).

### 4.7 Category Management (Admin)
A configurable two-level category tree (parent category → sub-category, e.g. `SAP → T Block, PLC, Duct, Ferrule`), manageable via manual entry or CSV upload — used to keep Product Master categorization consistent across a large SKU base.

### 4.8 Quick Notes
A lightweight, role-gated notes module (Warehouse/Admin) for operational notes that don't belong in a formal transaction record.

### 4.9 Low Stock / Reorder Point Alerts
Each product can carry a configurable minimum stock threshold (`minStock`). Stock status is derived per-SKU as `ok`, `low`, or `out` by comparing live stock-on-hand against that threshold (falling back to a default threshold of 10 if unset). This status feeds three places:
- A live count badge on the Stock Dashboard sidebar item
- A dedicated Replenishment Warning table (filterable by status/category/search)
- A one-time toast notification on login summarizing how many SKUs currently need attention

This turns reorder monitoring from a manual "check the sheet" habit into something the system surfaces proactively.

### 4.10 Stock Aging Report
Every SKU is classified by how long it has been since its last recorded movement (IN, OUT, or a Stock Opname count), into four buckets:

| Bucket | Age | Meaning |
|---|---|---|
| Fresh | ≤ 30 days | Regular movement |
| Watch | 31–90 days | Starting to slow down |
| Aging | 91–180 days | Slow-moving, needs review |
| Dead Stock | > 180 days, or never moved | Candidate for write-off/liquidation review |

The report is filterable by bucket, category, and search term, and sortable by age — designed to give a supervisor a direct, browsable answer to "what's been sitting here the longest," a question that's normally a manual pivot-table exercise in spreadsheet-based operations.

### 4.11 Barcode / QR Camera Scanning
A camera-based scan mode available on Stock Count, Stock In, Stock Out, and Stock-on-Hand Lookup. On supported browsers it uses the native `BarcodeDetector` API; where that's unavailable, it falls back to a `jsQR`-based canvas decode loop reading frames from the device camera. A detected code auto-fills the relevant input and submits it, removing the need for a dedicated hardware scanner or manual typing on the floor.

### 4.12 Activity Log
A system-wide, timestamped audit trail capturing significant actions: logins, product additions/deletions, stock in/out transactions, cycle creation and completion, CSV imports, and reorder-point changes. Each entry records the action, a human-readable detail string, the acting role, and a timestamp. Searchable and filterable by action type — this is the answerability layer referenced in the Business Analysis doc's "no audit trail" problem statement, extended beyond just Stock Opname counts to cover the full range of system actions.

### 4.13 Dark Mode
A full dark theme, toggled from the sidebar and persisted across sessions via `localStorage`. Implemented through CSS custom-property overrides on `body.dark-mode` rather than duplicated stylesheets, so all pages, cards, tables, and charts stay in sync automatically.

### 4.14 Data Backup & Restore (Admin)
- **Export Full Backup** — serializes the entire application state (products, cycles, transactions, categories, notes, activity log, settings) to a downloadable JSON file.
- **Import Backup** — restores that JSON on another device/browser, enabling manual "sync" across machines without a server.
- **Reset All Data** — admin-only, destructive, clears local storage.

### 4.15 QR Code Workflow
Each product can generate a scannable QR code (via `qrcodejs`), downloadable as an image — used to print physical warehouse labels that feed back into the Scan Count, In/Out, SOH, and camera-scan modules.

## 5. Data Model (conceptual)

```
Product {
  materialCode, description, systemQty,
  category, subCategory, location, unitPrice,
  minStock
}

StockOpnameCycle {
  id, name, status (active/closed), createdAt,
  counts: [{ materialCode, systemQty, actualQty, variance, officer, timestamp }]
}

IOTransaction {
  date, type (IN | OUT), materialCode, description,
  qty, note, officer, timestamp
}

Category {
  parent, subCategories: [ ... ]
}

ActivityLogEntry {
  timestamp, role, icon, action, detail
}
```

All entities are persisted together as a single JSON blob in `localStorage` and rehydrated on load. Stock-on-hand and stock aging are never stored directly — both are computed live from `Product.systemQty`/last count plus the full `IOTransaction` history, which keeps derived values consistent even after edits to historical records.

## 6. Design Decisions & Trade-offs

| Decision | Reasoning | Trade-off |
|---|---|---|
| Single HTML file, no framework | Zero install for non-technical warehouse staff; runs from a USB drive or local disk if needed | Larger single file to maintain; no component reuse tooling |
| `localStorage` instead of a database | No server/hosting cost; works fully offline | Data is local to one browser unless manually exported/imported; not a multi-user real-time system |
| Cycle-based Stock Opname | Matches how physical stock counts actually happen (periodic, discrete events) | Slightly more complex state model than a single running tally |
| Client-side CSV/Excel export | No backend needed to generate reports for management | Large datasets are bounded by browser memory/performance |
| Stock-on-hand & aging always derived, never stored directly | Prevents silent overwrites; every quantity is traceable to a logged transaction | Slightly more computation on each render vs. reading a cached field |
| `BarcodeDetector` API with `jsQR` fallback | Native API is fast and needs no extra payload where supported; fallback keeps the feature working on browsers that lack it | Two code paths to maintain instead of one |

## 7. Possible Next Steps

- Migrate persistence from `localStorage` to a lightweight backend (e.g. Supabase/Firebase) for true multi-device, multi-user sync.
- Add authentication per named user (not just shared role passwords) for a full audit trail of who counted/moved what.
- Add batch/lot tracking with FIFO/FEFO expiry logic.
- Export the Activity Log and Stock Aging Report to PDF/CSV for management reporting.
