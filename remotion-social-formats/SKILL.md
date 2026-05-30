---
name: remotion-social-formats
description: Platform-specific video format patterns for Remotion — exact dimensions, safe zones, frame rates, and export settings for TikTok, YouTube, Instagram Reels, LinkedIn, Twitter/X, and YouTube Shorts.
origin: community
tags: [remotion, social-media, tiktok, youtube, instagram, linkedin, formats, export]
---

# remotion-social-formats

Platform-specific video format patterns for Remotion. Exact dimensions, safe zones, frame rates, and export settings for every major social platform.

---

## 1. Platform Dimensions Reference Table

| Platform | Width | Height | FPS | Max Duration | Max File Size | Aspect Ratio |
|---|---|---|---|---|---|---|
| TikTok | 1080 | 1920 | 30 | 10 min | 4 GB | 9:16 |
| YouTube (Standard) | 1920 | 1080 | 24 / 30 / 60 | 12 hours | 256 GB | 16:9 |
| YouTube Shorts | 1080 | 1920 | 30 | 60 sec | 256 MB | 9:16 |
| Instagram Reels | 1080 | 1920 | 30 | 90 sec | 4 GB | 9:16 |
| Instagram Square | 1080 | 1080 | 30 | 60 sec | 4 GB | 1:1 |
| Instagram Landscape | 1080 | 566 | 30 | 60 sec | 4 GB | 1.91:1 |
| LinkedIn | 1920 | 1080 | 30 | 10 min | 5 GB | 16:9 |
| Twitter / X | 1280 | 720 | 30 | 2 min 20 sec | 512 MB | 16:9 |

---

## 2. Composition Setup per Platform

### Platform constants

```ts
// src/platforms.ts

export type Platform =
  | 'tiktok'
  | 'youtube'
  | 'youtube-shorts'
  | 'instagram-reels'
  | 'instagram-square'
  | 'instagram-landscape'
  | 'linkedin'
  | 'twitter';

export interface PlatformSpec {
  width: number;
  height: number;
  fps: number;
  maxDurationSeconds: number;
  maxFileSizeMB: number;
  label: string;
}

export const PLATFORMS: Record<Platform, PlatformSpec> = {
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    maxDurationSeconds: 600,
    maxFileSizeMB: 4096,
    label: 'TikTok',
  },
  youtube: {
    width: 1920,
    height: 1080,
    fps: 30,
    maxDurationSeconds: 43200,
    maxFileSizeMB: 262144,
    label: 'YouTube',
  },
  'youtube-shorts': {
    width: 1080,
    height: 1920,
    fps: 30,
    maxDurationSeconds: 60,
    maxFileSizeMB: 256,
    label: 'YouTube Shorts',
  },
  'instagram-reels': {
    width: 1080,
    height: 1920,
    fps: 30,
    maxDurationSeconds: 90,
    maxFileSizeMB: 4096,
    label: 'Instagram Reels',
  },
  'instagram-square': {
    width: 1080,
    height: 1080,
    fps: 30,
    maxDurationSeconds: 60,
    maxFileSizeMB: 4096,
    label: 'Instagram Square',
  },
  'instagram-landscape': {
    width: 1080,
    height: 566,
    fps: 30,
    maxDurationSeconds: 60,
    maxFileSizeMB: 4096,
    label: 'Instagram Landscape',
  },
  linkedin: {
    width: 1920,
    height: 1080,
    fps: 30,
    maxDurationSeconds: 600,
    maxFileSizeMB: 5120,
    label: 'LinkedIn',
  },
  twitter: {
    width: 1280,
    height: 720,
    fps: 30,
    maxDurationSeconds: 140,
    maxFileSizeMB: 512,
    label: 'Twitter / X',
  },
};
```

### Register all compositions

```ts
// src/Root.tsx
import { Composition } from 'remotion';
import { PLATFORMS, Platform } from './platforms';
import { SocialVideo } from './compositions/SocialVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      {(Object.entries(PLATFORMS) as [Platform, typeof PLATFORMS[Platform]][]).map(
        ([id, spec]) => (
          <Composition
            key={id}
            id={id}
            component={SocialVideo}
            width={spec.width}
            height={spec.height}
            fps={spec.fps}
            durationInFrames={spec.fps * 15} // override per project
            defaultProps={{ platform: id }}
          />
        )
      )}
    </>
  );
};
```

