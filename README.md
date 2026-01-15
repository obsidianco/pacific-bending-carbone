# Pacific Bending Carbone

Document generation service for Pacific Bending ERP using [Carbone.io](https://carbone.io).

## Overview

This service generates PDF and DOCX documents from templates using data from ERPNext. It's part of the Pacific Bending monorepo as a git submodule.

**Supported Documents:**
- Quotations
- Work Orders
- Job Cards
- Sales Invoices
- Delivery Notes

## Quick Start

```bash
# Start service
docker compose up -d

# Check health
curl http://localhost:4000/status

# Enable Studio for template editing
CARBONE_STUDIO_ENABLED=true docker compose up -d
```

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Pacific Bending ERP                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌──────────────┐    HTTP    ┌──────────────────────────┐  │
│   │   ERPNext    │ ─────────► │   Carbone Service        │  │
│   │   Backend    │            │   (this repo)            │  │
│   │              │ ◄───────── │                          │  │
│   │ - Mappers    │   PDF/DOCX │ - Templates              │  │
│   │ - API        │            │ - LibreOffice            │  │
│   └──────────────┘            └──────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Template Development

### Using Carbone Studio

1. Enable Studio:
   ```bash
   CARBONE_STUDIO_ENABLED=true docker compose up -d
   ```

2. Open http://localhost:4000/studio

3. Create or upload a template

4. Use the variable browser to insert data fields

5. Save template to `templates/<doctype>/`

6. Commit to Git

### Template Syntax

```
{d.customer_name}                    # Simple variable
{d.company.address}                  # Nested object
{d.items[i].item_code}              # Loop item
{d.grand_total:formatN(2)}          # Formatter
{d.notes:ifNE(''):show(d.notes)}    # Conditional
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `CARBONE_STUDIO_ENABLED` | `false` | Enable template editor |
| `CARBONE_EE_LICENSE` | - | Enterprise license key |
| `CARBONE_STUDIO_AUTH` | - | Studio basic auth (user:pass) |

## Deployment

This service is deployed via Dokploy as a separate project:

- **Domain:** pb-templates.erp.observer
- **Network:** Shared `dokploy-network` with backend

See `CLAUDE.md` for detailed deployment instructions.

## License

Proprietary - Pacific Bending Industries
