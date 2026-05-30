# awesome-remotion-skills — Claude Code Skill Suite

8 production-ready Claude Code skills for building videos with Remotion.

## Skills Included

### Core
- `/remotion-video-creation` — The complete Remotion reference: 3D, audio, captions, charts, transitions, timing, and more (29 rules)

### Platform & Distribution
- `/remotion-social-formats` — TikTok, YouTube, Instagram, LinkedIn dimensions, safe zones, export settings
- `/remotion-lambda` — AWS Lambda rendering for production at scale
- `/remotion-player` — Embed Remotion in React/Next.js with controls and live preview

### Templates & Design
- `/remotion-templates` — Ready-made templates: product demo, explainer, podcast, announcement
- `/remotion-motion-graphics` — Lower thirds, kinetic typography, logo animations, broadcast overlays
- `/remotion-data-stories` — Bar chart races, animated stats, data-driven video storytelling
- `/remotion-brand-kit` — Brand config system, intro/outro, watermark, consistent video identity

## Install

```powershell
# Windows (PowerShell)
$dest = "$env:USERPROFILE\.claude\skills"
Get-ChildItem -Directory | ForEach-Object { Copy-Item $_.FullName $dest -Recurse -Force }
```

```bash
# macOS / Linux
cp -r remotion-* ~/.claude/skills/
```

## Stack
Remotion · React · TypeScript · AWS Lambda · @remotion/player

## Related
Check out [awesome-web-skills](https://github.com/dinaf2026-web/awesome-web-skills) for 14 web development skills.

## Author
[@dinaf2026-web](https://github.com/dinaf2026-web)

## License
MIT
