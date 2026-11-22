---
name: hermes-agent-engineer
description: "Hermes Agent architecture, profile management, memory configuration, reusable skills, gateway/integration troubleshooting, configuration review, documentation alignment."
version: 1.0.0
author: Everlily Nexus
license: MIT
platforms: [android, linux, termux]
metadata:
  hermes:
    tags: [Hermes Agent, Profiles, Skills, Configuration, Gateways, Integrations]
    related_skills: [termux-linux-engineer, telegram-bot-engineer, automation-manager]
---

# Hermes Agent Engineer

Understands Hermes Agent architecture, manages agent profiles, configures memory, creates and maintains reusable skills, troubleshoots gateways and integrations, reviews configuration changes, and follows the installed Hermes version's documentation.

Installed Hermes version: v0.22.2 (2026.9.24, stable). Official docs: https://hermes-agent.nousresearch.com/docs

## Activation

Reference by name: `hermes-agent-engineer`.

## Responsibilities

- Understand Hermes Agent architecture.
- Manage agent profiles (`hermes profile {list,use,create,delete,describe,show}`).
- Help configure memory.
- Create and maintain reusable skills (SKILL.md format).
- Troubleshoot gateways and integrations.
- Review configuration changes.
- Follow the installed Hermes version's documentation; do not invent unsupported commands or options.
- For third-party agent integrations, identify the target project first, then use the vendor's documented setup flow and verify each artifact and dependency separately.

## Third-Party Agent Integration Workflow

1. Resolve the target project explicitly. If the user says “default project,” use the repository/project already established as the active default; do not scan or modify unrelated sibling projects.
2. Read the vendor's current integration documentation before changing project files. Prefer the vendor CLI's setup/prompt command because it detects the language, package manager, and supported integration path.
3. Install the vendor's official agent skill into the project when the documentation recommends it; retain the generated lock file and review the installed `SKILL.md` before relying on it.
4. Apply the smallest project changes: the documented dependency, a non-secret `.env.example`, and integration code only when the user requested actual application wiring. Never create or populate a secret-bearing `.env` file without the user's key.
5. Verify independently: inspect the diff, confirm the dependency resolves or installs, import/load the package with the project's interpreter, and run the relevant smoke test. Report setup scaffolding separately from an installed and working runtime.
6. If a pre-existing dependency blocks resolution or build, do not remove or silently alter it. Report the blocker, preserve the documented integration changes only when they are valid, and avoid claiming the integration is operational.

### Integration pitfalls

- Run setup commands from the target project root; vendor CLIs commonly install project-scoped skills and lock files relative to the current directory.
- Treat a successful skill installation, dependency declaration, package installation, and working API integration as separate acceptance criteria; a successful earlier step does not prove the later ones.
- Keep API keys in ignored local environment files and expose only an empty variable in `.env.example`, because setup documentation should be shareable without leaking credentials.
- Prefer the official SDK major version and namespace-based API shape documented by the vendor; do not invent legacy parameter names or claim support from an unverified wrapper.

## Project Archive Workflow

Use this workflow when creating a durable project archive inside an existing Hermes profile:

1. Inspect before changing anything: run `pwd`, `hermes --version`, `hermes profile list`, and `hermes profile describe <active-profile>`/`show <active-profile>`; enumerate the active profile directory and inspect only relevant memory files.
2. Resolve naming explicitly: distinguish the requested agent name, requested profile name, and the actually active Hermes profile. If they differ, preserve the active profile and record the discrepancy; do not rename or create a profile unless Captain Abinav authorizes it.
3. Choose a project-local root under the active profile's `projects/` directory, and create only new archive subdirectories. Never place project records in Hermes configuration, credential, runtime, or memory files unless that separate change is explicitly requested.
4. Create a small class-level structure with a README, topical folders, an index, operating rules, and only the records supplied or verified by the owner. Use Markdown for human-maintained archival material.
5. Write complete files with the file-writing tool, then independently verify the archive file list, key file contents, and preservation of pre-existing sentinel files such as `config.yaml`, `SOUL.md`, and profile memory files.
6. Report the exact archive root, current profile facts, relevant existing memory paths, files created, verification results, and any authorization-blocked naming or configuration decision.

### Archive pitfalls

- Inspect the active profile before choosing a destination — an assumed profile name can send durable records into the wrong Hermes profile.
- Treat a requested agent/profile rename as a configuration change requiring authorization — silently aligning names can disrupt the existing active profile.
- Keep historical records, project documentation, and Hermes operational state separate — mixing them makes future updates harder to audit and risks overwriting configuration.
- Record only owner-supplied or independently verified history — a plausible connective event is still fabricated history.
- Verify after writes with a file listing and sentinel checks — a successful write response confirms the new file, not preservation of unrelated state.

## Safety Guidelines

- Verify against official docs before suggesting config changes.
- Do not invent unsupported Hermes commands or configuration options.
- Request confirmation before destructive operations (profile delete, config overwrite, or profile rename).

## Host Gateway (Multiplexing) Setup and Troubleshooting

**Rule: Exactly one gateway per host.** Hermes does not support per-profile gateways — starting a second gateway double-binds platform pollers (bot tokens, ports). Every profile is served by the single host gateway running from the default profile.

### Fix "profile does not get a gateway of its own"

1. Confirm state: `hermes gateway list` shows profiles as `not running`; the affected profile may appear to run its own gateway.
2. Install/start the host gateway from the default profile: `hermes -p default gateway run` (run in background on Termux; `hermes gateway install` is not supported on Termux — "run manually").
3. Verify: `hermes gateway list` should show the affected profile as `✓ — served by the default multiplexer`; `hermes gateway status` should say "Gateway is running via the default-profile multiplexer".

### Alternative: fold existing per-profile gateways
`hermes gateway migrate --multiplex` converges all secondary profile gateways onto the one host gateway. Runs a preflight first and is safe to re-run.

### Temporary compatibility shim
Set `gateway.standalone: true` in the profile's `config.yaml`, then wait for the host gateway to rescan (<=30s) or send its rescan-profiles control verb. This is temporary — plan to fold with `hermes gateway migrate --multiplex`.

### Termux-specific notes
- `hermes gateway install` returns "Gateway service installation is not supported on Termux. Run manually: hermes gateway" — start with `hermes -p default gateway run &` instead.
- Keep the gateway process running in a persistent terminal session for Telegram/WhatsApp/cron dispatch across profiles.
- Optional dependency build failures (`nemo_relay`, `aiohttp`, `jiter` needing Rust) in gateway logs are non-fatal to core gateway operation; WhatsApp may need `hermes pm install --extra sms`.

### What is NOT a fix
- Do NOT run `hermes -p everlily gateway install --force` unless the profile is on a separate UNIX user or HERMES_HOME outside profiles/.
- Do not install a separate per-profile gateway on the same host — it will double-bind platforms.

## Example

```
User: "Create the everlily profile and add skills."
Everlily: Use `hermes profile create everlily --clone`, then write SKILL.md files into ~/.hermes/profiles/everlily/skills/<name>/.
```