---

## 3. Safe Zone System

Safe zones prevent important content from being cropped by UI overlays (TikTok bottom bar, YouTube controls, platform chrome).

### Safe zone constants

```ts
// src/safeZones.ts
import { Platform } from './platforms';

export interface SafeZone {
  top: number;
  bottom: number;
  left: number;
  right: number;
}

// All values in pixels at native resolution
export const SAFE_ZONES: Record<Platform, SafeZone> = {
  tiktok: { top: 130, bottom: 300, left: 24, right: 24 },
  youtube: { top: 40, bottom: 40, left: 60, right: 60 },
  'youtube-shorts': { top: 130, bottom: 300, left: 24, right: 24 },
  'instagram-reels': { top: 130, bottom: 250, left: 24, right: 24 },
  'instagram-square': { top: 24, bottom: 24, left: 24, right: 24 },
  'instagram-landscape': { top: 16, bottom: 16, left: 24, right: 24 },
  linkedin: { top: 40, bottom: 40, left: 60, right: 60 },
  twitter: { top: 24, bottom: 24, left: 40, right: 40 },
};

export const getSafeZone = (platform: Platform): SafeZone => SAFE_ZONES[platform];
```

### Safe zone overlay component (dev only)

```tsx
// src/components/SafeZoneOverlay.tsx
import React from 'react';
import { Platform } from '../platforms';
import { getSafeZone } from '../safeZones';

interface Props {
  platform: Platform;
  width: number;
  height: number;
  /** Only render in development — pass process.env.NODE_ENV */
  env?: string;
}

export const SafeZoneOverlay: React.FC<Props> = ({ platform, width, height, env }) => {
  if (env === 'production') return null;

  const zone = getSafeZone(platform);

  return (
    <div
      style={{
        position: 'absolute',
        inset: 0,
        pointerEvents: 'none',
        zIndex: 9999,
      }}
    >
      {/* Safe area border */}
      <div
        style={{
          position: 'absolute',
          top: zone.top,
          bottom: zone.bottom,
          left: zone.left,
          right: zone.right,
          border: '2px dashed rgba(0, 255, 128, 0.6)',
          boxSizing: 'border-box',
        }}
      />

      {/* Danger zone overlays */}
      {/* Top */}
      <div style={{
        position: 'absolute',
        top: 0, left: 0, right: 0,
        height: zone.top,
        background: 'rgba(255, 0, 0, 0.12)',
      }} />
      {/* Bottom */}
      <div style={{
        position: 'absolute',
        bottom: 0, left: 0, right: 0,
        height: zone.bottom,
        background: 'rgba(255, 0, 0, 0.12)',
      }} />
      {/* Left */}
      <div style={{
        position: 'absolute',
        top: zone.top, bottom: zone.bottom,
        left: 0,
        width: zone.left,
        background: 'rgba(255, 165, 0, 0.12)',
      }} />
      {/* Right */}
      <div style={{
        position: 'absolute',
        top: zone.top, bottom: zone.bottom,
        right: 0,
        width: zone.right,
        background: 'rgba(255, 165, 0, 0.12)',
      }} />

      {/* Label */}
      <div style={{
        position: 'absolute',
        top: zone.top + 8,
        left: zone.left + 8,
        color: 'rgba(0, 255, 128, 0.9)',
        fontFamily: 'monospace',
        fontSize: Math.min(width, height) * 0.018,
        background: 'rgba(0,0,0,0.5)',
        padding: '2px 6px',
        borderRadius: 4,
      }}>
        {platform} safe zone
      </div>
    </div>
  );
};
```

---

## 4. Dynamic Platform Switcher

One composition that switches aspect ratios and layouts via props. Use this during development to preview all formats from a single component.

