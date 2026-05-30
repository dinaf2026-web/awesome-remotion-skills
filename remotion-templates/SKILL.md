`C:\Users\farha\.claude\skills\remotion-templates\SKILL.md`

The skill is written and saved. Here is what it covers:

**Complete components (full working code):**
- `ProductDemo` — 60s, 1920x1080, screen recording + CalloutAnnotation bubbles + caption bar + title card + outro CTA
- `SocialAnnouncement` — 15s, 9:16 vertical, word-by-word headline stagger + particle burst + eyebrow/CTA

**Abbreviated but pattern-complete components:**
- `PodcastAudiogram` — sine-wave driven waveform bars + cover art + pull quote
- `ExplainerVideo` — numbered step slides in `<Sequence>` blocks + SVG-free progress bar
- `NewsTicker` — seamless scrolling overlay, composable on top of any other template
- `CountdownTimer` — SVG arc ring + per-second scale pop + configurable end message

**Supporting infrastructure:**
- `BrandTheme` interface and `defaultTheme` — one object controls all colors, fonts, and logo
- `TemplateShell` shared wrapper component
- `Root.tsx` registration pattern
- Full TypeScript props reference table for every template
- CLI `--props` override pattern for runtime theming
- Quick start checklist and common animation pattern cheatsheet (stagger, loop-safe, exit fade, Sequence guard)
