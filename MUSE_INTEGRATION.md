# Running SecondMind Independently or Inside Meta Muse

`index.html` is deliberately self-contained: no package manager, framework, external font, image, or network dependency is required for the prototype.

## Independent app

Open `index.html` in any modern browser or serve the repository as static files:

```bash
python3 -m http.server 8080
```

Then open `http://localhost:8080`.

The independent build uses a deterministic local demo reasoner so every interaction works without an API key. Replace the adapter described below to connect a production model.

## Embedded Muse mode

Load the app with:

```text
index.html?embed=muse
```

Embedded mode removes the standalone chrome, identifies itself to the parent, accepts user-approved context, and emits response events.

### Host-to-SecondMind messages

```javascript
iframe.contentWindow.postMessage({
  type: "secondmind:set-context",
  payload: {
    source: "approved-message-selection",
    text: "Context explicitly shared by the user",
    retention: "transient"
  }
}, "*");

iframe.contentWindow.postMessage({
  type: "secondmind:open-mode",
  payload: { mode: "view" }
}, "*");
```

Supported modes:

- `listen`
- `view`
- `challenge`
- `decide`
- `wisdom`
- `secondlook`

### SecondMind-to-host messages

```javascript
window.addEventListener("message", event => {
  if (event.data?.type === "secondmind:ready") {
    // The embedded skill is ready for approved context.
  }

  if (event.data?.type === "secondmind:response") {
    // Render, log or evaluate the response according to host policy.
  }
});
```

## Direct JavaScript adapter

When the app runs in the same window, use:

```javascript
SecondMindApp.setContext({
  source: "user-selected-text",
  text: "Only the context the user approved",
  retention: "transient"
});

SecondMindApp.openMode("challenge");
const state = SecondMindApp.getState();
```

## Connecting a Meta model

Provide a host adapter before SecondMind processes a message:

```javascript
window.SecondMindHost = {
  async reason(request) {
    // Call a trusted backend. Never expose a Meta Model API key in index.html.
    const response = await fetch("/api/secondmind/reason", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(request)
    });

    if (!response.ok) throw new Error("Reasoning request failed");
    return response.json();
  }
};
```

Expected response shape:

```json
{
  "kind": "analysis",
  "message": "Here is another credible interpretation.",
  "fact": "What is supported by the provided context.",
  "assumption": "What the user may be inferring.",
  "unknown": "What remains unclear.",
  "alt": "A plausible alternative interpretation.",
  "next": "One optional, proportional next step."
}
```

## Production requirements

The prototype is intentionally not a production mental-health product. A production implementation requires:

- Server-side model credentials
- Data minimization and redaction
- Origin checks instead of wildcard `postMessage` targets
- Authentication and authorization
- Explicit retention and deletion controls
- Safety evaluation and adversarial testing
- Human-reviewed escalation behavior
- Consent controls for third-party conversation data
- Product and legal review for every connected application

