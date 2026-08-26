# nexora-skills

Official skills for [Nexora AI](https://github.com/GarraIA/nexora-ai). Each top-level folder is one skill.
Nexora reads skills **only** from this repository; a user installs a skill explicitly from the app's
Settings → Skills. Skills are **declarative data** — instructions and metadata, never executable code.

## Layout

```
nexora-skills.json   # the registry: the allow-list of installable skills
web/
  SKILL.md           # the skill's instructions (injected into the planner)
```

## The registry (`nexora-skills.json`)

```json
{
  "version": 1,
  "skills": [
    {
      "id": "web",                 // unique, [a-z0-9-]
      "path": "web",               // folder holding SKILL.md
      "title": "AAA Website",
      "description": "…",          // one line shown in the app
      "icon": "globe",             // lucide icon name
      "runtime": "web",            // "planner" (inject instructions) or "web" (built-in site builder)
      "version": "1.0.0",
      "capabilities": ["text-to-image", "text-generation"],
      "triggers": ["website", "landing page"],   // phrases that auto-suggest the skill
      "files": []                  // extra files to fetch alongside SKILL.md (paths under the folder)
    }
  ]
}
```

## Authoring a skill

1. Add a folder `your-skill/` with a `SKILL.md`. The body is guidance appended to Nexora's planner
   system prompt: describe *how* to build what this skill makes (structure, style, which capabilities to
   use). It steers the plan; it can never widen what the app is allowed to do.
2. Add an entry to `nexora-skills.json`. `runtime: "planner"` is the default — the skill simply injects
   its instructions and biases model selection toward its `capabilities`. `runtime: "web"` opts into
   Nexora's built-in website builder + GitHub publisher.
3. Open a PR. Because this repo is public and installable, keep skills free of secrets and of any content
   that tells the agent to exfiltrate data or act outside the user's request.

## Security

Nexora fetches only from this repo over HTTPS, validates the registry, size-caps every file, and stores
skills read-only under `~/.nexora/skills/`. It never executes skill files. See Nexora's `SECURITY.md`.
