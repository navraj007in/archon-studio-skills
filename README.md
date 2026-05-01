# archon-studio-skills

Public skill catalogue for [Archon Studio](https://github.com/navraj007in/archon-studio-skills)'s Workbench. Each skill is a self-contained capability bundle: a system-prompt fragment that teaches the model an artefact shape, plus a declaration of which export formats the artefact supports.

## How it works

Archon Studio's Workbench fetches `manifest.json` from this repo (via jsdelivr CDN, pinned to a tag) on app startup. For each skill listed there, it fetches the corresponding `skills/<id>.json` file, validates it against the local schema, and registers it. Bundled skills that ship with the app stay as the offline fallback; remote skills with the same id replace the bundled body.

```
https://cdn.jsdelivr.net/gh/navraj007in/archon-studio-skills@main/manifest.json
```

## Manifest format

```json
{
  "schemaVersion": 1,
  "releaseTag": "main",
  "skills": [
    { "id": "pitch-deck", "path": "skills/pitch-deck.json" }
  ]
}
```

## Skill format

```json
{
  "id": "pitch-deck",
  "label": "Pitch deck",
  "version": 1,
  "body": "## Active skill: pitch-deck\n\n...markdown...",
  "outputs": [
    { "format": "pdf", "renderer": "pitch-deck-pdf" },
    { "format": "md", "renderer": "markdown-passthrough" }
  ]
}
```

### Renderer keys

A skill's `outputs[].renderer` must match a renderer the host app has registered. Skills are **data**, not code — they declare which renderer to use, never how to render. As of v1 the available renderers are:

| Key | Format | Notes |
|---|---|---|
| `pitch-deck-pdf` | pdf | Landscape A4, one slide per page; requires `kind: 'pitchdeck'` spec |
| `markdown-pdf` | pdf | Portrait A4 wrapper around the rendered README.md |
| `markdown-passthrough` | md | Emits README.md bytes verbatim |
| `card-docx` | docx | Walks a `kind: 'card'` NodeSpec into a Word document |

Skills that declare a renderer the running app version doesn't have are rejected at load time, with a clear error pointing at the missing renderer key.

## Contributing a skill

1. Add `skills/<id>.json` (must validate against the schema above)
2. Add an entry to `manifest.json`
3. Open a PR

## Trust model

Skills feed into the LLM's system prompt. The only sink is the `emit_artefact` tool, whose output is Zod-validated against `ArtefactRootSchema` (NodeSpec card) or `PitchDeckSchema`. A malicious skill can degrade output quality but cannot exfiltrate stage data, write outside the sandboxed output folder, or run arbitrary code.

## Versioning

Pin to a tag in production (`@v1`) for stability. The `main` branch is for development; breaking schema changes go in a major-version bump. Skill-level `version` numbers track skill-internal changes.
