---
name: feature-playbook
description: End-to-end pipeline for building a new feature the proven way — grill → shell → data spine → perceived-speed → single ship. Instantiate the project's existing primitives instead of reinventing them. Trigger: /feature-playbook
trigger: /feature-playbook
---

# /feature-playbook

The repeatable recipe for any non-trivial new feature, so it inherits the same
architecture, design system, and perf model as everything already built. The core
idea: **a mature project has primitives you instantiate, not code you rewrite** —
find them before writing anything new.

Originally derived from a feature that shipped clean end to end. This is the
project-agnostic version; if the current project has its own
`.claude/skills/feature-playbook/`, that one wins — it will name the actual primitives.

## When to use
- A new domain concept / entity / role
- Anything touching multiple modules, or with a two-mode either/or split
- **Skip for:** bug fixes, copy tweaks, single-component changes (those go straight to code)

## Usage
```
/feature-playbook <feature name>        # start at Phase 0
/feature-playbook <feature> --phase 2   # resume at a specific phase
```

---

## The pipeline

### Phase 0 — Align (NO code)
Run **`/grill-with-docs`**. It interviews to surface decisions and resolve ambiguity.
Outputs, before any code:
- Domain language locked in the project's domain doc (**`CONTEXT.md`** or equivalent) — a named entry per new concept, each with an `_Avoid_` anti-pattern.
- An **ADR** in `docs/adr/NNNN-slug.md` for every hard-to-reverse decision (schema, deletion behavior, status semantics, role model). Increment from the last file.

**Gate:** do not write code until the glossary is settled and the headline decisions are ADR'd. Undocumented decisions become invisible debt.
**Model:** light model to draft candidates, Opus to lock.

### Phase 1 — Shell on the existing primitives (Opus)
- **Route architecture = index + sub-routes** — never tabs or modal-tabs for a multi-category feature. The sub-routes ARE the category/type field. Index = grid of cards; each sub-page standalone, owning its own subscriptions and state; back button → index.
- **Reuse the project's design idiom** rather than inventing a look for this feature. Read how the nearest existing surface is styled and match it.
- **Two-mode split? Reuse the established switcher.** Where a signature interaction already exists for binary either/or, instantiate it — don't design a second one. (Give each instance its own shared-layout animation id, or the instances will animate into each other.)
- **Derive, don't precompute.** Keep server-side compute out until a genuine cross-user or server-only signal requires it.
- Back buttons on the **LEFT** of page headers, before the title.
- Check the project's **component registry first** before writing custom UI; extend installed components by composition, never edit them in place.

### Phase 2 — One data spine
- A single **`xService.ts`** matching the project's existing service idiom: plain functions over the client, realtime listeners that degrade gracefully to an empty callback on error, merge-style writes, and a sanitizer before every write (many stores reject `undefined`).
- **Subscribe ONCE in the layout** via an `XDataProvider`. The layout stays mounted across navigation, so detail pages read from the warm in-memory list instead of firing a per-id fetch. Key any per-entity map by id for correctness across navigation; expose `…Loaded` flags and gate mount-sensitive children on them.
- Match the project's existing timestamp convention rather than introducing a second one.
- Editing security rules? Activate the rules-auditor skill. Anchor ownership on a stored owner-id field; validate shape, not client-trusted honesty.

### Phase 3 — Perceived-speed layer
- **Layout-matched skeletons** that mirror the real DOM — never a bare centered spinner for a page whose layout is already known.
- **Hover-prefetch** on cards that navigate programmatically (imperative navigation, unlike a link component, does not prefetch): prefetch on `onMouseEnter` + `onFocus`.
- **Optimized image component** for any remote media, with explicit sizing.
- Check whether **offline/local persistence** is already enabled globally — if so, repeat reads and resubscriptions are free.

### Phase 4 — Ship once
Run **`/ship`**: review → docs → lint → build (user-verified) → infra deploy if rules changed → single push → verify. **One ship at the end** — a feature with no user value until complete ships as a unit, not in half-states. Categorized commits during the build, single push at the end.

---

## Finding the primitives (do this in Phase 1, before writing)
Build a small table for the current project before starting:

| Primitive | Where it lives |
|---|---|
| Two-mode switcher / signature interaction | _find it_ |
| Shared-subscription provider (instant detail pages) | _find it_ |
| Layout-matched skeletons | _find it_ |
| Sandboxed rich-content block | _find it_ |
| Service idiom (CRUD + realtime) | _find it_ |
| Master-detail admin management | _find it_ |

If a row is genuinely empty, that's the one thing this feature legitimately builds new — and it should be built as a reusable primitive, then added to this table.

## Non-negotiables
- **Docs during the session, not after** — every design/UX/architecture decision → domain-doc entry + ADR, same session.
- **Match the model to the step** — light model for drafts/mapping/boilerplate, Opus for locking design, non-trivial execution, and review.
- **Never run the build automatically** — the user verifies manually; use a typecheck + linter to self-check.
- **Reduce human error** — persist drafts, prefer upload over paste, never make the user re-key work.
