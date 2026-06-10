# Professional PDF Invoice Generator

Generate professional, branded PDF invoices from Python dicts or CSV files using ReportLab.

## Features

- **Branded layout** — dark indigo color scheme, logo support, clean typography
- **Two input modes** — Python dict or CSV files (invoices + line items)
- **Automatic invoice numbering** — scans the output folder and increments
- **Batch export** — generate dozens of PDFs in a single call
- **Full financials** — subtotal, configurable tax rate, grand total
- **Customizable** — currency symbol, notes, due date, company/client info

## Project Structure

```
invoice-generator/
├── invoice_generator.py   # Core library (InvoiceGenerator class)
├── demo.py                # Demo: runs all three usage modes
├── requirements.txt
├── sample_data/
│   ├── invoices.csv       # One row per invoice (header / client info)
│   └── invoice_items.csv  # Line items linked by invoice_id
└── output/                # Generated PDFs land here (created automatically)
```

## Installation

```bash
pip install -r requirements.txt
```

Requires Python 3.10+ (uses `list[dict]` type hints).

## Quick Start

```bash
python demo.py
```

PDFs are written to `output/`.

---

## Usage

### 1 — Single invoice from a dict

```python
from invoice_generator import InvoiceGenerator

gen = InvoiceGenerator(output_dir="output")

data = {
    # Seller
    "company_name":    "Acme Solutions LLC",
    "company_address": "123 Business Ave, New York, NY 10001",
    "company_email":   "billing@acme.com",
    "company_phone":   "+1 (212) 555-0100",
    # Buyer
    "client_name":    "Globex Corporation",
    "client_address": "742 Evergreen Terrace, Springfield, IL 62701",
    "client_email":   "accounts@globex.com",
    # Invoice meta
    "invoice_date":    "2026-06-10",   # defaults to today if omitted
    "due_date":        "2026-07-10",   # optional
    "tax_rate":        0.08,           # 8%
    "currency_symbol": "$",
    "logo_path":       "assets/logo.png",  # optional
    "notes":           "Net 30. Thank you for your business.",
    # Line items
    "items": [
        {"description": "Web Development (80 hrs)", "quantity": 1,  "unit_price": 7600.00},
        {"description": "UI/UX Design",              "quantity": 1,  "unit_price": 2400.00},
        {"description": "QA & Testing",              "quantity": 8,  "unit_price": 120.00},
    ],
}

pdf_path = gen.generate(data)
print(pdf_path)  # output/INV-00001.pdf
```

### 2 — Batch from a list of dicts

```python
paths = gen.generate_batch_from_dicts([invoice_a, invoice_b, invoice_c])
```

### 3 — Batch from CSV files

```python
paths = gen.generate_batch_from_csv(
    csv_path="sample_data/invoices.csv",
    items_csv_path="sample_data/invoice_items.csv",
)
```

#### `invoices.csv` columns

| Column | Required | Description |
|---|---|---|
| `id` | ✓ | Unique ID, links to `invoice_id` in items CSV |
| `invoice_number` | | If blank, auto-generated |
| `invoice_date` | | YYYY-MM-DD, defaults to today |
| `due_date` | | YYYY-MM-DD |
| `company_name` | ✓ | Your company name |
| `company_address` | ✓ | Your address |
| `company_email` | ✓ | Your email |
| `company_phone` | | Your phone |
| `client_name` | ✓ | Client name |
| `client_address` | ✓ | Client address |
| `client_email` | ✓ | Client email |
| `tax_rate` | | Float, e.g. `0.09` for 9% |
| `currency_symbol` | | Defaults to `$` |
| `notes` | | Free text printed at the bottom |
| `logo_path` | | Relative path to logo image |

#### `invoice_items.csv` columns

| Column | Required | Description |
|---|---|---|
| `invoice_id` | ✓ | Must match `id` in `invoices.csv` |
| `description` | ✓ | Line item description |
| `quantity` | ✓ | Number (can be decimal) |
| `unit_price` | ✓ | Price per unit |

---

## Adding a Logo

Place any PNG/JPG in `assets/` and set `logo_path` in the data dict or CSV column:

```python
"logo_path": "assets/my_logo.png"
```

If the path is missing or empty, the company name is rendered as large text instead.

---

## Auto-Numbering

When `invoice_number` is not provided, the generator scans the output folder for existing `INV-XXXXX.pdf` files and assigns the next available number (zero-padded to 5 digits).

You can customize the prefix:

```python
data["prefix"] = "ACME"   # produces ACME-00001.pdf
```

---

## Output Example

Each PDF contains:

1. Header with logo (or company name) and invoice metadata
2. "From" and "Bill To" side-by-side panels
3. Itemized table with description, quantity, unit price, and line total
4. Totals block — subtotal, tax, and grand total
5. Notes section (optional)
6. Footer with company contact

---

## Dependencies

| Package | Purpose |
|---|---|
| `reportlab` | PDF generation engine |
