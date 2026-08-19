# Project Documentation — Warehouse Management System (WMS)

## 1. Overview

This document describes the technical design, data model, and functional modules of the WMS application. It is intended for anyone evaluating the codebase — recruiters, engineers, or future contributors.

- **Type:** Single-page, single-file web application
- **File:** `WMS.html` (~4,600 lines — HTML, CSS, and JavaScript in one file)
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
│          categories, notes, roles)           │
│                                               │
│              localStorage (JSON)             │
│         key: wms data blob, persisted        │
│         on every mutation                    │
└─────────────────────────────────────────────┘
         │                        │
         ▼                        ▼
   Chart.js (dashboards)   PapaParse (CSV I/O)
   qrcodejs (QR labels)
```

There is no build pipeline. All three layers (structure, style, behavior) live in one file, loaded via CDN dependencies (Chart.js, PapaParse, qrcodejs, Google Fonts) plus inline application code.

## 3. Access Control Model

The application implements three roles, gated by a password prompt at load time. Role state is held in a single `USER_ROLE` variable for the session and re-checked on every page navigation via a permission table (`navRules`) mapping each page to the roles allowed to view it.

| Role | Can view | Can write | Can administer |
|---|---|---|---|
| **Viewer** | Stock Dashboard, Stock Opname, Rekap Stok, SOH, In/Out history | — | — |
| **Warehouse** | All Viewer pages + Product Master, Quick Notes | Add/edit products, record In/Out, run Stock Opname, add Quick Notes | — |
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
A fast, standalone lookup: scan or type a material code to see its current stock level, unit price, total value, and status — alongside a live table of all stock for reference.

### 4.6 Dashboards & Reporting
Two dashboard layers, both powered by Chart.js:
- **Stock Dashboard** — overall inventory health: category-level stock charts and in/out movement.
- **Stock Opname Reports** — cycle-specific analytics: total variance, accuracy rate, items requiring review, inventory value, variance-by-category bar chart, discrepancy detail table, and in/out movement chart. Reports are scoped to the active/latest cycle and exportable (report CSV, all-transactions CSV).

### 4.7 Category Management (Admin)
A configurable two-level category tree (parent category → sub-category, e.g. `SAP → T Block, PLC, Duct, Ferrule`), manageable via manual entry or CSV upload — used to keep Product Master categorization consistent across a large SKU base.

### 4.8 Quick Notes
A lightweight, role-gated notes module (Warehouse/Admin) for operational notes that don't belong in a formal transaction record.

### 4.9 Data Backup & Restore (Admin)
- **Export Full Backup** — serializes the entire application state (products, cycles, transactions, categories, notes, settings) to a downloadable JSON file.
- **Import Backup** — restores that JSON on another device/browser, enabling manual "sync" across machines without a server.
- **Reset All Data** — admin-only, destructive, clears local storage.

### 4.10 QR Code Workflow
Each product can generate a scannable QR code (via `qrcodejs`), downloadable as an image — used to print physical warehouse labels that feed back into the Scan Count, In/Out, and SOH modules.

## 5. Data Model (conceptual)

```
Product {
  materialCode, description, systemQty,
  category, subCategory, location, unitPrice
}

StockOpnameCycle {
  id, name, status (active/closed), createdAt,
  counts: [{ materialCode, systemQty, actualQty, variance, officer, timestamp }]
}

IOTransaction {
  date, type (IN | OUT), materialCode, description,
  qty, note, officer
}

Category {
  parent, subCategories: [ ... ]
}
```

All entities are persisted together as a single JSON blob in `localStorage` and rehydrated on load.

## 6. Design Decisions & Trade-offs

| Decision | Reasoning | Trade-off |
|---|---|---|
| Single HTML file, no framework | Zero install for non-technical warehouse staff; runs from a USB drive or local disk if needed | Larger single file to maintain; no component reuse tooling |
| `localStorage` instead of a database | No server/hosting cost; works fully offline | Data is local to one browser unless manually exported/imported; not a multi-user real-time system |
| Cycle-based Stock Opname | Matches how physical stock counts actually happen (periodic, discrete events) | Slightly more complex state model than a single running tally |
| Client-side CSV/Excel export | No backend needed to generate reports for management | Large datasets are bounded by browser memory/performance |

## 7. Possible Next Steps

- Migrate persistence from `localStorage` to a lightweight backend (e.g. Supabase/Firebase) for true multi-device, multi-user sync.
- Add authentication per named user (not just shared role passwords) for a full audit trail of who counted/moved what.
- Add a native barcode camera scanner (currently keyboard-wedge/manual-input based).
