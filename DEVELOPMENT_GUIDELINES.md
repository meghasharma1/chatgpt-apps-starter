# Development Guidelines for ChatGPT Apps

## Critical Setup Requirements

### 1. Asset Serving Configuration

**IMPORTANT**: Assets and MCP server MUST be served from the same URL/server.

- ✅ **Correct Setup**: MCP server serves both protocol endpoints AND static assets
  - MCP endpoints: `http://localhost:8000/mcp`
  - Static assets: `http://localhost:8000/example-*.js`, `http://localhost:8000/example-*.css`

- ❌ **Incorrect Setup**: Separate servers for assets and MCP
  - MCP on port 8000, assets on port 4444 (causes CORS and loading issues)

**Configuration Steps**:
1. Build assets with the MCP server URL:
   ```bash
   BASE_URL=http://localhost:8000 pnpm run build
   ```

2. MCP server configuration (`mcp/src/server.ts`):
   - Ensure static file serving is enabled
   - Fix path handling: use `path.join(ASSETS_DIR, url.pathname.slice(1))` to remove leading slash
   - Serve from the assets directory in the project root

### 2. Naming Conventions and ID Consistency

**IMPORTANT**: DOM element IDs must match between HTML and JavaScript.

#### Component Naming Pattern
When creating a new component in `src/example-{name}/`:

1. **Folder name**: `example-{name}` (e.g., `example-tfsa-rrsp`)
2. **HTML root element ID**: `example-{name}-root` (e.g., `id="example-tfsa-rrsp-root"`)
3. **React mount point**: Must match the HTML ID exactly
   ```javascript
   createRoot(document.getElementById("example-{name}-root")).render(...)
   ```

#### Common ID Mismatch Issues

❌ **Wrong**:
```html
<!-- HTML -->
<div id="example-tfsa-rrsp-root"></div>
```
```javascript
// JavaScript
createRoot(document.getElementById("tfsa-rrsp-root")) // MISSING "example-" prefix!
```

✅ **Correct**:
```html
<!-- HTML -->
<div id="example-tfsa-rrsp-root"></div>
```
```javascript
// JavaScript
createRoot(document.getElementById("example-tfsa-rrsp-root")) // Matches!
```

### 3. Build Configuration

Add new components to `build-all.mts`:
```typescript
const targets: string[] = [
  "example-pizza",
  "example-pizza-carousel",
  "example-pizza-list",
  "example-pizza-albums",
  "example-tfsa-rrsp", // Add your new component here
];
```

### 4. MCP Server Registration

Add widget to `mcp/src/server.ts`:
```typescript
const widgets: PizzazWidget[] = [
  // ... existing widgets
  {
    id: "tfsa-rrsp-calculator",              // Tool ID
    title: "TFSA vs RRSP Calculator",        // Display name
    templateUri: "ui://widget/tfsa-rrsp-calculator.html",
    invoking: "Loading calculator...",
    invoked: "Displayed calculator",
    html: readWidgetHtml("example-tfsa-rrsp"), // Matches folder name
    responseText: "Here's your calculator!",
  },
];
```

### 5. Pre-Launch Checklist

Before testing with ChatGPT:

- [ ] Component folder created: `src/example-{name}/`
- [ ] Component added to `build-all.mts` targets
- [ ] Widget registered in `mcp/src/server.ts`
- [ ] HTML element ID matches JavaScript mount point
- [ ] Built with correct BASE_URL: `BASE_URL=http://localhost:8000 pnpm run build`
- [ ] MCP server running on port 8000
- [ ] Assets accessible at `http://localhost:8000/example-{name}-*.js`
- [ ] ngrok tunnel active (for external testing)

## Testing

### Local Testing
```bash
# 1. Build assets
BASE_URL=http://localhost:8000 pnpm run build

# 2. Start MCP server
cd mcp && pnpm start

# 3. Verify asset serving
curl http://localhost:8000/example-{name}-2d2b.js
curl http://localhost:8000/example-{name}-2d2b.css
curl http://localhost:8000/example-{name}-2d2b.html
```

### Production Testing
```bash
# Start ngrok tunnel
ngrok http 8000

# Use the ngrok URL in ChatGPT configuration
# Example: https://xxx.ngrok-free.dev
```

## Common Errors and Solutions

### React Error #299: "Target container is not a DOM element"
**Cause**: ID mismatch between HTML and JavaScript
**Solution**: Ensure `document.getElementById()` matches the HTML `id` attribute exactly

### 404 on Asset Files
**Cause**: Incorrect path handling in MCP server
**Solution**: Use `url.pathname.slice(1)` to remove leading slash before joining with ASSETS_DIR

### CORS Errors
**Cause**: Assets served from different origin than MCP server
**Solution**: Serve everything from the same MCP server, rebuild with correct BASE_URL
