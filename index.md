---
# NOTE: This index.md renders the root README.md on GitHub Pages
layout: null
title: Home
---

<style>
  :root {
    --bg: #0b0c10;
    --panel: #111318;
    --text: #e6e6e6;
    --muted: #b9c0cb;
    --brand: #4ea1ff;
    --brand-strong: #2f8cff;
    --border: #1d2330;
    --code-bg: #0f1117;
    --code-border: #232a3a;
  }

  html { scroll-behavior: smooth; }
  body {
    margin: 0;
    background: var(--bg);
    color: var(--text);
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", sans-serif;
    line-height: 1.6;
  }

  .doc-container {
    max-width: 900px;
    margin: 0 auto;
    padding: 32px 20px 80px;
  }

  /* Panels for README sections rendered from Markdown */
  .doc-container > *:not(:first-child) {
    margin-top: 28px;
  }

  h1, h2, h3 { line-height: 1.25; }
  h1 { font-size: 2rem; margin: 0 0 0.75rem; }
  h2 { font-size: 1.5rem; margin: 2rem 0 0.5rem; padding-top: 0.5rem; border-top: 1px solid var(--border); }
  h3 { font-size: 1.25rem; margin: 1.25rem 0 0.5rem; }

  p { color: var(--text); margin: 0.5rem 0 1rem; }
  strong { color: #fff; }

  a { color: var(--brand); text-decoration: none; }
  a:hover { text-decoration: underline; color: var(--brand-strong); }

  ul, ol { padding-left: 1.25rem; }
  li { margin: 0.25rem 0; }

  blockquote {
    margin: 1rem 0;
    padding: 0.75rem 1rem;
    background: var(--panel);
    border-left: 4px solid var(--brand);
    color: var(--muted);
  }

  table { border-collapse: collapse; width: 100%; margin: 1rem 0; }
  th, td { border: 1px solid var(--border); padding: 0.5rem 0.75rem; }
  th { background: #0d1117; text-align: left; }
  tr:nth-child(even) td { background: #0c1016; }

  code, pre { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; }
  code { background: rgba(255,255,255,0.04); padding: 0.15rem 0.35rem; border-radius: 6px; border: 1px solid var(--border); }
  pre {
    background: var(--code-bg);
    border: 1px solid var(--code-border);
    border-radius: 10px;
    padding: 14px 16px;
    overflow: auto;
  }
  pre > code { background: transparent; border: 0; padding: 0; }

  hr { border: 0; border-top: 1px solid var(--border); margin: 2rem 0; }

  /* Nice callout boxes if you use > [!NOTE] style later */
  .callout { border: 1px solid var(--border); border-left: 4px solid var(--brand); background: var(--panel); padding: 12px 14px; border-radius: 8px; }
</style>

<main class="doc-container">
{% include_relative README.md %}

{% include_relative upscale-quickstart.md %}
</main>