```tsx
// src/compositions/SocialVideo.tsx
import React from 'react';
import { useVideoConfig } from 'remotion';
import { Platform, PLATFORMS } from '../platforms';
import { SafeZoneOverlay } from '../components/SafeZoneOverlay';
import { getSafeZone } from '../safeZones';

interface Props {
  platform: Platform;
}

export const SocialVideo: React.FC<Props> = ({ platform }) => {
  const { width, height } = useVideoConfig();
  const spec = PLATFORMS[platform];
  const zone = getSafeZone(platform);
  const isVertical = height > width;

  return (
    <div style={{ width, height, position: 'relative', background: '#0a0a0a', overflow: 'hidden' }}>

      {/* Your video content goes here — positioned inside safe zone */}
      <div
        style={{
          position: 'absolute',
          top: zone.top,
          bottom: zone.bottom,
          left: zone.left,
          right: zone.right,
          display: 'flex',
          flexDirection: 'column',
          alignItems: 'center',
          justifyContent: isVertical ? 'flex-start' : 'center',
          gap: 24,
          paddingTop: isVertical ? 40 : 0,
        }}
      >
        {/* Slot for actual content */}
      </div>

      {/* Dev overlay — remove in prod */}
      <SafeZoneOverlay
        platform={platform}
        width={width}
        height={height}
        env={process.env.NODE_ENV}
      />
    </div>
  );
};
```

### Input props schema (Zod)

```ts
// src/schemas.ts
import { z } from 'zod';

export const SocialVideoSchema = z.object({
  platform: z.enum([
    'tiktok',
    'youtube',
    'youtube-shorts',
    'instagram-reels',
    'instagram-square',
    'instagram-landscape',
    'linkedin',
    'twitter',
  ]),
  titleText: z.string().default('Your Title Here'),
  subtitleText: z.string().default(''),
  accentColor: z.string().default('#00ff80'),
});

export type SocialVideoProps = z.infer<typeof SocialVideoSchema>;
```

---

## 5. Platform-Aware Text Sizing

Font sizes that read well at each resolution, accounting for compression artifacts and mobile viewing distance.

```ts
// src/typography.ts
import { Platform, PLATFORMS } from './platforms';

export interface TextScale {
  /** Display / hero headline */
  display: number;
  /** Section heading */
  heading: number;
  /** Body copy */
  body: number;
  /** Caption / label */
  caption: number;
  /** Minimum legible size — never go below this */
  min: number;
}

// Base scale designed for 1080px wide canvas.
// Values scale proportionally for other widths.
const BASE_WIDTH = 1080;

const BASE_SCALE: TextScale = {
  display: 96,
  heading: 64,
  body: 36,
  caption: 28,
  min: 22,
};

export const getTextScale = (platform: Platform): TextScale => {
  const spec = PLATFORMS[platform];
  const ratio = spec.width / BASE_WIDTH;

  return {
    display: Math.round(BASE_SCALE.display * ratio),
    heading: Math.round(BASE_SCALE.heading * ratio),
    body: Math.round(BASE_SCALE.body * ratio),
    caption: Math.round(BASE_SCALE.caption * ratio),
    min: Math.round(BASE_SCALE.min * ratio),
  };
};

// Convenience hook
// Usage: const { heading, body } = usePlatformText('tiktok');
export const usePlatformText = (platform: Platform): TextScale =>
  getTextScale(platform);
```

### Text component with safe sizing

```tsx
// src/components/PlatformText.tsx
import React from 'react';
import { Platform } from '../platforms';
import { getTextScale } from '../typography';

type TextVariant = 'display' | 'heading' | 'body' | 'caption';

interface Props {
  platform: Platform;
  variant: TextVariant;
  children: React.ReactNode;
  color?: string;
  style?: React.CSSProperties;
}

export const PlatformText: React.FC<Props> = ({
  platform,
  variant,
  children,
  color = '#ffffff',
  style,
}) => {
  const scale = getTextScale(platform);

  return (
    <p
      style={{
        fontSize: scale[variant],
        color,
        margin: 0,
        lineHeight: 1.2,
        fontFamily: 'Inter, system-ui, sans-serif',
        fontWeight: variant === 'display' ? 800 : variant === 'heading' ? 700 : 400,
        letterSpacing: variant === 'display' ? '-0.03em' : 'normal',
        ...style,
      }}
    >
      {children}
    </p>
  );
};
```

---

