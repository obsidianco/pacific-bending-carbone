# Sample Carbone Templates

This folder contains sample templates for testing and reference. These are **not production templates** but serve as examples of how to build professional Carbone.io templates for Pacific Bending ERP.

## Contents

| File | Description |
|------|-------------|
| `quotation-modern.html` | Modern HTML quotation template with all available variables |
| `quotation-sample-data.json` | Sample JSON data for testing the quotation template |

## Quick Start

### Testing with Carbone Studio

1. **Start Carbone Studio:**
   ```bash
   cd pacific-bending-carbone
   CARBONE_STUDIO_ENABLED=true docker compose up -d
   ```

2. **Open Studio:** http://localhost:4000/studio (or https://pb-templates.erp.observer/studio)

3. **Upload Template:** Click "Upload" and select `quotation-modern.html`

4. **Add Test Data:** Paste the contents of `quotation-sample-data.json` into the data panel

5. **Preview:** Click "Render" to see the PDF output

### Testing via API

```bash
# Render template with sample data
curl -X POST "http://localhost:4000/render" \
  -H "Content-Type: application/json" \
  -d '{
    "template": "<html-content-here>",
    "data": {...json-data-here...},
    "options": {
      "convertTo": "pdf"
    }
  }'
```

## Template Variables Reference

### Header Fields
| Variable | Description | Example |
|----------|-------------|---------|
| `{d.quote_number}` | Quotation ID | QTN-00042 |
| `{d.customer_name}` | Customer display name | Acme Manufacturing Ltd. |
| `{d.customer_address}` | Full address (multi-line) | 1234 Industrial Blvd... |
| `{d.transaction_date}` | Quote date (formatted) | January 16, 2026 |
| `{d.valid_till}` | Expiry date (formatted) | February 15, 2026 |
| `{d.status}` | Quote status | Draft, Submitted, etc. |
| `{d.watermark}` | Draft watermark text | "DRAFT" or "" |
| `{d.prepared_by}` | Author name | Sarah Johnson |
| `{d.currency}` | Currency code | CAD |
| `{d.currency_symbol}` | Currency symbol | $ |

### Contact Person (Optional)
| Variable | Description |
|----------|-------------|
| `{d.contact_name}` | Contact full name |
| `{d.contact_email}` | Contact email |
| `{d.contact_phone}` | Contact phone |
| `{d.contact_designation}` | Contact job title |
| `{d.has_contact}` | Boolean: has contact info |

### Line Items (Loop)
Use `{d.items[i].field}` for iteration, close with `{d.items[i+1]}`:

| Variable | Description |
|----------|-------------|
| `{d.items[i].row_num}` | Row number (1, 2, 3...) |
| `{d.items[i].item_code}` | Item code |
| `{d.items[i].item_name}` | Item name |
| `{d.items[i].description}` | Full description |
| `{d.items[i].qty}` | Quantity (number) |
| `{d.items[i].qty_formatted}` | Quantity (formatted) |
| `{d.items[i].uom}` | Unit of measure |
| `{d.items[i].rate}` | Unit price (number) |
| `{d.items[i].rate_formatted}` | Unit price (with currency) |
| `{d.items[i].amount}` | Line total (number) |
| `{d.items[i].amount_formatted}` | Line total (with currency) |
| `{d.items[i].notes}` | Line item notes |
| `{d.items[i].has_notes}` | Boolean: has notes |
| `{d.items[i].lead_time}` | Lead time value |
| `{d.items[i].lead_time_unit}` | Lead time unit (days/weeks) |
| `{d.items[i].has_lead_time}` | Boolean: has lead time |
| `{d.items[i].dimension_1}` | First dimension |
| `{d.items[i].dimension_2}` | Second dimension |
| `{d.items[i].wall_thickness}` | Wall thickness |
| `{d.items[i].freight_cost_formatted}` | Item freight cost |

