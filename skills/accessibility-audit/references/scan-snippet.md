<!-- overlay: vscode — add — the page script the scanner runs; tested against axe-core 4.10.2 on a page with known violations -->

# Scan snippet

Run this function on each page through the browser tool's evaluate call
(for the Playwright MCP server, `browser_navigate` to the plain URL, wait
about 3 seconds for the page to settle, then `browser_evaluate` with this
function). Use it verbatim. It loads axe-core, runs only the WCAG 2.0/2.1
A and AA rules, and returns a compact result. Keep the result small: the
first 5 nodes per violation plus the full node count is enough for pooling
and the report.

```js
async () => {
  if (!window.axe) {
    await new Promise((resolve, reject) => {
      const s = document.createElement('script');
      s.src = 'https://cdnjs.cloudflare.com/ajax/libs/axe-core/4.10.2/axe.min.js';
      s.onload = resolve;
      s.onerror = () => reject(new Error('axe-core failed to load (likely CSP)'));
      document.head.appendChild(s);
    });
  }
  const r = await axe.run(document, { runOnly: { type: 'tag', values: ['wcag2a','wcag2aa','wcag21a','wcag21aa'] } });
  return {
    url: r.url,
    violations: r.violations.map(v => ({
      id: v.id, impact: v.impact, helpUrl: v.helpUrl,
      tags: v.tags.filter(t => /^wcag/.test(t)),
      nodes: v.nodes.slice(0, 5).map(n => { const tag = (n.html.match(/^<[^>]*>/) || [n.html])[0]; return { target: n.target, html: tag.length > 400 ? tag.slice(0, 397) + '…>' : tag }; }),
      nodeCount: v.nodes.length
    })),
    incompleteCount: r.incomplete.length
  };
}
```

**If it throws "axe-core failed to load"**, the page's Content Security
Policy blocked the script. Follow `wcag-violations.md` → "If injection
fails": retry once, and if it's site-wide, say so in the report rather
than switching engines.

When pooling, use `nodeCount` (not the length of `nodes`) as the
instance count.
