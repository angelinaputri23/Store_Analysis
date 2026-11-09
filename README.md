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
- `Product_Sales_Stats` has no dates → set `CUR_DAYS` / `CUR_LABEL` in CONFIG to the real period covered (currently assumed 1–8 Nov). Transactions, average transaction value and the weekly sales trend need a daily `Sales_Data` tab (date × branch).
- Rising/falling products compare the current period with October (`Product_Master_Prev` in the snapshot), per day.
- Points data in the snapshot is still October (up to 19 Oct); add a `Points_History` tab to the sheet for November.
- `CUR_DAYS` in CONFIG = number of days covered by `Product_Master` (default 31 = October).
- Point transactions without a store are not forced into the branch ranking.
- Negative stock is treated as 0 (flagged in the KPI "Negative stock records").
