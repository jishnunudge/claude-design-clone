# Design Philosophy Showcases — Asset Index

> 8 scenes × 3 styles = 24 prebuilt design samples
> Used during Phase 3 direction recommendations to show "what this style actually looks like"

## Style Guide

| Code | School | Style Name | Visual Character |
|------|--------|------------|-----------------|
| **Pentagram** | Information Architecture | Pentagram / Michael Bierut | Black-and-white restraint, Swiss grid, strong typographic hierarchy, #E63946 red accent |
| **Build** | Minimalism | Build Studio | Luxury-level whitespace (70%+), subtle weight range (200–600), #D4A574 warm gold, refined |
| **Takram** | Eastern Philosophy | Takram | Soft tech feel, natural palette (beige/grey/green), rounded corners, charts as art |

## Scene Quick Reference

### Content Design Scenes

| # | Scene | Spec | Pentagram | Build | Takram |
|---|-------|------|-----------|-------|--------|
| 1 | WeChat article cover | 1200×510 | `cover/cover-pentagram` | `cover/cover-build` | `cover/cover-takram` |
| 2 | PPT data slide | 1920×1080 | `ppt/ppt-pentagram` | `ppt/ppt-build` | `ppt/ppt-takram` |
| 3 | Vertical infographic | 1080×1920 | `infographic/infographic-pentagram` | `infographic/infographic-build` | `infographic/infographic-takram` |

### Website Design Scenes

| # | Scene | Spec | Pentagram | Build | Takram |
|---|-------|------|-----------|-------|--------|
| 4 | Personal homepage | 1440×900 | `website-homepage/homepage-pentagram` | `website-homepage/homepage-build` | `website-homepage/homepage-takram` |
| 5 | AI directory site | 1440×900 | `website-ai-nav/ainav-pentagram` | `website-ai-nav/ainav-build` | `website-ai-nav/ainav-takram` |
| 6 | AI writing tool | 1440×900 | `website-ai-writing/aiwriting-pentagram` | `website-ai-writing/aiwriting-build` | `website-ai-writing/aiwriting-takram` |
| 7 | SaaS landing page | 1440×900 | `website-saas/saas-pentagram` | `website-saas/saas-build` | `website-saas/saas-takram` |
| 8 | Developer docs | 1440×900 | `website-devdocs/devdocs-pentagram` | `website-devdocs/devdocs-build` | `website-devdocs/devdocs-takram` |

> Each entry has both a `.html` (source) and `.png` (screenshot) file

## Usage Notes

### Referencing During Phase 3 Recommendations
After recommending design directions, show the prebuilt screenshot for the matching scene:
```
"Here's what the Pentagram style looks like for an article cover → [show cover/cover-pentagram.png]"
"This is how Takram style handles a PPT data slide → [show ppt/ppt-takram.png]"
```

### Scene Matching Priority
1. User's requested scene has an exact match → show that scene directly
2. No exact match but similar category → show closest scene (e.g., "product website" → show SaaS landing page)
3. No match at all → skip prebuilt samples, go straight to Phase 3.5 live generation

### Side-by-side Comparison
Showing all 3 styles for the same scene helps users compare visually:
- "This is the same article cover implemented in 3 different styles"
- Display order: Pentagram (rational restraint) → Build (luxurious minimalism) → Takram (soft warmth)

## Content Details

### Article Cover (cover/)
- Content: Claude Code Agent workflow — 8 parallel agent architecture
- Pentagram: Giant red "8" + Swiss grid lines + data bars
- Build: Ultra-light weight "Agent" floating in 70% whitespace + warm gold hairline
- Takram: 8-node radial flow diagram as artwork + beige background

### PPT Data Slide (ppt/)
- Content: GLM-4.7 open-source model coding benchmark breakthrough (AIME 95.7 / SWE-bench 73.8% / τ²-Bench 87.4)
- Pentagram: 260px "95.7" anchor + red/grey/light-grey contrast bar chart
- Build: Three groups of 120px ultra-light numbers floating + warm gold gradient comparison bars
- Takram: SVG radar chart + three-color overlay + rounded data cards

### Vertical Infographic (infographic/)
- Content: AI memory system CLAUDE.md optimized from 93KB to 22KB
- Pentagram: Giant "93→22" numbers + numbered sections + CSS data bars
- Build: Extreme whitespace + soft-shadow cards + warm gold connection lines
- Takram: SVG ring chart + organic curve flow diagram + frosted glass cards

### Personal Homepage (website-homepage/)
- Content: Independent developer Alex Chen's portfolio homepage
- Pentagram: 112px large name + Swiss grid columns + editorial numbers
- Build: Glassmorphism nav + floating stats cards + ultra-light weight
- Takram: Paper texture + small circular avatar + hairline dividers + asymmetric layout

### AI Directory Site (website-ai-nav/)
- Content: AI Compass — 500+ AI tool directory
- Pentagram: Square-cornered search box + numbered tool list + uppercase category labels
- Build: Rounded search box + refined white tool cards + pill tags
- Takram: Organic offset card layout + soft category tags + chart-style connections

### AI Writing Tool (website-ai-writing/)
- Content: Inkwell — AI writing assistant
- Pentagram: 86px large headline + wireframe editor mockup + grid feature columns
- Build: Floating editor card + warm gold CTA + luxurious writing experience
- Takram: Poetic serif headline + organic editor + flow diagram

### SaaS Landing Page (website-saas/)
- Content: Meridian — business intelligence analytics platform
- Pentagram: Black-and-white split layout + structured dashboard + 140px "3x" anchor
- Build: Floating dashboard card + SVG area chart + warm gold gradient
- Takram: Rounded bar chart + flow nodes + soft earth tones

### Developer Docs (website-devdocs/)
- Content: Nexus API — unified AI model gateway
- Pentagram: Left sidebar nav + square-cornered code blocks + red string highlights
- Build: Centered floating code card + soft shadow + warm gold icons
- Takram: Beige code blocks + flow chart connections + dashed feature cards

## File Count

- HTML source files: 24
- PNG screenshots: 24
- Total assets: 48 files

---

**Version**: v1.0
**Created**: 2026-02-13
**Used in**: design-philosophy skill Phase 3 recommendation flow
