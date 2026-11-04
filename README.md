# dsh-mermaid-plugin

English | [中文](#中文)

A DeepSeek Harness (DSH) client plugin that renders ` ```mermaid ` code blocks in chat messages as diagrams — with a diagram/source toggle, light/dark theme following, and source-preserving error fallback.

## Features

- **Auto detection** — finds mermaid code blocks in the chat transcript by code-block banner or first-effective-line matching (leading blank lines, `%%` comments and YAML frontmatter are skipped).
- **20 diagram types** — `flowchart`, `graph TB/TD/BT/RL/LR`, `sequenceDiagram`, `classDiagram`, `stateDiagram(-v2)`, `erDiagram`, `journey`, `gantt`, `pie`, `mindmap`, `timeline`, `gitGraph`, `quadrantChart`, `requirementDiagram`, `c4Context`, `sankey-beta`, `xychart-beta`, `block-beta`, `packet-beta`, `architecture-beta`.
- **Diagram / source toggle** — the original code block stays reachable; the host source is hidden by CSS `:has()` only while a rendered diagram stands in.
- **Theme following** — re-initializes mermaid (`dark` / `default`) and re-renders when the DSH dark-theme attribute changes.
- **Error fallback** — a broken diagram keeps the source visible and shows the mermaid error message instead of swallowing content.
- **Strict sanitization** — mermaid runs with `securityLevel: "strict"` and `htmlLabels: false`; diagram links open through `window.open(..., "noopener,noreferrer")` and form submission is blocked.
- **Streaming-safe rendering** — while the host assistant message is still streaming (`[data-streaming]` ancestor), the source block is left untouched; the diagram is rendered once the message settles.

## Install

- One-click from the [dsh-market](https://github.com/dsh-market/dsh-market) plugin market (Settings → Plugin Market), or
- `dsh plugin add` this repository from GitHub.

Requires DSH Web. No build step and no npm dependencies are needed — `client.js` is a prebuilt bundle (mermaid 11.17.2 plus the plugin glue).

## How it works

```
package.json (dsh.bundle.patch) ──► cordis.patch.yml ──► module "mermaid-chat" in host bundle
      │
      └─ dsh.client: platform web · immediately · inject conversation UI + locale
               │
               └─ client.js ──► window.__ModuleLoader__.load({ id, factory })
                        │
                        └─ apply(ctx): MutationObserver ──► detect ──► lazy-load mermaid ──► render SVG
```

- The host half (`index.js`) is an empty `apply()` by protocol; all rendering lives on the client side.
- UI strings resolve through the injected `locale` service with built-in zh/en fallback dictionaries.
- Plugin disposal disconnects observers, removes the injected style sheet and every surface element.

## Repository layout

| File | Kind | Purpose |
| --- | --- | --- |
| `package.json` | hand-written | plugin manifest: exports, `dsh.bundle.patch`, `dsh.client` options |
| `index.js` | hand-written | host-half entry, empty `apply()` |
| `client.js` | **prebuilt** (~3.4 MB) | mermaid 11.17.2 bundle + plugin logic (registration block at the end of file) |
| `cordis.patch.yml` | hand-written | bundle patch inserting module `mermaid-chat` |
| `locale/zh.json`, `locale/en.json` | hand-written | plugin metadata (title/description) |
| `icon.svg` | hand-written | plugin icon |
| `AGENTS.md` | hand-written | repository guide for agents and developers |

## Development

- Do not reformat `client.js` — it is a prebuilt mermaid bundle plus the plugin registration block at the end (the maintained source for this repository). Edit only that block and keep the minified style.
- Static check (no test/lint scripts exist): `node --check index.js && node --check client.js`
- See [AGENTS.md](AGENTS.md) for architecture notes, conventions and the manual verification checklist.

## License

[MIT](LICENSE)
