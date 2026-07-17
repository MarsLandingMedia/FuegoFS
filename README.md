# FuegoFS

FuegoFS is a lightweight file serving application for ServiceNow. It stores file content assets in a custom table and exposes a REST endpoint so files can be retrieved by path. Files are served with proper MIME types, cache headers, and SPA routing fallback.

Host web applications, static sites, and assets directly from your ServiceNow instance — outside of Service Portal and outside of Workspace.

---

## What's New (v2.0)

- **SPA routing fallback** — unmatched paths return `index.html` for client-side routing
- **Proper MIME types** — 20+ extensions mapped, correct `Content-Type` headers
- **Cache headers** — hashed assets get `immutable`, `index.html` gets `no-cache`
- **ETag support** — `If-None-Match` returns `304 Not Modified`
- **Auth gate** — requires authenticated SN session (no anonymous access)
- **Directory listing** — `?op=list&prefix=/path` returns JSON file list
- **CRUD API** — POST to create/update, DELETE to remove, bulk upload supported
- **Health check** — `?op=health` for monitoring

---

## How It Works

1. **Files stored in the FuegoFileService table** — records hold file body, MIME type, and a unique path
2. **REST endpoint `/api/x_fuegofs/service`** — serves files by path with proper headers
3. **SPA fallback** — paths that don't match a record return `index.html` (enables React Router, etc.)
4. **FuegoServe Script Include** — handles serving, MIME detection, caching, and CRUD operations

---

## API Reference

### Serve a File
```
GET /api/x_fuegofs/service?/path/to/file.html
```
Returns file content with correct `Content-Type` and `Cache-Control` headers.

### SPA Routing
```
GET /api/x_fuegofs/service?/any/client/route
```
If the path doesn't match a file, returns `index.html` (200, not 404). This enables client-side routing for SPAs.

### Directory Listing
```
GET /api/x_fuegofs/service?op=list&prefix=/view
```
Returns JSON array of files under the given prefix.

### Store a File
```
POST /api/x_fuegofs/service
Content-Type: application/json

{
  "path": "/js/main.js",
  "content": "console.log('hello');",
  "mimeType": "text/javascript"
}
```

### Bulk Upload
```
POST /api/x_fuegofs/service?op=bulk
Content-Type: application/json

{
  "files": [
    { "path": "/index.html", "content": "..." },
    { "path": "/css/style.css", "content": "..." }
  ]
}
```

### Delete a File
```
DELETE /api/x_fuegofs/service?/path/to/file
```

### Health Check
```
GET /api/x_fuegofs/service?op=health
```

---

## MIME Types Supported

HTML, JS, CSS, JSON, XML, SVG, PNG, JPG, GIF, WebP, ICO, WOFF, WOFF2, TTF, EOT, OTF, TXT, MD, PDF, WASM, source maps.

## Cache Strategy

| File Type | Cache-Control |
|---|---|
| `index.html` | `no-cache, no-store, must-revalidate` |
| Hashed assets (`main.a1b2c3d4.js`) | `public, max-age=31536000, immutable` |
| Other files | `public, max-age=300` |

---

## Installation

1. Import the FuegoFS scoped application into ServiceNow.
2. Verify `FuegoServe` Script Include and the REST service are active.
3. Upload files via POST API or create records manually in FuegoFileService.
4. Retrieve files: `https://your-instance.service-now.com/api/x_fuegofs/service?/index.html`

---

## Example: Hosting a React App

```bash
# Build your React app
npm run build

# Upload the build to FuegoFS
for file in dist/**/*; do
  path="/${file#dist/}"
  content=$(cat "$file")
  mime=$(file --mime-type -b "$file")
  curl -X POST "https://instance.service-now.com/api/x_fuegofs/service" \
    -H "Content-Type: application/json" \
    -u "admin:password" \
    -d "{\"path\": \"$path\", \"content\": \"$content\", \"mimeType\": \"$mime\"}"
done

# Access your app
open "https://instance.service-now.com/api/x_fuegofs/service?/index.html"
```

---

## Disclaimers

**Legal and Use Notice**
This project is licensed under the GNU Affero General Public License and is a product of Mars Landing Media LLC. Provided for demonstration and educational purposes. No warranties or support provided. Use at your own risk.

**ServiceNow Notice**
ServiceNow is a registered trademark of ServiceNow, Inc. This project is not affiliated with or endorsed by ServiceNow.

**GlideRecordSecure Notice**
This project does not enforce GlideRecordSecure and assumes a trusted development environment.
