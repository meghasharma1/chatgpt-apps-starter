# ChatGPT Apps - Prompt-to-App Guide for AI Assistants

This guide helps AI assistants (like Claude or Cursor) help users build ChatGPT apps from scratch using this starter template.

## Overview

When a user wants to build a ChatGPT app, guide them through the **OpenAI App Development Process**:

1. **Solidify Use Case** - Understand what they want to build
2. **Define Tools** - Design the MCP tools and data structure
3. **Design Components** - Create the UI widgets

## Step 1: Solidify Use Case

### Questions to Ask

Ask the user these questions to clarify their vision:

1. **What problem are you solving?**
   - What task will users accomplish with this app?
   - What pain point does it address?

2. **What data will the app display?**
   - Static data or dynamic/API data?
   - What's the data structure? (e.g., list of items, hierarchical, geographical)

3. **What's the primary interaction?**
   - Browse and explore?
   - Search and filter?
   - Create and edit?
   - Visualize data?

4. **What's the ideal UI pattern?**
   - Map view (geographical data)
   - List view (simple browsing)
   - Gallery/grid (visual content)
   - Carousel (featured items)
   - Custom visualization

5. **What actions should users be able to take?**
   - View details
   - Filter/search
   - Save/bookmark
   - Share
   - Navigate to external links

### Example Use Cases

**Good Examples:**
- "Show nearby coffee shops on a map with ratings and photos"
- "Browse my team's design files in a gallery view"
- "Visualize my GitHub stars as an interactive graph"
- "Display my calendar events in a timeline"

**Too Vague:**
- "Help me be more productive" → Need specifics about what data and actions
- "Show me stuff" → What stuff? From where?

### Output: Use Case Summary

After gathering answers, create a clear summary:

```markdown
## Use Case Summary

**Problem:** [What pain point this solves]
**Data Source:** [Where data comes from]
**Primary Interaction:** [How users will use it]
**UI Pattern:** [Which widget type to use]
**Key Features:** [List of 3-5 main features]
```

## Step 2: Define Tools

Based on the use case, generate the MCP server configuration.

### Tool Definition Template

```typescript
{
  id: "your-tool-id",
  title: "Your Tool Title",
  templateUri: "ui://widget/your-widget.html",
  invoking: "Action happening message...",
  invoked: "Action completed message",
  html: readWidgetHtml("your-widget-name"),
  responseText: "Tool executed successfully!",
}
```

### Input Schema Design

Define what parameters your tool accepts:

```typescript
const toolInputSchema = {
  type: "object",
  properties: {
    // Define your input parameters here
    query: {
      type: "string",
      description: "Search query or filter parameter"
    },
    limit: {
      type: "number",
      description: "Number of results to return"
    }
  },
  required: ["query"], // Which params are required
  additionalProperties: false,
};
```

### Structured Content Design

Define what data structure the widget will receive:

```typescript
// Example for a location-based app
structuredContent: {
  locations: [
    {
      id: string,
      name: string,
      latitude: number,
      longitude: number,
      description: string,
      imageUrl: string,
      // ... other fields
    }
  ],
  metadata: {
    totalCount: number,
    query: string,
    // ... other metadata
  }
}
```

### File to Modify

Update `example-pizza-server/src/server.ts`:

1. Replace the `widgets` array with your new tool definitions
2. Update `toolInputSchema` with your parameters
3. Modify the `CallToolRequestSchema` handler to return your structured content

## Step 3: Design Components

### Choose a Base Template

Start with the example widget that's closest to your use case:

- **`example-pizza/`** - Map widget (for geographical data)
- **`example-pizza-carousel/`** - Carousel (for featured content)
- **`example-pizza-list/`** - List view (for simple browsing)
- **`example-pizza-albums/`** - Gallery view (for visual content)

### Widget Creation Steps

1. **Copy the template:**
   ```bash
   cp -r src/example-pizza-list src/your-app-name
   ```

2. **Update the main component:**
   - Modify `src/your-app-name/index.jsx`
   - Use `useWidgetProps()` to access structured content from MCP
   - Design your UI using React and Tailwind CSS

3. **Add to build targets:**
   Update `build-all.mts`:
   ```typescript
   const targets: string[] = [
     "your-app-name",
   ];
   ```

4. **Update server configuration:**
   Update `example-pizza-server/src/server.ts` to reference your widget

### Widget Props Pattern

All widgets receive data via `useWidgetProps()`:

```typescript
import { useWidgetProps } from "../use-widget-props";

function App() {
  const props = useWidgetProps();

  // Access your structured content
  const data = props.structuredContent;

  // Render your UI
  return <div>{/* Your widget UI */}</div>;
}
```

### Styling Guidelines

- Use Tailwind CSS for all styling
- Ensure mobile responsiveness
- Follow the existing design patterns for consistency
- Use `useMaxHeight()` for proper iframe sizing
- Use `useDisplayMode()` to detect compact vs full view

## Step 4: Implementation Workflow

### Quick Start Script

Generate this script for the user:

