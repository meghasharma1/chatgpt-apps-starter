# TODO: Metadata Optimization

This document outlines tasks for optimizing MCP metadata to improve the ChatGPT integration experience.

## Why Optimize Metadata?

Proper metadata helps ChatGPT:
- Display better loading and success messages
- Understand tool capabilities and limitations
- Provide accessibility features
- Render widgets more effectively
- Give users better feedback

## Metadata Fields Reference

### Core Widget Metadata

```typescript
_meta: {
  "openai/outputTemplate": string,           // Widget template URI
  "openai/toolInvocation/invoking": string,  // Loading message
  "openai/toolInvocation/invoked": string,   // Success message
  "openai/widgetAccessible": boolean,        // Accessibility support
  "openai/resultCanProduceWidget": boolean,  // Can render widget
}
```

## Implementation Tasks

### 1. Optimize Loading Messages (`invoking`)

**Current:** Generic messages like "Hand-tossing a map"

**TODO:** Make messages contextual and informative

- [ ] Review all `invoking` messages in `example-pizza-server/src/server.ts`
- [ ] Update to be specific to user action
  ```typescript
  // ❌ Bad
  invoking: "Hand-tossing a map"

  // ✅ Good
  invoking: "Finding pizza places near you..."
  invoking: "Searching for ${query}..."
  invoking: "Loading your starred repositories..."
  ```

- [ ] Consider dynamic messages based on input parameters
  ```typescript
  const invokingMessage = args.location
    ? `Searching near ${args.location}...`
    : "Finding places nearby...";
  ```

### 2. Optimize Success Messages (`invoked`)

**Current:** Generic messages like "Served a fresh map"

**TODO:** Provide specific, actionable feedback

- [ ] Review all `invoked` messages
- [ ] Update to reflect what was accomplished
  ```typescript
  // ❌ Bad
  invoked: "Served a fresh map"

  // ✅ Good
  invoked: "Found 12 pizza places"
  invoked: "Loaded 47 repositories"
  invoked: "Showing results for ${query}"
  ```

- [ ] Include counts or key metrics when possible
  ```typescript
  const invokedMessage = `Found ${results.length} ${results.length === 1 ? 'result' : 'results'}`;
  ```

### 3. Accessibility Features (`widgetAccessible`)

- [ ] Ensure all widgets have `"openai/widgetAccessible": true`
- [ ] Add ARIA labels to interactive elements
  ```jsx
  <button aria-label="View details">
    <IconExpand />
  </button>
  ```

- [ ] Ensure keyboard navigation works
  - [ ] Tab order is logical
  - [ ] Focus states are visible
  - [ ] All actions accessible via keyboard

- [ ] Add alt text to images
  ```jsx
  <img src={place.imageUrl} alt={`Photo of ${place.name}`} />
  ```

- [ ] Test with screen readers

### 4. Widget Rendering Hints (`resultCanProduceWidget`)

- [ ] Verify all tools have `"openai/resultCanProduceWidget": true`
- [ ] Ensure this is only true for tools that actually render widgets
- [ ] Set to false for tools that only return text

### 5. Tool Annotations

**File:** `example-pizza-server/src/server.ts`

Current annotations:
```typescript
annotations: {
  destructiveHint: false,    // Tool doesn't modify data
  openWorldHint: false,      // Tool doesn't access external URLs
  readOnlyHint: true,        // Tool only reads data
}
```

- [ ] Review annotation accuracy for each tool
- [ ] Set `destructiveHint: true` if tool modifies data
- [ ] Set `openWorldHint: true` if tool accesses external APIs
- [ ] Set `readOnlyHint: false` if tool has side effects

### 6. Embedded Widget Resources

- [ ] Optimize embedded widget resource metadata
  ```typescript
  const widget Resource = {
    type: "resource",
    resource: {
      uri: widget.templateUri,
      mimeType: "text/html+skybridge",
      text: widget.html,
      title: widget.title,
      _meta: {
        // Add specific metadata for embedded resources
        "openai/cacheControl": "max-age=3600",  // TODO: Research this
        "openai/version": "1.0",                 // TODO: Research this
      }
    },
  };
  ```

