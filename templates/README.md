# Carbone Document Templates

This directory contains DOCX/ODT templates for document generation using Carbone.io.

## Directory Structure

```
carbone-templates/
├── quotation/           # Quotation document templates
│   └── default.docx     # Default quotation template
├── sales_invoice/       # Sales Invoice templates
│   └── default.docx
├── delivery_note/       # Delivery Note templates
│   └── default.docx
├── work_order/          # Work Order templates
│   └── default.docx
└── job_card/            # Job Card templates
    └── default.docx
```

## Creating Templates

### Using Carbone Studio (Recommended)

1. Start the services: `docker compose up -d`
2. Access Carbone Studio: http://localhost:4000/studio
3. Upload a base DOCX/ODT document
4. Insert variables using the drag-drop interface
5. Export the template and save to the appropriate directory
6. Commit to Git for version control

### Template Variable Syntax

**Simple Variables:**
```
{d.quote_number}
{d.customer_name}
{d.transaction_date}
```

**Nested Objects:**
```
{d.company.name}
{d.company.address}
{d.customer.email}
```

**Loops (Line Items):**
```
| # | Item Code | Description | Qty | Rate | Amount |
|---|-----------|-------------|-----|------|--------|
| {d.items[i].row_num} | {d.items[i].item_code} | {d.items[i].description} | {d.items[i].qty} | {d.items[i].rate_formatted} | {d.items[i].amount_formatted} |
{d.items[i+1]}
```

**Formatters:**
```
{d.grand_total:formatN(2)}           # Format number: 1000.00
{d.date:formatD('MMMM DD, YYYY')}    # Format date: December 27, 2025
{d.status:upper()}                   # Uppercase: DRAFT
```

**Conditionals:**
```
{d.discount:ifGT(0):show('Discount: ')}
{d.status:ifEQ('Draft'):show('DRAFT'):elseShow('')}
```

**Hide Sections:**
```
{d.terms:ifNE(''):showBegin}
Terms and Conditions:
{d.terms}
{d.terms:showEnd}
```

## Available Variables by Document Type

### Quotation
- `quote_number` - Quotation ID (e.g., QTN-00042)
- `transaction_date` - Quote date
- `valid_till` - Expiry date
- `lead_time_days` - Estimated lead time
- `customer_name` - Customer display name
- `customer_address` - Formatted address
- `items[]` - Line items array
- `subtotal`, `tax_total`, `grand_total` - Totals
- `company` - Company info (name, address, email, phone, logo_url)

### Work Order
- `work_order_number` - Work Order ID
- `production_item` - Item being manufactured
- `qty_to_manufacture` - Quantity
- `planned_start_date`, `planned_end_date` - Dates
- `operations[]` - Operations array
- `materials[]` - Required materials array
- `status` - Current status

### Job Card
- `job_card_number` - Job Card ID
- `work_order` - Parent Work Order
- `operation` - Operation name
- `workstation` - Workstation
- `started_time`, `completed_time` - Timestamps
- `status` - Current status

### Sales Invoice
- `invoice_number` - Invoice ID
- `posting_date`, `due_date` - Dates
- `customer_name` - Customer
- `items[]` - Line items
- `grand_total`, `outstanding_amount` - Amounts

### Delivery Note
- `delivery_note_number` - Delivery Note ID
- `posting_date` - Date
- `customer_name` - Customer
- `items[]` - Items delivered
- `delivery_address` - Ship-to address

## Best Practices

1. **Use formatted versions** for display: `{d.grand_total_formatted}` instead of `{d.grand_total}`
2. **Test with sample data** in Carbone Studio before deploying
3. **Version control** all templates via Git
4. **Name templates clearly**: `default.docx`, `detailed.docx`, `simple.docx`
5. **Keep templates simple** - complex logic should be in the mapper, not the template
