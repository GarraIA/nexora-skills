# nexora-skills

Official skills for [Nexora AI](https://github.com/GarraIA/nexora-ai). Each top-level folder is one skill.
Nexora reads skills **only** from this repository; a user installs a skill explicitly from the app's
Settings → Skills. Skills are **declarative data** — instructions and metadata, never executable code.

## Layout

```
nexora-skills.json   # the registry: the allow-list of installable skills
web/
  SKILL.md           # the skill's instructions (injected into the planner, capped at 8000 chars)
```

## The registry (`nexora-skills.json`)

```json
{
  "version": 1,                    // registry format — must stay 1
  "skills": [
    {
      "id": "web",                 // unique, [a-z0-9-]
      "path": "web",               // folder holding SKILL.md
      "title": "AAA Website",
      "description": "…",          // one line shown in the app
      "icon": "globe",             // lucide icon name
      "runtime": "web",            // "planner" (inject instructions) or "web" (built-in site builder)
      "version": "1.1.0",          // the skill's own semver; bump it on every SKILL.md change
      "capabilities": ["text-to-image", "text-generation"],
      "triggers": ["website", "landing page", "site", "página", "sitio web"],  // phrases that auto-suggest the skill (any language)
      "files": [],                 // extra files to fetch alongside SKILL.md (paths under the folder)
      "hashes": { "SKILL.md": "<sha256>" }   // optional integrity map, see below
    }
  ]
}
```

Nexora validates the whole file against `skillManifestSchema` in `src/shared/api.ts` before it installs
anything; an invalid registry means *no* skill is installable.

### `hashes` — file integrity

`hashes` maps a path relative to the skill folder (e.g. `"SKILL.md"`) to the **lowercase hex SHA-256 of
the raw file bytes** exactly as served by `raw.githubusercontent.com`. When a hash is present for a file,
the install verifies it and refuses a mismatch, so a skill cannot silently change under a pinned registry.
Compute it after the file is final and commit both in the same change:

```sh
sha256sum web/SKILL.md
```

### How Nexora pins this repo

Nexora fetches `https://raw.githubusercontent.com/GarraIA/nexora-skills/<ref>/…` and nothing else. The
ref comes from `NEXORA_SKILLS_REF` — a tag or a commit SHA — and defaults to `main` for now. Maintainers
tag releases of this repo; a tag plus `hashes` gives a fully reproducible install.

## The `web` skill — 1.1.0 contract

`web/SKILL.md` steers Nexora's planner into a fixed workflow that the built-in site builder consumes:

- **Imagery**: 1–3 `text-to-image` steps with ids `img1`, `img2`, `img3`; each prompt < 300 characters
  and every prompt ends with the same one-sentence *visual brief* so the images are cohesive.
- **Spec**: exactly one final `text-generation` step depending on all image steps, with any max-tokens
  input set to ≥ 4096 (the runtime enforces that floor). The model must return one bare JSON object —
  no prose, no markdown fences — and write fewer sections rather than truncate.
- **Strict shape**: `title` (2–80), `tagline` (≤ 160), `lang` (BCP-47), `theme` (`bg`/`fg`/`accent` as
  6-digit hex + `font` from a fixed Google Fonts allow-list), `hero` (`headline` 3–120, `sub` ≤ 240,
  `cta` ≤ 40, `image`), `sections` (2–6; kebab-case `id`, `heading` 2–80, `body` 10–600, `image` = step
  id or `""`), `footer` (≤ 120). Images are referenced by step id (`"img1"`); ordinals (`"1"`) are still
  accepted but not canonical.
- **Language**: all copy in the user's language with a matching `lang` (Portuguese ⇒ `pt-BR` and Brazilian
  CTA/price conventions). Never mixed.
- The runtime validates the spec, asks the model to repair up to twice, and **never publishes an invalid
  spec**.

## Authoring a skill

1. Add a folder `your-skill/` with a `SKILL.md`. The body is guidance appended to Nexora's planner
   system prompt (first 8000 characters): describe *how* to build what this skill makes (structure,
   style, which capabilities to use). It steers the plan; it can never widen what the app is allowed to do.
2. Add an entry to `nexora-skills.json`. `runtime: "planner"` is the default — the skill simply injects
   its instructions and biases model selection toward its `capabilities`. `runtime: "web"` opts into
   Nexora's built-in website builder + GitHub publisher.
3. Add `hashes` for every file you ship and bump the skill's `version` whenever a file changes.

## Contributing

- Open a pull request; a maintainer reviews, merges and tags. Do not push tags yourself.
- Skills are **declarative**: instructions and metadata only. No scripts, no executable content, nothing
  Nexora would have to run.
- Because this repo is public and installable, keep skills free of secrets, credentials or tokens, and of
  any content that tells the agent to exfiltrate data, contact third parties, or act outside the user's
  request. Such PRs are closed.
- Keep `SKILL.md` well under the 8000-character injection cap and keep the registry `version` at `1`.

## Security

Nexora fetches only from this repo over HTTPS, validates the registry, verifies `hashes` when present,
size-caps every file, and stores skills read-only under `~/.nexora/skills/`. It never executes skill
files. See Nexora's `SECURITY.md`.
