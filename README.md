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
| `Paste_Weekly_Sales` | sales by branch × product per week (or per day): the POS export 商品彙總銷售報表 pasted from column B, store chosen in column A |
| `By_Date_Product_Sales_Stats` | older weekly export blocks (8–14 Nov), kept as history |
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

## Dashboard 5 · Weekly Comparison
Compares two weeks (default: the last two) and shows **where the difference is** and whether it went **up or down**:
- **What changed?** one-line summary: total vs total (▲/▼ %), branch / category / product that went up or down most.
- **Week by week**: every week in the data (W1, W2, W3 … counted from `CONFIG.MONTH_START`, default 1 Nov; weeks start on Sunday, `CONFIG.WEEK_START`), days and branches in the data, net sales, per day, change vs the previous week, and a bar chart.
- **By branch** and **By category** (big category; pick a Category to see its mid categories), with the change in NT$ and %.
- **Products that went up / down** (Top 10 / 20 / 50): sales in both weeks, change in NT$ and %, quantity (e.g. `qty 181 → 399`); `new` = no sales in the first week, `no sales` = none in the second week.
- **Fair comparison:** only branches that have data in **both** weeks are compared (the left-out branches are listed). If the weeks have a different number of days, the dashboard compares the **average per day** automatically (you can force Total or Per day in *Basis*). Net = sales minus returns.
- Needs at least 2 weeks. A week with only some branches pasted is marked “partial”.

### New in tab 5: all weeks side by side
Above the two-week comparison you now also get, for **every week in the data (W1, W2, W3, W4 …)**:
- **By branch** – net sales per week and the % change from each week to the next (▲/▼).
- **By big category (大分類)** – same layout (mid categories when you pick a Category).
- **Products up / down, each week vs the previous one** – one card per step (W1 → W2, W2 → W3, W3 → W4) with the top 5 products that went up and the top 5 that went down, plus how many products went up/down in total.
Weeks with different numbers of days are compared per day automatically. The Branch and Category filters apply to all of it. Nothing to configure: paste the next week in `Paste_Weekly_Sales`, click **Refresh data**.

## Store names in the sheet dropdown
The dashboard reads the **letters** in front of the store label: `MF` = Minghu, `DW` = Dewei, `TN` = Tainan, `DT` = **Daxue**, `FN` = **Hunei**. The Chinese text after the letters does not matter. Change `STORE_CODES` in `CONFIG` if the codes change.

## Paste_Weekly_Sales (how the dashboard reads it)
Paste the export 商品彙總銷售報表 from **column B**, choose the store in **column A** on the first row of each pasted block. The dashboard reads: column A (store, carried down), `商品編號`, `商品名稱`, `銷售淨數量`, `銷售淨額` and the line `資料日期：2026/11/09 - 2026/11/15` (a single day or a range). It works from the pasted values, so the automatic columns on the right are not needed. If the same period and branch are pasted twice, the last one is used. Rows of `By_Date_Product_Sales_Stats` that overlap the same branch and days are ignored.

## Category: pick big, then mid (大分類 → 中分類)
Choose a **Category (大分類)** and a **Mid category (中分類)** dropdown appears with only the mid categories of that big category. When a big category is chosen, the category tables, best sellers, weekly tables, vouchers and stock show its **mid categories** (short names, no big category repeated). The mid category comes from `中分類名稱` in `Stock_Report` (the code `X01001` in front is removed) and from `Category_Map`. Best sellers have a **Products per category** selector (Top 3 / 5 / 10).

## Snapshot (`data/snapshot.json`)
Built from `Recap_November_Completed.xlsx` plus the 16 Nov Daxue export. `Point_Redemptions` only keeps non-personal columns (no names, e-mails, phones or member IDs).
