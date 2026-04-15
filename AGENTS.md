# AGENTS.md

Agent guidance for the ENS documentation site ([docs.ens.domains](https://docs.ens.domains)).
Built with [Vocs](https://vocs.dev/), deployed to Cloudflare Pages.

## Commands

```bash
bun install              # install dependencies
bun run dev              # generate external content, then start dev server
bun run build            # generate external content, then production build
bun run generate         # re-fetch external content only
bun run preview          # preview production build locally
bunx prettier --write .  # format all files
```

**Requirements:** Bun >=1.2.8, Node >=22

## Architecture

```
src/pages/       — MDX documentation content (edit these)
src/components/  — React components used in MDX files
src/data/        — static data + generated JSON (do not hand-edit generated/)
scripts/         — content generation scripts
vocs.config.tsx  — sidebar structure, theme, edit links
functions/api/   — Cloudflare Pages Functions (OG images, analytics proxy)
```

### Content generation (`scripts/`)

Run at build time via `bun run generate`. Three scripts:

- **ensips.ts** — fetches ENSIPs from [ensdomains/ensips](https://github.com/ensdomains/ensips), writes MDX to `src/pages/ensip/`, writes sidebar JSON to `src/data/generated/ensips-sidebar.json`
- **deployments.ts** — fetches contract addresses from [ensdomains/ens-contracts](https://github.com/ensdomains/ens-contracts), writes to `src/data/generated/`
- **dao-proposals.ts** — scans `src/pages/dao/proposals/` for local proposal files, writes sidebar JSON to `src/data/generated/dao-proposals-sidebar.json`

## Generated Files — Do Not Edit Directly

The following paths are auto-generated and must not be modified by hand:

- `src/data/generated/` — JSON produced by `scripts/`
- `src/pages/ensip/` — MDX produced by `scripts/ensips.ts`

Generated files are committed to the repo as a cache to avoid network calls on every dev start.
Delete these directories to force a re-fetch on the next `bun run generate`.

## Validation

There is no test suite. `bun run build` is the only correctness check — it runs TypeScript
compilation and the full Vocs build. All TypeScript and build errors must be resolved before
opening a PR.

## MDX Conventions

- **Frontmatter:** `title` is required. `description` is optional but recommended for SEO.
- **Components:** import explicitly at the top of each MDX file. Available components are in `src/components/`.
- **Callouts:** use GitHub-flavored admonition syntax supported by Vocs:
  ```
  > [!NOTE]
  > [!WARNING]
  > [!IMPORTANT]
  ```
- **Mermaid diagrams:** fenced code blocks with the `mermaid` language tag are rendered as diagrams.

Example frontmatter:

```mdx
---
title: Page Title
description: Optional description for SEO and OG image.
---
```

## Sidebar

Defined in `vocs.config.tsx`. Two sections are injected from generated JSON at startup:

- ENSIPs: `src/data/generated/ensips-sidebar.json`
- DAO Proposals: `src/data/generated/dao-proposals-sidebar.json`

To add a new documentation section:
1. Add an entry to the `sidebar` array in `vocs.config.tsx`.
2. Create the corresponding MDX files under `src/pages/`.

## Cloudflare Pages Functions

`functions/api/` contains:

- `og.tsx` — generates OG images per page using `@cloudflare/pages-plugin-vercel-og`
- `blah/event.ts`, `blah/script.ts` — analytics proxy to avoid ad-blocker interference
- `example/basic-gateway.ts` — CCIP-read gateway example

The Vocs dev server does not run Cloudflare Functions. For full function testing:

```bash
bun run build && bunx wrangler pages dev src/dist
```

## Code Style

Prettier is enforced. Key settings (from `.prettierrc`):

- No semicolons
- Single quotes
- 2-space indentation
- Import order: third-party → `@/` → relative (via `prettier-plugin-sort-imports`)

Run `bunx prettier --write .` before committing.

TypeScript strict mode is enabled (`tsconfig.json`). Do not disable `strict`, `noUnusedLocals`,
or `noUnusedParameters`.

## What Not To Do

- Do not edit `src/data/generated/` or `src/pages/ensip/` by hand.
- Do not commit `bun.lock` changes unless dependency versions actually changed.
- Do not add `console.log` to production components or pages.
- Do not disable TypeScript strict mode or add `@ts-ignore` without a comment explaining why.
- Do not run `bun test` — there is no test suite; use `bun run build` instead.

## context7.json

`context7.json` at the repo root configures [Context7](https://context7.com/) to index
`src/pages/` as the documentation source for the ENS protocol. Do not modify this file
unless the indexed folder structure changes.
