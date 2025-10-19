# Apps SDK Examples Gallery

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

This repository showcases example UI components to be used with the Apps SDK, as well as example MCP servers that expose a collection of components as tools.
It is meant to be used as a starting point and source of inspiration to build your own apps for ChatGPT.

## MCP + Apps SDK overview

The Model Context Protocol (MCP) is an open specification for connecting large language model clients to external tools, data, and user interfaces. An MCP server exposes tools that a model can call during a conversation and returns results according to the tool contracts. Those results can include extra metadata—such as inline HTML—that the Apps SDK uses to render rich UI components (widgets) alongside assistant messages.

Within the Apps SDK, MCP keeps the server, model, and UI in sync. By standardizing the wire format, authentication, and metadata, it lets ChatGPT reason about your connector the same way it reasons about built-in tools. A minimal MCP integration for Apps SDK implements three capabilities:

1. **List tools** – Your server advertises the tools it supports, including their JSON Schema input/output contracts and optional annotations (for example, `readOnlyHint`).
2. **Call tools** – When a model selects a tool, it issues a `call_tool` request with arguments that match the user intent. Your server executes the action and returns structured content the model can parse.
3. **Return widgets** – Alongside structured content, return embedded resources in the response metadata so the Apps SDK can render the interface inline in the Apps SDK client (ChatGPT).

Because the protocol is transport agnostic, you can host the server over Server-Sent Events or streaming HTTP—Apps SDK supports both.

The MCP servers in this demo highlight how each tool can light up widgets by combining structured payloads with `_meta.openai/outputTemplate` metadata returned from the MCP servers.

## Repository structure

- `src/` – Source for each widget example.
- `assets/` – Generated HTML, JS, and CSS bundles after running the build step.
- `pizzaz_server_node/` – MCP server implemented with the official TypeScript SDK.
- `pizzaz_server_python/` – Python MCP server that returns the Pizzaz widgets.
- `solar-system_server_python/` – Python MCP server for the 3D solar system widget.
- `build-all.mts` – Vite build orchestrator that produces hashed bundles for every widget entrypoint.

## Prerequisites

- Node.js 18+
- pnpm (recommended) or npm/yarn
- Python 3.10+ (for the Python MCP server)

## Install dependencies

Clone the repository and install the workspace dependencies:

```bash
pnpm install
```

> Using npm or yarn? Install the root dependencies with your preferred client and adjust the commands below accordingly.

## Build the components gallery

The components are bundled into standalone assets that the MCP servers serve as reusable UI resources.

### For local development (default):

```bash
pnpm run build
```

This builds assets that reference `http://localhost:4444` for local testing.

### For ngrok/remote testing:

When testing with ChatGPT via ngrok, set the `BASE_URL` environment variable to your ngrok URL:

```bash
BASE_URL=https://your-ngrok-url.ngrok-free.dev pnpm run build
```

This ensures the generated HTML files reference the correct public URL for loading CSS and JS files.

**Note:** The build command runs `build-all.mts`, producing versioned `.html`, `.js`, and `.css` files inside `assets/` with content-based hashes (e.g., `pizzaz-2d2b.js`). Each widget is wrapped with the CSS it needs.

To iterate on your components locally, you can also launch the Vite dev server:

```bash
pnpm run dev
```

## Serve the static assets (Optional)

**The Pizzaz Node server automatically serves static assets**, so you typically don't need to run this separately when using the Node MCP server.

However, if you want to preview the bundles independently, you can start the static file server:

```bash
pnpm run serve
```

The assets are exposed at [`http://localhost:4444`](http://localhost:4444) with CORS enabled.

## Run the MCP servers

The repository ships several demo MCP servers that highlight different widget bundles:

- **Pizzaz (Node & Python)** – pizza-inspired collection of tools and components
- **Solar system (Python)** – 3D solar system viewer

Every tool response includes plain text content, structured JSON, and `_meta.openai/outputTemplate` metadata so the Apps SDK can hydrate the matching widget.

### Pizzaz Node server

```bash
cd pizzaz_server_node
pnpm start
```

### Pizzaz Python server

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r pizzaz_server_python/requirements.txt
uvicorn pizzaz_server_python.main:app --port 8000
```

### Solar system Python server

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r solar-system_server_python/requirements.txt
uvicorn solar-system_server_python.main:app --port 8000
```

You can reuse the same virtual environment for all Python servers—install the dependencies once and run whichever entry point you need.

## Testing in ChatGPT

To add these apps to ChatGPT, enable [developer mode](https://platform.openai.com/docs/guides/developer-mode), and add your apps in Settings > Connectors.

### Local testing with ngrok

The Node MCP server serves both the API endpoints and static assets from a single port (8000), making it easy to expose with ngrok.

**Step-by-step:**

1. **Build with your ngrok URL:**
   ```bash
   # First, start ngrok to get your URL
   ngrok http 8000

   # Copy the ngrok URL (e.g., https://abc123.ngrok-free.dev)
   # Then rebuild with that URL:
   BASE_URL=https://abc123.ngrok-free.dev pnpm run build
   ```

2. **Start the MCP server:**
   ```bash
   cd pizzaz_server_node
   pnpm start
   ```

3. **Add to ChatGPT:**
   - Go to Settings > Connectors
   - Add your ngrok URL with `/mcp` endpoint: `https://abc123.ngrok-free.dev/mcp`

Once you add a connector, you can use it in ChatGPT conversations.

You can add your app to the conversation context by selecting it in the "More" options.

![more-chatgpt](https://github.com/user-attachments/assets/26852b36-7f9e-4f48-a515-aebd87173399)

You can then invoke tools by asking something related. For example, for the Pizzaz app, you can ask "What are the best pizzas in town?".

## Next steps

- Customize the widget data: edit the handlers in `pizzaz_server_node/src`, `pizzaz_server_python/main.py`, or the solar system server to fetch data from your systems.
- Create your own components and add them to the gallery: drop new entries into `src/` and they will be picked up automatically by the build script.

### Deploy your MCP server

You can use the cloud environment of your choice to deploy your MCP server.

When deploying, build your assets with the production URL:

```bash
BASE_URL=https://your-production-server.com pnpm run build
```

This ensures the generated HTML files reference your production server for loading CSS and JS files. The Node MCP server will serve both the API endpoints and static assets from the same domain.

## Contributing

You are welcome to open issues or submit PRs to improve this app, however, please note that we may not review all suggestions.

## License

This project is licensed under the MIT License. See [LICENSE](./LICENSE) for details.
