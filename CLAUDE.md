# CLAUDE.md - Carbone Document Generation Service

## Overview

Standalone Carbone.io service for PDF/document generation in Pacific Bending ERP. Part of the Pacific Bending monorepo as a git submodule.

**Architecture:**
- Docker image: `carbone/carbone-ee:full-5.1.1-fonts`
- Port: 4000 (internal), https://pb-templates.erp.observer (external)
- Templates: Stored in Frappe (backup) + Carbone service (active)

**Related Components:**
- Backend DocType: `pacific-bending-backend/.../doctype/carbone_template/` - Template metadata
- Backend API: `pacific-bending-backend/.../api/carbone_api.py` - 12 endpoints
- Backend mappers: `pacific-bending-backend/.../carbone/` - Data transformation
- This repo: Docker service only (templates uploaded via API)

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

## Template Management (Jan 2026 Update)

### Storage Architecture

Templates are now stored in **two locations** for reliability:

1. **Frappe File System** (backup): Template files attached to `Carbone Template` DocType
2. **Carbone Service** (active): Auto-uploaded when template is created/updated

**Template Naming:** `TPL-{doc_type_code}-####` (e.g., TPL-QTN-0001, TPL-INV-0002)

### Adding New Templates

**Via Admin UI (Recommended):**

1. Go to ERPNext → Carbone Template → New
2. Fill in: template_name, doc_type, file_extension
3. Upload template file (Attach field)
4. Save → Auto-syncs to Carbone service

**Via API:**

```python
# POST /api/method/pacific_bending_app.api.carbone_api.upload_template
{
  "doc_type": "Quotation",
  "template_name": "Detailed",
  "file_data": "<base64>",
  "file_extension": "docx",
  "is_default": false
}
```

### Editing Templates with Carbone Studio

1. Enable Studio:
   ```bash
   CARBONE_STUDIO_ENABLED=true docker compose up -d
   ```
2. Access Studio: http://localhost:4000/studio (or https://pb-templates.erp.observer/studio)
3. Upload existing template or create new
4. Edit variables using drag-drop interface
5. Export DOCX → Upload via Admin UI or API
6. Disable Studio when done:
   ```bash
   CARBONE_STUDIO_ENABLED=false docker compose up -d
   ```

### Backup/Restore

**Export all templates:**
```python
# POST /api/method/pacific_bending_app.api.carbone_api.backup_templates
# Creates: templates.json manifest + template files by DocType
```

**Import from backup:**
```python
# POST /api/method/pacific_bending_app.api.carbone_api.import_templates
{ "input_dir": "/path/to/backup", "overwrite": false }
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

**Rendering Flow (Updated Jan 2026):**
```
Frontend Request (template_id or template_name)
    → Backend API (carbone_api.py)
    → _get_template() → Carbone Template DocType
    → template_doc.get_carbone_template_id() → Auto-sync if needed
    → MapperRegistry → DocType Mapper → Transform data
    → CarboneClient.render_with_template_id()
    → HTTP POST /render/:template_id
    → Return PDF/DOCX bytes
```

**Key Carbone Service Endpoints:**
- `POST /render/:template_id` - Render document with JSON data
- `POST /template` - Upload template (returns SHA256 ID)
- `GET /template/:template_id` - Download template
- `DELETE /template/:template_id` - Delete template
- `GET /status` - Health check

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

### Authentication Architecture

**Important:** Carbone authentication (`CARBONE_EE_AUTHENTICATION`) is **disabled** by design:

1. **Backend integration** - ERPNext's `carbone_client.py` makes unauthenticated HTTP requests to render documents. The Carbone service is on the internal `dokploy-network`, not publicly exposed.

2. **Studio access** - When `CARBONE_STUDIO_ENABLED=true`, the Studio UI is accessible. Access control should be handled at the Traefik/proxy layer (e.g., IP allowlist, BasicAuth middleware), not Carbone's built-in JWT auth.

3. **Why not JWT?** - Carbone's JWT authentication requires ES512 private/public key pairs and token generation. Adding this to the ERPNext backend would require significant changes. Since Carbone runs on an internal network, the simpler approach is proxy-level access control.

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