## 6. Export Config per Platform

### Codec and quality recommendations

```ts
// src/exportConfigs.ts
import { Platform } from './platforms';

export interface ExportConfig {
  codec: 'h264' | 'h265' | 'vp8' | 'vp9' | 'prores';
  crf: number;          // lower = better quality (h264: 18-28 is good)
  videoBitrate: string; // e.g. '8M'
  audioBitrate: string;
  pixelFormat: 'yuv420p' | 'yuv422p' | 'yuv444p';
  scale?: string;       // ffmpeg scale filter if downscaling
}

export const EXPORT_CONFIGS: Record<Platform, ExportConfig> = {
  tiktok: {
    codec: 'h264',
    crf: 18,
    videoBitrate: '15M',
    audioBitrate: '192k',
    pixelFormat: 'yuv420p',
  },
  youtube: {
    codec: 'h264',
    crf: 16,
    videoBitrate: '20M',
    audioBitrate: '320k',
    pixelFormat: 'yuv420p',
  },
  'youtube-shorts': {
    codec: 'h264',
    crf: 18,
    videoBitrate: '15M',
    audioBitrate: '192k',
    pixelFormat: 'yuv420p',
  },
  'instagram-reels': {
    codec: 'h264',
    crf: 18,
    videoBitrate: '15M',
    audioBitrate: '192k',
    pixelFormat: 'yuv420p',
  },
  'instagram-square': {
    codec: 'h264',
    crf: 18,
    videoBitrate: '10M',
    audioBitrate: '192k',
    pixelFormat: 'yuv420p',
  },
  'instagram-landscape': {
    codec: 'h264',
    crf: 18,
    videoBitrate: '8M',
    audioBitrate: '192k',
    pixelFormat: 'yuv420p',
  },
  linkedin: {
    codec: 'h264',
    crf: 18,
    videoBitrate: '10M',
    audioBitrate: '192k',
    pixelFormat: 'yuv420p',
  },
  twitter: {
    // Twitter re-encodes everything; keep file small
    codec: 'h264',
    crf: 22,
    videoBitrate: '5M',
    audioBitrate: '128k',
    pixelFormat: 'yuv420p',
  },
};
```

### renderMedia call per platform

```ts
// src/render.ts
import { renderMedia, selectComposition } from '@remotion/renderer';
import { Platform, PLATFORMS } from './platforms';
import { EXPORT_CONFIGS } from './exportConfigs';
import path from 'path';

export const renderPlatform = async (
  platform: Platform,
  outputDir: string,
  inputProps: Record<string, unknown> = {},
) => {
  const spec = PLATFORMS[platform];
  const config = EXPORT_CONFIGS[platform];
  const outputPath = path.join(outputDir, `${platform}.mp4`);

  const composition = await selectComposition({
    serveUrl: 'http://localhost:3000', // remotion dev server
    id: platform,
    inputProps: { platform, ...inputProps },
  });

  await renderMedia({
    composition,
    serveUrl: 'http://localhost:3000',
    codec: config.codec,
    outputLocation: outputPath,
    inputProps: { platform, ...inputProps },
    videoBitrate: config.videoBitrate,
    audioBitrate: config.audioBitrate,
    pixelFormat: config.pixelFormat,
    crf: config.crf,
    enforceAudioTrack: false,
    concurrency: 4,
    onProgress: ({ progress }) => {
      process.stdout.write(`\r[${spec.label}] ${Math.round(progress * 100)}%`);
    },
  });

  console.log(`\n[${spec.label}] saved to ${outputPath}`);
  return outputPath;
};
```

---

## 7. Thumbnail Generation

Extract a still frame at a specific timestamp for each platform.

