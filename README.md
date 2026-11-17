# Xmart Store Dashboard

One `index.html` (no build, no dependencies) + `data/snapshot.json` (October 2026 data as a fallback).
Three dashboards: **Store Sales Performance**, **Redeemed Points**, **Stock & Promo Planning**.
The **⟳ Refresh data** button pulls the latest data from Google Sheets.

## 1. Host it on GitHub Pages
1. Create a repo and upload `index.html` and the `data/` folder.
2. Settings → Pages → Branch `main` / root → Save.
3. Your dashboard will be at `https://<username>.github.io/<repo>/`.

## 2. Connect it to Google Sheets
1. Upload `Recap_October.xlsx` to Google Drive → open with Google Sheets (or use your original Google Sheet).
2. Tab names (November layout, already preset to your sheet): `Product_Sales_Stats` (branch × product), `Product_Sales_Report` (chain-wide product totals), `Stock_Report`. The older October names (`Sales_Data`, `Points_History`, `Product_Master`, `Stock_Data`) still work.
3. Click **Share → General access → Anyone with the link → Viewer**.
4. Copy the Sheet ID from the URL: `https://docs.google.com/spreadsheets/d/<SHEET_ID>/edit`
5. In `index.html`, find `CONFIG` and paste it: `SHEET_ID:'<SHEET_ID>'`. Commit the change.
6. Open the dashboard and click **Refresh data**. The status line under the title shows "Source: Google Sheets (live)". If it says "Could not read…", re-check steps 2–3.

From now on: update the data in the sheet, then click Refresh. No need to touch GitHub again.

### Optional tabs (if missing, the dashboard uses the snapshot)
- `Category_Map`: columns `商品編號 | 大分類 | 中分類` (from sheet 基本-1001 in 全店.xlsx)
- `Product_Master_Prev`: same layout as `Product_Master`, for the comparison period (rising/falling products). Set `PREV_DAYS` in CONFIG.
- **`Product_Sales_Weekly`** – enables weekly best sellers by branch and category. Columns:
  `Week_Start | Store | 商品編號 | 商品名稱 | 銷售淨數量 | 銷售淨額`
  (one row per product, per branch, per week; any date within the week works)

## Privacy
`Points_History` contains member emails, phones and names. The snapshot in this repo has those columns removed.
If the live sheet is shared as "anyone with the link", anyone with the link can read them – better to create a dashboard-only tab/sheet without personal columns, and/or use a private repo.