### 7. Error Handling Metadata

- [ ] Add error metadata when tool calls fail
  ```typescript
  return {
    content: [{
      type: "text",
      text: "Error: Unable to fetch data"
    }],
    isError: true,
    _meta: {
      "openai/errorType": "API_ERROR",
      "openai/errorMessage": "Failed to connect to external service",
      "openai/retryable": true,
    }
  };
  ```

### 8. Performance Hints

- [ ] Research and add performance-related metadata
  ```typescript
  _meta: {
    // TODO: Research these fields
    "openai/cacheable": true,
    "openai/cacheKey": string,
    "openai/estimatedDuration": number,
  }
  ```

### 9. Content Metadata

- [ ] Add metadata about structured content
  ```typescript
  _meta: {
    "openai/contentType": "location-list",
    "openai/itemCount": results.length,
    "openai/hasMore": hasMoreResults,
    "openai/pageSize": 20,
  }
  ```

### 10. Widget Sizing Hints

- [ ] Add metadata about preferred widget size
  ```typescript
  _meta: {
    "openai/preferredHeight": "400px",
    "openai/minHeight": "300px",
    "openai/maxHeight": "800px",
    "openai/responsive": true,
  }
  ```

## Research Needed

These metadata fields need research to understand their purpose and usage:

- [ ] `openai/cacheControl` - How does ChatGPT cache widgets?
- [ ] `openai/version` - Version numbering strategy?
- [ ] `openai/errorType` - What error types does ChatGPT recognize?
- [ ] `openai/retryable` - Does ChatGPT auto-retry failed tools?
- [ ] `openai/contentType` - Does ChatGPT use this for anything?
- [ ] `openai/preferredHeight` - Does ChatGPT respect size hints?

**Action:** Review OpenAI Apps SDK documentation for these fields

## Testing Checklist

After optimizing metadata:

- [ ] Test all tools in ChatGPT
- [ ] Verify loading messages appear correctly
- [ ] Verify success messages are accurate
- [ ] Test with screen reader
- [ ] Verify keyboard navigation
- [ ] Check that error messages are helpful
- [ ] Ensure widgets render at appropriate sizes

## Example: Optimized Metadata

```typescript
// Before
{
  id: "pizza-map",
  title: "Show Pizza Map",
  invoking: "Hand-tossing a map",
  invoked: "Served a fresh map",
  // ...
}

// After
{
  id: "pizza-map",
  title: "Show Pizza Map",
  invoking: "Finding pizza places near you...",
  invoked: "Found pizza places in your area",
  // ...
  _meta: {
    "openai/outputTemplate": "ui://widget/pizza-map.html",
    "openai/toolInvocation/invoking": "Finding pizza places near you...",
    "openai/toolInvocation/invoked": "Found pizza places in your area",
    "openai/widgetAccessible": true,
    "openai/resultCanProduceWidget": true,
    "openai/contentType": "location-map",
    "openai/preferredHeight": "500px",
  },
  annotations: {
    destructiveHint: false,
    openWorldHint: false,
    readOnlyHint: true,
  }
}
```

## Benefits of Optimized Metadata

1. **Better UX** - Clear, contextual messages
2. **Accessibility** - Usable by everyone
3. **Performance** - Proper caching and sizing hints
4. **Error Handling** - Helpful error messages
5. **ChatGPT Integration** - Better understanding of tool capabilities

## Resources

- [OpenAI Apps SDK Documentation](https://platform.openai.com/docs/guides/apps-sdk)
- [MCP Specification](https://modelcontextprotocol.io/)
- [WCAG Accessibility Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)

## Priority

- **High:** Loading/success messages, accessibility, annotations
- **Medium:** Error handling, content metadata
- **Low:** Performance hints (pending research)

---

**Status:** TODO - Metadata not yet optimized
**Priority:** Medium (improves UX but not blocking)
**Estimated Effort:** 2-4 hours