```ts
// src/thumbnails.ts
import { renderStill, selectComposition } from '@remotion/renderer';
import { Platform, PLATFORMS } from './platforms';
import path from 'path';

interface ThumbnailOptions {
  platform: Platform;
  /** Time in seconds to extract the frame from */
  atSecond?: number;
  outputDir: string;
  inputProps?: Record<string, unknown>;
}

export const generateThumbnail = async ({
  platform,
  atSecond = 0,
  outputDir,
  inputProps = {},
}: ThumbnailOptions): Promise<string> => {
  const spec = PLATFORMS[platform];
  const outputPath = path.join(outputDir, `${platform}-thumbnail.jpg`);
  const frameNumber = Math.round(atSecond * spec.fps);

  const composition = await selectComposition({
    serveUrl: 'http://localhost:3000',
    id: platform,
    inputProps: { platform, ...inputProps },
  });

  await renderStill({
    composition,
    serveUrl: 'http://localhost:3000',
    output: outputPath,
    frame: frameNumber,
    inputProps: { platform, ...inputProps },
    imageFormat: 'jpeg',
    jpegQuality: 92,
  });

  console.log(`[${spec.label}] thumbnail saved to ${outputPath}`);
  return outputPath;
};

// Generate thumbnails for all platforms at the same relative time
export const generateAllThumbnails = async (
  outputDir: string,
  atSecond = 1,
  inputProps: Record<string, unknown> = {},
) => {
  const platforms = Object.keys(PLATFORMS) as Platform[];
  const results: Record<string, string> = {};

  for (const platform of platforms) {
    results[platform] = await generateThumbnail({
      platform,
      atSecond,
      outputDir,
      inputProps,
    });
  }

  return results;
};
```

---

## 8. Multi-Platform Render Script

Render all formats in one command. Runs platforms in configurable batches to avoid OOM on lower-spec machines.

```ts
// scripts/renderAll.ts
import { renderPlatform } from '../src/render';
import { generateAllThumbnails } from '../src/thumbnails';
import { Platform, PLATFORMS } from '../src/platforms';
import path from 'path';
import fs from 'fs';

// Which platforms to render. Comment out platforms you don't need.
const TARGET_PLATFORMS: Platform[] = [
  'tiktok',
  'youtube',
  'youtube-shorts',
  'instagram-reels',
  'instagram-square',
  'linkedin',
  'twitter',
];

// How many platforms to render concurrently.
// 2 is safe on 16 GB RAM. Reduce to 1 on 8 GB.
const BATCH_SIZE = 2;

const OUTPUT_DIR = path.join(process.cwd(), 'out');

const chunk = <T>(arr: T[], size: number): T[][] =>
  Array.from({ length: Math.ceil(arr.length / size) }, (_, i) =>
    arr.slice(i * size, i * size + size)
  );

const renderAll = async () => {
  fs.mkdirSync(OUTPUT_DIR, { recursive: true });

  const inputProps = {
    titleText: process.env.TITLE ?? 'Your Title',
    subtitleText: process.env.SUBTITLE ?? '',
    accentColor: process.env.ACCENT ?? '#00ff80',
  };

  const batches = chunk(TARGET_PLATFORMS, BATCH_SIZE);
  const results: Record<string, string> = {};

  console.log(`Rendering ${TARGET_PLATFORMS.length} platforms in ${batches.length} batches...\n`);

  for (const [i, batch] of batches.entries()) {
    console.log(`Batch ${i + 1}/${batches.length}: ${batch.join(', ')}`);
    const rendered = await Promise.all(
      batch.map((platform) => renderPlatform(platform, OUTPUT_DIR, inputProps))
    );
    batch.forEach((platform, idx) => {
      results[platform] = rendered[idx];
    });
  }

  console.log('\nGenerating thumbnails...');
  await generateAllThumbnails(OUTPUT_DIR, 1, inputProps);

  console.log('\nAll done. Output files:');
  Object.entries(results).forEach(([platform, filePath]) => {
    const stat = fs.statSync(filePath);
    const sizeMB = (stat.size / 1024 / 1024).toFixed(1);
    console.log(`  ${platform.padEnd(22)} ${sizeMB} MB  ->  ${filePath}`);
  });
};

renderAll().catch((err) => {
  console.error(err);
  process.exit(1);
});
```

### package.json scripts