## Data notes
- November layout: branch sales, best/low sellers and stock cover are calculated **per branch** from `Product_Sales_Stats`. Store names are normalised (Minghu, Dewei, Hunei, Daxue, Tainan) – edit `STORE_ALIAS` in CONFIG if names change.
- `Product_Sales_Stats` has no dates → set `CUR_DAYS` / `CUR_LABEL` in CONFIG to the real period covered (currently assumed 1–9 Nov, matching the last points transaction on 9 Nov). Transactions, average transaction value and the weekly sales trend need a daily `Sales_Data` tab (date × branch).
- Rising/falling products compare the current period with October (`Product_Master_Prev` in the snapshot), per day.
- Points: tab ``Point_Redemptions` or `Points_Redemptions` (old name `Points_History` also works). The snapshot contains October + November 1–9 so the monthly trend works; in the live sheet, keep appending rows to that tab.
- `CUR_DAYS` in CONFIG = number of days covered by `Product_Master` (default 31 = October).
- Point transactions without a store are not forced into the branch ranking.
- Negative stock is treated as 0 (flagged in the KPI "Negative stock records").

## Promo Calculator (tab 4)
Office-funded promo calculator. Store still receives the **normal price** per unit; office pays the difference.
- Office cost = (normal price − promo price − supplier support/unit) × promo units
- Margin/unit during promo = promo price + supplier support − cost (HPP 成本, taken from Stock_Report)
- Net impact = promo units × margin during promo − baseline units × normal margin
- Break-even units = baseline units × normal margin ÷ margin during promo (never, if promo price + support ≤ cost)
Add products by code/name or with "Saran: stok berlebih". Entries are kept in the browser (localStorage).

### Bundle calculator (B)
For add-on deals such as "spend NT$199, add NT$39". Add the items inside one gift set; the page shows the set's normal value, our cost, what the office pays per bundle (normal value − add-on price − supplier support) and profit per bundle. "Load poster example" pre-fills the Christmas poster items (check the matches).

## Sheet tabs (final names)
| Tab | Purpose |
|---|---|
| `Product_Sales_Stats` | branch × product sales |
| `By_Date_Product_Sales_Stats` | branch × product sales **per week** (POS export with a `Date Range` column) |
| `Product_Sales_Report` | chain-wide product totals |
| `Stock_Report` | stock, price, cost, **大分類名稱 and 中分類名稱** (big and mid category) |
| `Point_Redemptions` | points log (points, LELE, vouchers) – no store info |
| `Voucher_Redemptions_By_Store` | vouchers used at the till per branch (old name `Copy of Point_Redemptions` also works) |

To keep `Voucher_Redemptions_By_Store` up to date automatically, put this in cell A1 of that tab:
`=QUERY(Product_Sales_Stats!A:J,"select A,B,C,D,G,H,I,J where G > 0",1)`

### Buy X get Y free (C)
Pick the "buy" product and (optionally) a different "free" product; leave the free product empty for buy 1 get 1. The page shows what the customer pays, the normal value of everything handed out, what the office pays for the free items (store still receives the normal price), profit per set, profit change vs no promo and break-even sets.

## Redeemed Points
1. **Voucher redemption by branch** (from tab `Voucher_Redemptions_By_Store`): vouchers used per branch, category and product, with *What we pay* (unit cost 成本 × vouchers) and *Sales value given away* (selling price 售價 × vouchers). The Branch and Category filters work on this part.
2. **Points log** (from tab `Point_Redemptions`): total points, trends, LELE, telecom share, held/reversed. It has no store info, so it is shown for all branches together.

Office = the company; branches are company-owned (no franchise), so no money moves between office and stores. A discount is simply sales we give up (*marketing budget needed*).

## Upload a file instead of (or on top of) Google Sheets
Click **⬆ Upload file** (or drag a file onto the page). Accepts `.xlsx` / `.xls` / `.csv`.
- Each sheet is recognised by its **name** (e.g. `Product_Sales_Stats`, `Stock_Report`, `Point_Redemptions`, `Copy of Point_Redemptions`) or, if the name is different, by its **column headers**. Unrecognised sheets are skipped and listed.
- Uploaded tables **replace** the same tab from Google Sheets / the snapshot until you click **✕ Clear uploads**. Refresh data keeps your uploads on top.
- A sheet you upload replaces that whole table (it is not merged with older rows). Uploads are kept only while the page is open.
- `.xlsx` needs the SheetJS reader, which loads from cdnjs the first time (internet needed). `.csv` works fully offline – one file per table, named like the tab.

**📄 Open Google Sheet** (header) opens the sheet set in `CONFIG.SHEET_ID` in a new tab; the button is hidden when no sheet ID is set.

## Category level: 大分類 (big) ↔ 中分類 (mid)
Use the **Category level** switch in the top bar. Everything that is split by category follows it: the Category filters, the category tables, best sellers by category and the category × week table (tab 1), vouchers and points by category (tab 2), stock by category (tab 3). In *Mid* mode categories are shown as `big › mid` (e.g. `冷凍冰品 › 冰淇淋 Ice cream`).
- The mid category comes from the column **中分類名稱** in `Stock_Report` (a leading code like `X01001` is removed for display). Products that are not in `Stock_Report` use `Category_Map` (`商品編號 | 大分類 | 中分類`); otherwise they show as `(Uncategorized)`.
- *Best sellers by category* (tab 1) has a **Products per category** selector (Top 3 / 5 / 10). It follows the Branch, Category and Week filters, so you can see e.g. the top 5 products of every mid category in one branch for one week.
- In tab 2, the points log is matched to categories by product name, so a few items stay as `Other (LELE)` / `Other (voucher)`.

## Weekly sales by category (tab `By_Date_Product_Sales_Stats`)
Paste the POS weekly export (one block per branch) into this tab. Columns used: `Date Range` (e.g. `11/8/2026-11/14/2026`, month/day/year), `Store` (e.g. `30002 Minghu`), `商品編號`, `商品名稱`, `銷售淨數量`, `銷售淨額`.
- `Date Range` and `Store` only need to be filled on the **first row** of each block; the dashboard fills them down. The `合計` (total) rows, empty rows and repeated header rows are skipped.
- Add more branches and more weeks by pasting more blocks below. Each week = its own date range. If the same week/branch/product appears twice, the last one is used (no double counting).
- In *Store Sales Performance* a **Week** filter appears at the top. Pick a week to see KPIs, branch comparison, best sellers and rising/falling products for that week. A **📅 Category sales per week** section shows category × week (net sales or net items), per branch, with a chart.
- A week only contains the branches that are in the tab; the dashboard lists the missing ones. When a week has fewer branches than the chain, rising/falling products compare against October scaled to the same number of branches (an estimate).
- Net = sales minus returns. Member purchases and transactions are not in this export, so they show “-” for a week.
- Upload works too: a sheet named `By_Date_Product_Sales_Stats` (or any sheet with the `Date Range` + `Store` + `銷售淨額` columns) is recognised.

## Snapshot (`data/snapshot.json`)
Built from `Recap_November_Completed.xlsx`. `Point_Redemptions` only keeps non-personal columns (transaction id, points, type, message, date, source, store, metadata): no names, e-mails, phones or member IDs. `Stock_Report` keeps the columns the dashboard uses.
