# Missing UPC

A browser-based tool for filling in missing UPCs in product CSVs and producing a Salesforce-ready upload file.

**Live tool:** https://caitlinsc.github.io/missing-upc/

Everything runs in the browser. No data is uploaded anywhere.

## How to use it

1. **Upload your main CSV** — the file with missing UPCs. The tool auto-detects SKU and UPC columns by header name (`SKU`, `Item Number`, `ID`, `UPC`, `UPC Code`, `Barcode`, etc.). Cells that are blank, `0`, or `N/A` count as missing.

2. **(Optional) Filter** — preview rows that already have UPCs, or download just the rows still needing them.

3. **Fill from IMU** — upload the IMU master CSV (Compass export works as-is). The tool builds a SKU → UPC lookup, fills in missing UPCs, and shows a preview. Rows still missing a UPC are highlighted in yellow.

4. **Download Salesforce-ready file** — a two-column CSV (`ID,UPC`) of just the rows that got filled, ready to upload.

UPCs are validated as 8–14 digit numeric strings — junk values like `3650F` are ignored. When the same SKU appears multiple times in IMU, the first valid UPC wins.

## ⚠️ Excel will silently destroy your data

**Do not use Excel to view, edit, or re-save any CSV in this workflow.** It corrupts SKUs in ways you won't notice until after you've uploaded bad data to Salesforce:

- Leading zeros are stripped (`0001476` → `1476`)
- Long numeric SKUs become floats with garbage trailing zeros
- SKUs containing `E` are interpreted as scientific notation (`70781555E104` becomes a 105-digit number)

**Use Google Sheets instead.** It treats CSV cells as text by default and round-trips safely.

If you must use Excel — e.g., the Compass download is `.xlsx` only — import via **Data → From Text/CSV** and explicitly mark the SKU column as **Text** before loading. Double-clicking a CSV will corrupt it before you can do anything.

If your Salesforce import expects field names other than `ID` and `UPC`, rename the headers before uploading.

## Built with

[PapaParse](https://www.papaparse.com/) for CSV parsing. Single-file vanilla JS — no build step.
