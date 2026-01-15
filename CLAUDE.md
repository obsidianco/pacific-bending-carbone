# CLAUDE.md - Carbone Document Generation Service

## Overview

Standalone Carbone.io service for PDF/document generation in Pacific Bending ERP. Part of the Pacific Bending monorepo as a git submodule.

**Architecture:**
- Docker image: `carbone/carbone-ee:full-5.1.1-fonts`
- Port: 4000 (internal), https://pb-templates.erp.observer (external)
- Templates: Git-tracked in `./templates/`

**Related Components:**
- Backend mapper code: `pacific-bending-backend/pacific_bending_app/carbone/`
- Backend API: `pacific-bending-backend/pacific_bending_app/api/carbone_api.py`
- This repo: Docker service + templates only

## Common Commands

```bash
# Start service (local development)
docker compose up -d

# View logs
docker compose logs -f carbone

# Check health
curl http://localhost:4000/status

# Enable Studio for template editing
CARBONE_STUDIO_ENABLED=true docker compose up -d

# Access Studio (when enabled)
open http://localhost:4000/studio
```

## Template Management

### Directory Structure

Templates are organized by ERPNext DocType:

```
templates/
├── quotation/        # Customer quotations
├── work_order/       # Manufacturing work orders
├── job_card/         # Shop floor job cards
├── sales_invoice/    # Customer invoices
└── delivery_note/    # Shipping documents
```

### Adding/Editing Templates

**Using Carbone Studio (Recommended):**

1. Enable Studio:
   ```bash
   CARBONE_STUDIO_ENABLED=true docker compose up -d
   ```
2. Access Studio: http://localhost:4000/studio (or https://pb-templates.erp.observer/studio)
3. Upload a base DOCX/ODT document
4. Insert variables using the drag-drop interface
5. Export template and save to appropriate directory
6. Commit to Git
7. Disable Studio:
   ```bash
   CARBONE_STUDIO_ENABLED=false docker compose up -d
   ```

**Template Variable Syntax:**

```
Simple:     {d.quote_number}
Nested:     {d.company.name}
Loops:      {d.items[i].item_code} ... {d.items[i+1]}
Formatters: {d.grand_total:formatN(2)}
Conditions: {d.discount:ifGT(0):show('Discount: ')}
```

### Available Variables

See `pacific-bending-backend/carbone-templates/README.md` for complete variable reference by DocType.

## Integration with Backend

The backend communicates via HTTP to this service:

```python
# Backend CarboneClient configuration (site_config.json)
{
    "carbone_url": "http://<container-name>:4000"
}
```

**Communication Flow:**
```
Frontend Request
    → Backend API (carbone_api.py)
    → MapperRegistry → DocType Mapper
    → CarboneClient.render()
    → HTTP POST to this service
    → Return PDF/DOCX bytes
```

## Dokploy Deployment

This service runs as a separate Dokploy project:

| Setting | Value |
|---------|-------|
| Project | `pacific-bending-carbone` |
| Build | Docker Compose |
| Domain | `pb-templates.erp.observer` |
| Port | 4000 |
| Network | `dokploy-network` (shared with backend) |

### Environment Variables (Production)

Set in Dokploy UI:
```
CARBONE_STUDIO_ENABLED=false
```

### Finding Container Name

After Dokploy deployment:
```bash
docker ps --filter "name=carbone" --format "{{.Names}}"
# Example: pacificbendingcarbone-carbone-xxxx-carbone-1
```

This container name is used in the backend's `CARBONE_URL` environment variable.

## Health Check

```bash
# Local
curl http://localhost:4000/status

# Production
curl https://pb-templates.erp.observer/status

# Expected response
{"success":true,"version":"5.1.1","..."}
```

## Troubleshooting

### Templates Not Loading

1. Check volume mount: `docker exec <container> ls /app/template/`
2. Verify templates directory has correct structure
3. Templates must be DOCX or ODT format

### PDF Generation Fails

1. Check Carbone logs: `docker compose logs carbone`
2. Verify LibreOffice is available (use `full` image, not `slim`)
3. Increase memory if OOM errors: set `memory: 4G` in docker-compose.yml

### Studio Not Accessible

1. Verify `CARBONE_STUDIO_ENABLED=true` is set
2. Check if port 4000 is accessible
3. In production, ensure Traefik routing is configured
