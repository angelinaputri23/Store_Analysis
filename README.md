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
| `Product_Sales_Report` | chain-wide product totals |
| `Stock_Report` | stock, price, cost, category |
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
