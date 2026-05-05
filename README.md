# Missing UPC Tool

A browser-based tool for finding and filling in missing UPCs in product CSVs, then producing a Salesforce-ready upload file.

## What it does

The tool has two modes that share an uploaded CSV:

1. **Filter** — preview which rows already have a UPC, or remove those rows and download what's left (useful for working on the gaps).
2. **Fill from IMU** — upload a master IMU export, look up each missing UPC by SKU, and generate a `salesforce-upc-update.csv` with just the rows that got filled. This is the file you upload to Salesforce.

Everything runs in the browser. No data is uploaded anywhere.

## How to use it

### Setup

Open `index.html` in a browser. That's it — no install, no server.

### Step 1: Upload your main CSV

This is the file with missing UPCs you want to fill in. The tool needs:

- A column for the SKU. The tool recognizes headers named `SKU`, `Item Number`, `Item #`, `Item Code`, `Product ID`, `Product Number`, `Style Number`, `Part Number`, and similar variants. As a fallback it'll accept generic names like `ID` or `Item` if no specific match is found.
- A column for the UPC. The tool recognizes `UPC`, `UPC Code`, `UPC Number`, `Barcode`, `Bar Code`, `GTIN`, `GTIN-12`, `GTIN-13`, and similar.

A cell is treated as a missing UPC if it's blank, `0`, or `N/A`.

### Step 2: Filter (optional)

- **Preview rows with UPC** — shows which rows already have one. Useful sanity check.
- **Remove rows with UPC and download** — outputs a CSV of just the rows still needing UPCs. Useful if you want to hand a smaller list to someone.

### Step 3: Fill missing UPCs from IMU

Upload the IMU master CSV. It needs a SKU column and a UPC column with the same naming rules as above (the IMU export from Compass uses `Item Number` and `UPC Code`, both of which work).

Click **Fill UPCs and preview**. The tool will:

- Build a SKU → UPC lookup from the IMU file
- For each row in your main CSV that's missing a UPC, look up the SKU and fill it in
- Show a preview of the full file with newly-filled UPCs in place
- Highlight in yellow any rows that are still missing a UPC (SKU not found in IMU, or IMU's UPC was blank/invalid)
- Show a button: **Download Salesforce-ready file (N rows)** — only the rows that actually got filled, with just `ID,UPC` columns

The tool only accepts UPCs that look real (8–14 digits, all numeric). Junk values like `3650F` or vendor part numbers that ended up in the UPC column are silently ignored.

If the same SKU appears multiple times in the IMU file (e.g., once per store), the tool takes the first valid UPC it sees and ignores the rest. That's stable regardless of how the IMU file is sorted.

## ⚠️ Excel will silently destroy your data

This is the most important section. **Do not use Excel to view, edit, or re-save any of the CSVs in this workflow.** It will corrupt SKUs in ways you may not notice until after you've uploaded bad data to Salesforce.

### What Excel does wrong

When Excel opens a CSV, it tries to be helpful by guessing types. For text columns that happen to look numeric, this is destructive:

- **Leading zeros are stripped.** SKU `0001476` becomes `1476`. The original is gone after you save.
- **Long numeric SKUs become floats.** A 16+ digit SKU exceeds float precision and gets padded with zeros (`7078155500000000000000000…`). The original is gone after you save.
- **SKUs containing `E` become scientific notation.** SKU `70781555E104` is interpreted as 7.0781555 × 10¹⁰⁴ — a 105-digit number Excel happily displays. The original is gone after you save.

The damage happens *silently*, often before you even notice. The first time you save back to CSV, the corruption is locked in.

### What to do instead

**For viewing CSVs:** use Google Sheets. It treats imported CSV cells as text by default and doesn't coerce types. It's safe to open the tool's output in Sheets and even re-save it as CSV — round-tripping through Sheets preserves all values exactly.

**For downloading from Compass:** if Compass offers a "Download as CSV" option directly, use that. Avoid downloading as `.xlsx` and saving-as-CSV in Excel — that path will corrupt SKU columns silently.

**If you must use Excel** (e.g., the Compass download is `.xlsx` only):

1. Open Excel first, then go to **Data → From Text/CSV** (or "From Workbook")
2. In the import preview that appears, click the SKU column header and change its type to **Text** before clicking **Load**
3. Save as CSV

This is the only path that prevents Excel from mangling SKUs at load time. Double-clicking a CSV in Explorer or "Save As → CSV" from an already-corrupted workbook won't work — by then the values are already lost.

### How to spot already-corrupted SKUs

Before trusting any output:

- Open in Google Sheets or a plain text editor (VS Code, Notepad, etc.) — never just Excel
- Look for SKUs that should have leading zeros but don't
- Look for very long zero-padded numbers (`12345000000000000…`) that didn't exist in the source
- Look for unexpected scientific notation in SKU columns

If you spot any of this, regenerate the source CSV using one of the safe methods above and re-run the tool. The output is only as good as the input — the tool can't recover SKUs that were truncated before it ever saw them.

## Tips and gotchas

- **The "Salesforce-ready file" button only appears if at least one row was filled.** If every missing UPC is for a SKU not in the IMU file, you'll get a preview with everything highlighted yellow and no download button.
- **The status message tells you how many IMU values were ignored** as not-real-UPCs. If that number is high, it's worth checking what's actually in the IMU UPC column.
- **The tool reads the IMU CSV in a few seconds even at hundreds of thousands of rows.** If you're tempted to upload an .xlsx instead — don't. Earlier versions supported it but it took 30+ seconds and ~1GB of memory and was prone to leading-zero corruption. CSV is faster and safer.
- **The Salesforce-ready file uses headers `ID,UPC`.** If your Salesforce import expects different field names, you may need to rename the columns before uploading.

## Edge cases the tool handles

- Quoted fields with commas inside (`"Widget, Large"`)
- Embedded newlines inside quoted fields
- Escaped double quotes (`""`)
- UTF-8 BOM at the start of the file
- Blank rows
- Mixed line endings (LF, CRLF)

(All thanks to [PapaParse](https://www.papaparse.com/), which the tool uses for CSV parsing.)

## Built with

- [PapaParse](https://www.papaparse.com/) for CSV parsing
- Vanilla JavaScript, HTML, and CSS — no build step, no framework, single file
