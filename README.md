# ChatGPT Apps SDK - TypeScript/Node Boilerplate

A ready-to-use TypeScript/Node.js template for building custom ChatGPT apps with interactive widgets using the MCP (Model Context Protocol) and OpenAI Apps SDK.

## Quick Start (5 minutes)

Get your app running in ChatGPT in 5 steps:

### 1. Install Dependencies
```bash
pnpm install
```

### 2. Build the Widgets
```bash
pnpm run build
```

### 3. Start ngrok and Get Your URL
```bash
# In a new terminal
ngrok http 8000
# Copy the https URL (e.g., https://abc123.ngrok-free.dev)
```

### 4. Rebuild with Your ngrok URL
```bash
BASE_URL=https://your-ngrok-url.ngrok-free.dev pnpm run build
```

### 5. Start the MCP Server
```bash
cd pizzaz_server_node
pnpm start
```

### 6. Add to ChatGPT
1. Enable [developer mode](https://platform.openai.com/docs/guides/developer-mode)
2. Go to Settings > Connectors
3. Add your ngrok URL: `https://your-ngrok-url.ngrok-free.dev/mcp`
4. Start chatting! Try: *"Show me the best pizzas on a map"*

---

## What's Included

This boilerplate includes 4 example widgets showcasing different UI patterns:

- **📍 Pizza Map** - Interactive map with markers
- **🎠 Pizza Carousel** - Image carousel widget
- **📔 Pizza Albums** - Grid/gallery layout
- **📝 Pizza List** - Simple list view

All widgets demonstrate the same MCP + Apps SDK integration pattern, just with different UIs.

---

## Project Structure

```
├── src/                          # Widget source code (React components)
│   ├── pizzaz/                   # Map widget
│   ├── pizzaz-carousel/          # Carousel widget
│   ├── pizzaz-list/              # List widget
│   └── pizzaz-albums/            # Gallery widget
├── assets/                       # Built widget bundles (HTML/JS/CSS)
├── pizzaz_server_node/           # Node MCP server
│   └── src/server.ts             # Main server file
├── build-all.mts                 # Build script
└── package.json
```

---

## Development Workflow

### Making Changes to Widgets

1. **Edit your React component** in `src/your-widget/`
2. **Rebuild**: `BASE_URL=https://your-ngrok-url.ngrok-free.dev pnpm run build`
3. **Restart server**: `cd pizzaz_server_node && pnpm start`
4. **Test in ChatGPT** - invoke the tool again

### Adding a New Widget

1. **Create new directory** in `src/` (e.g., `src/my-widget/`)
2. **Add your React component** as `index.jsx`
3. **Update `build-all.mts`** - add to `targets` array
4. **Update `pizzaz_server_node/src/server.ts`** - add to `widgets` array
5. **Build and test**

### Local Development (without ChatGPT)

For faster iteration without ChatGPT:

```bash
# Use Vite dev server for hot reload
pnpm run dev

# Or preview built bundles locally
pnpm run build          # Default uses http://localhost:4444
pnpm run serve          # Serves on :4444
```

---

## How It Works

### The MCP Server

The Node server (`pizzaz_server_node/src/server.ts`) does three things:

1. **Exposes MCP endpoints** at `/mcp` for ChatGPT to connect
2. **Serves static assets** (CSS/JS) from the `assets/` directory
3. **Returns widget HTML** when tools are called

### The Build Process

`build-all.mts` compiles your React widgets into standalone bundles:

- **Generates hashed files** (e.g., `pizzaz-2d2b.js`) for cache busting
- **Creates HTML templates** that reference the correct assets
- **Uses BASE_URL** to set where assets load from

### Widget Rendering Flow

```
User asks ChatGPT → ChatGPT calls MCP tool → Server returns HTML + metadata
→ ChatGPT renders widget in iframe → Widget loads CSS/JS from server
```

---

## Environment Variables

### `BASE_URL`

**Required for ngrok/production**. Sets the base URL for loading widget assets.

- **Local dev**: Defaults to `http://localhost:4444`
- **ngrok**: `BASE_URL=https://your-ngrok-url.ngrok-free.dev`
- **Production**: `BASE_URL=https://your-production-domain.com`

Usage:
```bash
BASE_URL=https://your-url.com pnpm run build
```

---

## Troubleshooting

### Widgets don't render / 404 errors

**Problem**: HTML references wrong asset URLs

**Solution**: Rebuild with correct BASE_URL
```bash
BASE_URL=https://your-ngrok-url.ngrok-free.dev pnpm run build
cd pizzaz_server_node && pnpm start
```

### CORS errors

**Problem**: Browser blocks cross-origin requests

**Solution**: This is fixed in the latest code. Make sure you're running the updated server.

### "Connector not available"

**Problem**: ChatGPT can't reach your server

**Solution**:
- Check ngrok is running: `ngrok http 8000`
- Verify MCP server is running: `cd pizzaz_server_node && pnpm start`
- Ensure connector URL ends with `/mcp`

### Changes not appearing

**Problem**: Old bundle is cached

**Solution**:
1. Rebuild: `BASE_URL=... pnpm run build`
2. Restart server: `cd pizzaz_server_node && pnpm start`
3. Remove and re-add connector in ChatGPT

---

## Deployment

### Production Setup

1. **Deploy your server** (Heroku, Railway, Render, etc.)
2. **Build with production URL**:
   ```bash
   BASE_URL=https://your-app.com pnpm run build
   ```
3. **Upload built assets** to your server
4. **Run the MCP server**: `cd pizzaz_server_node && pnpm start`
5. **Add connector** in ChatGPT: `https://your-app.com/mcp`

The Node MCP server serves both API and static assets from the same domain.

---

## Customizing for Your App

### 1. Rename from "Pizzaz"

Update these files:
- `pizzaz_server_node/` → Rename directory
- `pizzaz_server_node/src/server.ts` → Update server name and tool names
- `src/pizzaz*/` → Rename widget directories
- `build-all.mts` → Update `targets` array

### 2. Change Widget Data

Edit `pizzaz_server_node/src/server.ts`:
- Modify the `widgets` array
- Update tool handlers in `CallToolRequestSchema`
- Change `structuredContent` returned by tools

### 3. Customize UI

Edit React components in `src/your-widget/index.jsx`:
- Modify JSX/styling
- Add new components
- Update data rendering logic

---

## Tech Stack

- **MCP Server**: Node.js + [@modelcontextprotocol/sdk](https://github.com/modelcontextprotocol/typescript-sdk)
- **UI Framework**: React 19
- **Styling**: Tailwind CSS 4
- **Build Tool**: Vite 7
- **Package Manager**: pnpm

---

## Resources

- [Apps SDK Documentation](https://platform.openai.com/docs/guides/apps-sdk)
- [MCP Specification](https://modelcontextprotocol.io/)
- [Developer Mode](https://platform.openai.com/docs/guides/developer-mode)

---

## Contributing

Contributions welcome! Please open an issue or submit a PR.

## License

MIT License - see [LICENSE](./LICENSE) for details.
