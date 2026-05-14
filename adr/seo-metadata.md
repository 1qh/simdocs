# seo-metadata

## Decision

Per-route metadata via Next 16 `generateMetadata`. Sitemap + robots.txt + structured data. Free learning tool — organic search is the primary discovery funnel.

## Per-route metadata

| Route | Title pattern | Description pattern |
|---|---|---|
| `/` | "MIPS datapath visualizer + Karnaugh map tool" | One-sentence positioning, domain language |
| `/datapath` | "MIPS datapath — interactive 3D visualizer" | Step through MIPS instructions in a 3D datapath |
| `/kmap` | "Karnaugh map — interactive 2D and 3D" | Minimize Boolean functions with toroidal 3D for 5+ vars |
| `/pipeline` | "MIPS pipeline — stage-time diagram with hazards" | Visualize pipelined instruction execution |
| `/compare` | "Compare MIPS instructions side-by-side" | See how two instructions differ in the datapath |
| `/learn/<slug>` | "<page title> — sim" | Per-page description from frontmatter |
| `/s/<hash>` | Generated from snapshot type — "MIPS state shared via sim" / "K-map exercise shared via sim" | Brief state summary |

## OG cards

Dynamic via `ImageResponse` per `OG-IMAGES.md`. Every metadata includes `openGraph.images` + `twitter.card`.

## Structured data (JSON-LD)

For learn pages: `LearningResource` schema
For interactive tools: `SoftwareApplication` schema
For shared snapshots: minimal (`WebPage`)

Schema embedded via `<script type="application/ld+json">` server-rendered.

## Sitemap

`/sitemap.xml` enumerates:
- All static routes
- All learn pages (from MDX content)
- All examples (from MDX content)

Updated at build time. No per-snapshot URL (shares are user-generated, transient discovery surface — not in sitemap).

## robots.txt

```
User-agent: *
Allow: /
Disallow: /api/
Disallow: /me
Disallow: /admin
Disallow: /s/

Sitemap: https://<domain>/sitemap.xml
```

`/s/` excluded — shared snapshots are user-private (anonymous shares still don't need indexing — they're shared links not discovery surfaces).

## Canonical URLs

Every page emits `<link rel="canonical">` pointing at its non-trailing-slash URL.

## hreflang

Single-language English (per `NON-GOALS.md`). hreflang not emitted until trigger fires.

## Banned

- Title-stuffing
- Hidden text for SEO
- Doorway pages
- Spammy backlinks
- Cloaking

## Caught by

- Per-route metadata test: every route emits required tags
- Sitemap test: every published page appears
- robots.txt smoke: deployed URL serves expected directives
- Lighthouse SEO score ≥ 100 (in `adr/perf-budget.md` Lighthouse-CI thresholds)
