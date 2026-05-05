# Missing UPC Tool

A lightweight web-based tool for managing and cleaning CSV files containing product data with UPC (Universal Product Code) values.

## Features

- **Upload CSV files** - Load product data directly in the browser
- **Find missing UPCs** - Display all rows where UPC values are empty or missing
- **Clean data** - Remove rows with missing UPCs and download the filtered CSV
- **Auto-detection** - Automatically identifies UPC columns (searches for "UPC", "Barcode", or "GTIN" headers)

## Usage

1. Open `index.html` in a web browser
2. Click "Choose File" and select your CSV file
3. Use one of two actions:
   - **Show rows missing UPC** - View which products are missing UPC data
   - **Remove rows missing UPC** - Filter out incomplete rows and download the cleaned file
4. Download the filtered CSV if needed

## Requirements

- Modern web browser (Chrome, Firefox, Safari, Edge)
- CSV file with a UPC/Barcode/GTIN column header

## File Structure

- `index.html` - Complete application (HTML, CSS, and JavaScript)

## Notes

- The tool runs entirely in your browser—no server or installation required
- Supports standard CSV format with quoted fields and escaped quotes
- UPC column must have a header named "UPC", "Barcode", or "GTIN"