### Computed/Totals
| Variable | Description |
|----------|-------------|
| `{d.subtotal}` | Subtotal (number) |
| `{d.subtotal_formatted}` | Subtotal (with currency) |
| `{d.tax_total_formatted}` | Tax amount |
| `{d.discount_formatted}` | Discount amount |
| `{d.freight_total_formatted}` | Total freight |
| `{d.grand_total_formatted}` | Grand total |
| `{d.item_count}` | Number of line items |
| `{d.total_qty}` | Sum of all quantities |
| `{d.days_valid}` | Days until expiry |
| `{d.has_discount}` | Boolean: has discount |
| `{d.has_tax}` | Boolean: has tax |
| `{d.has_freight}` | Boolean: has freight |

### Company Info
| Variable | Description |
|----------|-------------|
| `{d.company.name}` | Company name |
| `{d.company.address}` | Company address |
| `{d.company.phone}` | Phone number |
| `{d.company.fax}` | Fax number |
| `{d.company.email}` | Email address |
| `{d.company.website}` | Website URL |
| `{d.company.tax_id}` | Tax/Business ID |

## Carbone Syntax Reference

### Conditionals
```html
<!-- Show block if condition is true -->
{d.has_contact:ifEQ(true):showBegin}
  <p>Contact: {d.contact_name}</p>
{d.has_contact:showEnd}

<!-- Inline conditional -->
{d.discount:ifGT(0):show('Discount: ')}{d.discount_formatted}

<!-- Hide if condition is true -->
{d.watermark:ifEQ(''):hideBegin}
  <div class="watermark">{d.watermark}</div>
{d.watermark:hideEnd}
```

### Formatters
```html
<!-- Numbers -->
{d.grand_total:formatN(2)}           <!-- 14433.85 -->

<!-- Dates (use _raw fields) -->
{d.transaction_date_raw:formatD('LL')}  <!-- January 16, 2026 -->
{d.transaction_date_raw:formatD('YYYY-MM-DD')}  <!-- 2026-01-16 -->

<!-- Currency -->
{d.grand_total:formatC(2)}           <!-- $14,433.85 -->

<!-- Text transforms -->
{d.status:upper}                     <!-- DRAFT -->
{d.customer_name:lowerCase:ucFirst}  <!-- Acme manufacturing ltd. -->
```

### Loops
```html
<!-- Table row loop -->
<tr>
  <td>{d.items[i].item_code}</td>
  <td>{d.items[i].description}</td>
  <td>{d.items[i].amount_formatted}</td>
</tr>
{d.items[i+1]}

<!-- Filter within loop -->
{d.items[i, has_notes == true].notes}  <!-- Only items with notes -->
```

### Drop/Keep Elements
```html
<!-- Drop table row if condition -->
{d.items[i].has_freight:ifEQ(false):drop(row)}

<!-- Keep paragraph only if -->
{d.terms:ifNE(''):keep(p)}Terms: {d.terms}{d.terms:showEnd}
```

## Design Tips

1. **Use formatted values** for display: `{d.grand_total_formatted}` instead of `{d.grand_total}`

2. **Use boolean flags** for conditionals: `{d.has_contact:ifEQ(true):showBegin}` is cleaner than checking empty strings

3. **Keep CSS inline or in `<style>`** tags - external stylesheets don't work in Carbone

4. **Test with edge cases:**
   - No line items
   - Very long descriptions
   - Zero discount/tax/freight
   - Draft vs Submitted status

5. **Use print-specific CSS** for PDF output:
   ```css
   @media print {
     .no-print { display: none; }
   }
   ```

## Converting to Production

To use a sample template in production:

1. **Test thoroughly** in Carbone Studio with various data scenarios

2. **Export as DOCX** (optional) - DOCX templates are more reliable for complex layouts

3. **Upload via Admin UI:**
   - Go to ERPNext → Carbone Template → New
   - Fill in template_name, doc_type (Quotation)
   - Upload the template file
   - Save → Auto-syncs to Carbone service

4. **Set as default** if replacing the standard template

## Support

- **Carbone Documentation:** https://carbone.io/documentation.html
- **Carbone Studio:** Enable with `CARBONE_STUDIO_ENABLED=true`
- **Backend API:** See `pacific-bending-backend/API_REFERENCE.md`