```json
{
  "scripts": {
    "dev": "remotion studio",
    "render:all": "ts-node scripts/renderAll.ts",
    "render:tiktok": "remotion render tiktok out/tiktok.mp4",
    "render:youtube": "remotion render youtube out/youtube.mp4",
    "render:shorts": "remotion render youtube-shorts out/youtube-shorts.mp4",
    "render:reels": "remotion render instagram-reels out/instagram-reels.mp4",
    "render:square": "remotion render instagram-square out/instagram-square.mp4",
    "render:linkedin": "remotion render linkedin out/linkedin.mp4",
    "render:twitter": "remotion render twitter out/twitter.mp4",
    "thumbnail:all": "ts-node -e \"require('./src/thumbnails').generateAllThumbnails('out')\""
  }
}
```

---

## 9. Anti-Patterns

### Text too close to edges

```tsx
// BAD - text will be hidden under TikTok's username overlay at bottom
<p style={{ position: 'absolute', bottom: 20, left: 20, fontSize: 32 }}>
  Follow for more tips
</p>

// GOOD - respect safe zone bottom (300px on TikTok)
const zone = getSafeZone('tiktok');
<p style={{ position: 'absolute', bottom: zone.bottom + 16, left: zone.left, fontSize: 32 }}>
  Follow for more tips
</p>
```

### Wrong FPS for platform

```ts
// BAD - 24fps on TikTok looks choppy in the app player; 60fps inflates file size beyond limits
<Composition id="tiktok" fps={24} ... />
<Composition id="tiktok" fps={60} ... />

// GOOD - use platform-native fps from the spec
const spec = PLATFORMS['tiktok']; // fps: 30
<Composition id="tiktok" fps={spec.fps} ... />
```

### Oversized output files

```ts
// BAD - uncontrolled CRF on Twitter produces 800 MB files that fail upload
await renderMedia({ codec: 'h264' }); // no crf, no bitrate cap

// GOOD - always set crf AND videoBitrate together; let the lower one win
await renderMedia({
  codec: 'h264',
  crf: 22,              // Twitter: quality ceiling
  videoBitrate: '5M',   // Twitter: hard size ceiling
});
```

### Font size too small after compression

```ts
// BAD - 18px body text at 1920x1080 becomes unreadable after platform re-encode
<p style={{ fontSize: 18 }}>Key takeaway here</p>

// GOOD - use platform text scale; minimum 22px at 1080px wide
const scale = getTextScale('youtube'); // body: 36, min: 22
<p style={{ fontSize: scale.body }}>Key takeaway here</p>
```

### Ignoring vertical vs horizontal layout

```tsx
// BAD - centering works fine for YouTube but leaves dead space top and bottom on TikTok
<div style={{ display: 'flex', alignItems: 'center', justifyContent: 'center' }}>
  <Content />
</div>

// GOOD - shift content toward top on vertical formats to work with the bottom UI
const isVertical = height > width;
<div style={{
  display: 'flex',
  flexDirection: 'column',
  alignItems: 'center',
  justifyContent: isVertical ? 'flex-start' : 'center',
  paddingTop: isVertical ? zone.top + 48 : 0,
}}>
  <Content />
</div>
```

### Skipping the safe zone during development

```tsx
// BAD - removing the overlay to "see the clean design" means you find cropping bugs after upload
<SafeZoneOverlay env="production" /> // always hidden, defeats the purpose

// GOOD - let process.env control it; it auto-hides in production builds
<SafeZoneOverlay platform={platform} width={width} height={height} env={process.env.NODE_ENV} />
```

### Using one composition for all platforms and forcing width/height in CSS

```tsx
// BAD - Remotion's canvas is fixed; CSS scaling creates blurry output
<Composition id="all-platforms" width={1920} height={1080} ... />
// then scaling down for vertical with CSS transform -- don't do this

// GOOD - register a separate Composition per platform with correct native dimensions
// Each renders at pixel-perfect native resolution
```

---

## Quick Reference Cheatsheet

```
Platform           W      H      FPS   Safe T  Safe B  Safe L/R
-----------------------------------------------------------------
TikTok           1080   1920    30    130px   300px   24px
YouTube          1920   1080    30     40px    40px   60px
YouTube Shorts   1080   1920    30    130px   300px   24px
Instagram Reels  1080   1920    30    130px   250px   24px
Instagram Square 1080   1080    30     24px    24px   24px
LinkedIn         1920   1080    30     40px    40px   60px
Twitter / X      1280    720    30     24px    24px   40px
```
