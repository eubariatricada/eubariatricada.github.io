---
name: mario-blog-publisher
description: >-
  Generate a new post for the eubariatricada.github.io Hugo blog using the
  mario-blog-post-generator skill, land it in blog/content/posts/, and open a
  pull request for it. Use whenever the user asks to "publish a new blog
  post," "write and publish a post about X," "add a post to the blog," or
  gives a topic and wants it to end up live on the site — not just drafted.
  Always ask up front whether PR review is required or optional: if optional,
  this skill merges the PR immediately after opening it, which pushes the
  post live via the site's GitHub Pages deploy; if required, it opens the PR
  and stops, leaving the merge to a human reviewer. Never assume which mode
  the user wants.
compatibility: >-
  Requires git and the GitHub CLI (gh) with push access to
  eubariatricada/eubariatricada.github.io, run from a checkout of that repo.
  Depends on the mario-blog-post-generator skill for drafting — this skill
  does not draft content itself.
metadata:
  author: mario
  version: "1.0"
  category: content-creation
---

# Blog Publisher

Turn an approved draft from `mario-blog-post-generator` into a live post on
eubariatricada.github.io. Generate, land the files, open a PR, and — only
when the user said review is optional — merge it immediately so it deploys.

Work through the phases in order. Do not skip Phase 1's question or Phase 3's
gate.

```
- [ ] Phase 1: Gather inputs (topic/sources, review mode)
- [ ] Phase 2: Generate the post (delegates to mario-blog-post-generator)
- [ ] Phase 3: Land the files + present the plan (GATE: approval)
- [ ] Phase 4: Branch, commit, open PR
- [ ] Phase 5: Merge (optional review) or stop and hand off (required review)
```

## Phase 1 — Gather inputs

Ask for whatever isn't already stated:

| Input | Default | Notes |
|---|---|---|
| Topic / sources | — required | Whatever `mario-blog-post-generator` needs: topic, sources, sort order, etc. Its own Phase 1 will ask for the rest. |
| **PR review mode** | **none — must ask** | "Required" or "optional." This decides whether the post goes live automatically after the PR opens. Getting it wrong means either publishing something nobody reviewed, or leaving a post stuck in review the user expected live — always confirm, never infer from tone or urgency. |

Phrase the question plainly, e.g.: "Once the PR is open, should I wait for
someone to review it, or merge it right away so it publishes live?"

## Phase 2 — Generate the post

Invoke `mario-blog-post-generator` (via the Skill tool) with the topic and
sources from Phase 1. Let it run its own phases in full, including its own
Phase 4 approval gate — do not shortcut that gate just because this skill has
its own gate later. The output of this phase is an approved Markdown draft
(and optionally a top image) sitting wherever that skill saved them.

If the user already has an approved draft in hand (they wrote it themselves,
or generated it earlier in the conversation), skip straight to Phase 3 with
that content instead of re-running the generator.

## Phase 3 — Land the files + present the plan (GATE)

Study the existing convention before writing anything — look at an existing
post under `blog/content/posts/<slug>/` (e.g.
`blog/content/posts/13-motivos-cirurgia-bariatrica/index.md`) for the exact
frontmatter shape, since the theme reads specific fields:

```yaml
draft: false
title: "..."
date: "YYYY-MM-DD"
summary: "..."
description: "..."
toc: true
readTime: true
autonumber: false
math: false
tags: ["...", "..."]
showTags: true
hideBackToTop: false
```

Steps:

1. **Slug**: kebab-case the title (matching existing folder names, e.g.
   `13-motivos-cirurgia-bariatrica`), ASCII only, no accents.
2. **Path**: `blog/content/posts/<slug>/index.md`, plus `capa.png` alongside
   it if the generator produced a top image (rename to `capa.png` — that's
   the convention this repo uses, not `hero.png` or the generator's default
   name).
3. **Frontmatter**: fill every field above from the draft's content — don't
   leave placeholders. `date` is today's date unless the user specifies
   otherwise. `tags` should reflect the post's actual topics.
4. Write the file(s) to a **new local branch** named `post/<slug>` off the
   current default branch (`main`), not directly on the branch you're
   currently on.

**GATE:** Before committing, show the user: the file path(s), the full
frontmatter, and the post body (or a summary of it if it's long). Confirm
this is what should land in the repo — this is the last cheap point to fix a
wrong slug, a missing tag, or frontmatter that doesn't match the theme's
schema (a mismatch here has broken rendering site-wide before — see
`db3e818`). Do not proceed to Phase 4 without explicit go-ahead.

## Phase 4 — Branch, commit, open PR

1. Confirm you're not on `main`; create/switch to `post/<slug>` off `main`.
2. `git add` the new post directory only — never a broad `git add -A`.
3. Commit: `Add post: <title>`.
4. Push the branch: `git push -u origin post/<slug>`.
5. Open the PR with `gh pr create`:
   - base = `main`, head = `post/<slug>`
   - title: `Add post: <title>`
   - body: one or two sentences on what the post covers, plus a line noting
     it was generated via the mario-blog-post-generator and
     mario-blog-publisher skills.

Report the PR number and URL.

## Phase 5 — Merge or hand off

This is where the Phase 1 answer matters:

- **Review required**: Stop here. Report the PR URL and tell the user it's
  waiting for review — do not merge it yourself, even if you're confident in
  the content.
- **Review optional**: Merge immediately with `gh pr merge --squash --delete-branch`.
  This repo deploys to GitHub Pages on push to `main`
  (`baseURL = 'https://eubariatricada.github.io/'`), so a successful merge
  publishes the post live within a few minutes. If the merge is blocked
  (branch protection, required checks, conflicts), do **not** force it —
  report the blocker and the PR URL so the user can resolve or merge
  manually. On success, report the merge commit and the live post URL
  (`https://eubariatricada.github.io/posts/<slug>/`).

## Gotchas

- **The review-mode question has no default.** Silently picking "optional"
  publishes unreviewed content live; silently picking "required" leaves the
  user waiting on a review they didn't ask for. Always ask, every run — don't
  reuse an answer from an earlier post in the same conversation without
  re-confirming.
- **Frontmatter schema is theme-specific, not freeform.** This theme reads
  exact param names (e.g. `breadcrumbs.enabled` as an object, not a bare
  bool, broke the whole site once — `db3e818`). Copy the shape of an existing
  post's frontmatter rather than inventing field names.
- **Image naming**: this repo's convention is `capa.png`, not `hero.png` or
  whatever the generator names it by default — rename before committing.
- **Never commit to `main` directly.** Always branch as `post/<slug>` even
  when review is optional — the PR is what gives you a clean auto-merge point
  and a paper trail, and squash-merging still requires it.
- **Approval in Phase 3 covers content, not the merge decision.** The
  authorization to auto-merge comes from the Phase 1 answer, not from the
  Phase 3 content approval — don't merge on "optional" alone if the user
  changes their mind about content after Phase 3; re-confirm if anything
  about scope shifts after the gate.
- **Slug collisions**: if `blog/content/posts/<slug>` already exists, don't
  silently overwrite it — ask whether this is an edit to the existing post
  (different flow) or needs a disambiguated slug.