```bash
# 1. Create new widget directory
mkdir src/your-app-name

# 2. Copy closest template
cp src/example-pizza-list/index.jsx src/your-app-name/

# 3. Update build targets in build-all.mts

# 4. Update server configuration in example-pizza-server/src/server.ts

# 5. Build
pnpm run build

# 6. Start server
cd example-pizza-server && pnpm start

# 7. Test in ChatGPT
```

### Generated Files Checklist

- [ ] `src/your-app-name/index.jsx` - Main widget component
- [ ] `example-pizza-server/src/server.ts` - Updated with new tool
- [ ] `build-all.mts` - Added to targets array
- [ ] Any additional components or assets

## Step 5: Testing & Refinement

### Testing Checklist

Guide the user through testing:

1. **Build succeeds:**
   ```bash
   pnpm run build
   ```

2. **Server starts without errors:**
   ```bash
   cd example-pizza-server && pnpm start
   ```

3. **Widget renders in ChatGPT:**
   - Test with sample data
   - Verify all interactions work
   - Check mobile responsiveness

4. **Data flows correctly:**
   - Tool receives correct input parameters
   - Structured content matches widget expectations
   - All fields display properly

## Advanced: Authentication (TODO)

Authentication should be added for production apps. Consider:

- OAuth 2.0 for user authentication
- API key validation for external data sources
- Session management
- Rate limiting

**Note:** Authentication implementation guide to be added.

## Advanced: Metadata Optimization (TODO)

Optimize the MCP metadata for better ChatGPT integration:

- `openai/toolInvocation/invoking` - Loading message
- `openai/toolInvocation/invoked` - Success message
- `openai/widgetAccessible` - Accessibility features
- `openai/resultCanProduceWidget` - Widget rendering hints

**Note:** Metadata optimization guide to be added.

## Example: Building a GitHub Stars Viewer

Let's walk through a complete example:

### 1. Solidify Use Case

**User prompt:** "I want to see my GitHub starred repos in a nice gallery"

**AI Response:**
- Problem: Browsing GitHub stars is tedious
- Data Source: GitHub API
- Primary Interaction: Browse and explore
- UI Pattern: Gallery view
- Key Features: Show repo cards, filter by language, link to repos

### 2. Define Tools

```typescript
// In example-pizza-server/src/server.ts
const widgets = [
  {
    id: "github-stars",
    title: "Show GitHub Stars",
    templateUri: "ui://widget/github-stars.html",
    invoking: "Fetching your starred repos",
    invoked: "Loaded your stars",
    html: readWidgetHtml("github-stars"),
    responseText: "Here are your GitHub stars!",
  }
];

const toolInputSchema = {
  type: "object",
  properties: {
    language: {
      type: "string",
      description: "Filter by programming language (optional)"
    }
  },
  required: [],
  additionalProperties: false,
};

// In CallToolRequestSchema handler:
structuredContent: {
  repos: [
    {
      name: "repo-name",
      description: "Repo description",
      language: "TypeScript",
      stars: 1234,
      url: "https://github.com/...",
    }
  ]
}
```

### 3. Design Component

```bash
# Copy gallery template
cp -r src/example-pizza-albums src/github-stars
```

```jsx
// src/github-stars/index.jsx
import { useWidgetProps } from "../use-widget-props";

function App() {
  const props = useWidgetProps();
  const repos = props.structuredContent?.repos || [];

  return (
    <div className="grid grid-cols-2 gap-4 p-4">
      {repos.map(repo => (
        <div key={repo.name} className="border rounded-lg p-4">
          <h3 className="font-bold">{repo.name}</h3>
          <p className="text-sm text-gray-600">{repo.description}</p>
          <div className="flex justify-between mt-2">
            <span className="text-xs">{repo.language}</span>
            <span className="text-xs">⭐ {repo.stars}</span>
          </div>
        </div>
      ))}
    </div>
  );
}
```

## Tips for AI Assistants

1. **Always start with questions** - Don't assume you know what they want
2. **Show, don't tell** - Generate actual code, not just descriptions
3. **Test as you go** - Run builds after each major change
4. **Iterate quickly** - Get something working first, then refine
5. **Use the examples** - The example-pizza widgets are great references
6. **Keep it simple** - Start with minimal features, add complexity later

## Common Pitfalls to Avoid

1. **Over-engineering** - Start simple, add features later
2. **Wrong UI pattern** - Gallery for text data is awkward
3. **Missing structured content** - Widget needs data from MCP
4. **Forgetting to rebuild** - Always rebuild after code changes
5. **Not testing mobile** - ChatGPT is often used on mobile

## Reference Files

- **Server:** `example-pizza-server/src/server.ts`
- **Widget Examples:** `src/example-pizza*/`
- **Build Config:** `build-all.mts`
- **Hooks:** `src/use-widget-props.ts`, `src/use-max-height.ts`
- **README:** Main documentation

## Quick Reference: File Structure

```
chatgpt-apps-starter/
├── src/
│   └── your-app-name/           # Your widget code
│       └── index.jsx            # Main component
├── example-pizza-server/
│   └── src/
│       └── server.ts            # MCP tool definitions
├── build-all.mts                # Build configuration
└── assets/                      # Built widget bundles (generated)
```

---

**For users:** Tell the AI what you want to build, and it will guide you through this process!

**For AI assistants:** Use this guide to help users create ChatGPT apps efficiently and effectively.
