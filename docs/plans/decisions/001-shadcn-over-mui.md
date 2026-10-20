# ADR-001 — Replace Material UI with shadcn/ui

**Status:** ✅ Decided & implemented  
**Date:** October 2026  
**Commit:** `f57c9a6`

---

## Context

The admin was initially scaffolded with Material UI (MUI) v6 + `@emotion/react`. MUI is a complete component library with its own design system, theme engine, and ~880 KB bundle.

## Decision

Replace MUI with shadcn/ui primitives built on Radix UI + Tailwind CSS v4.

## Reasons

- **Bundle size**: MUI added ~882 KB to the production bundle. After replacement: ~704 KB (−20%).
- **Customisability**: shadcn/ui copies component source into the project (`src/components/ui/`), making every component fully owned and editable without fighting a theme API.
- **Tailwind v4**: Zero config file — the Vite plugin handles everything. CSS custom properties (`--color-*`) replace runtime theme injection.
- **No runtime overhead**: MUI injects CSS-in-JS at runtime via Emotion. Tailwind generates static CSS at build time.
- **Alignment**: shadcn/ui is the dominant pattern in the React ecosystem in 2025–2026 and is what most contributors will expect.

## Trade-offs

- MUI has more pre-built complex components (data grids, date pickers). We built simple equivalents; complex ones can be added from Radix or headless-ui as needed.
- shadcn/ui components are copied into the repo — updates require re-copying rather than a version bump. Acceptable for a small admin UI.

## What changed

| Removed | Added |
|---|---|
| `@mui/material`, `@mui/icons-material` | `tailwind-merge`, `clsx`, `class-variance-authority` |
| `@emotion/react`, `@emotion/styled` | `@radix-ui/react-dialog`, `react-label`, `react-slot`, `react-alert-dialog` |
| MUI `ThemeProvider` | CSS custom properties in `src/index.css` |
| MUI Icons | `lucide-react` |

Components added to `src/components/ui/`: `button`, `card`, `badge`, `input`, `label`, `dialog`, `table`, `alert`.
