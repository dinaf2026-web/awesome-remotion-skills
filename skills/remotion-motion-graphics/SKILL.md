---
name: remotion-motion-graphics
description: Broadcast-quality motion graphics in Remotion — lower thirds, kinetic typography, logo animations, title cards, score bugs, and news-style overlays.
origin: community
tags: [remotion, motion-graphics, lower-thirds, kinetic-typography, broadcast, overlays]
---

# Remotion Motion Graphics

Broadcast-quality motion graphics in Remotion: lower thirds, kinetic typography, logo animations, title cards, score bugs, news-style overlays, animated charts, and particle systems.

---

## 1. Motion Graphics Principles

### Timing Philosophy

Broadcast motion graphics follow strict timing conventions built over decades of television production. Every beat must be intentional.

**Frames vs Seconds (30fps baseline)**

| Duration | Frames (30fps) | Use Case |
|---|---|---|
| 4 frames | ~0.13s | Snap cut accent |
| 8 frames | ~0.27s | Fast wipe/reveal |
| 15 frames | 0.5s | Quick lower third in |
| 20 frames | ~0.67s | Standard element entry |
| 30 frames | 1.0s | Title card reveal |
| 45 frames | 1.5s | Deliberate build |
| 90 frames | 3.0s | Lower third hold minimum |
| 150 frames | 5.0s | Feature title hold |

**Easing Vocabulary**

```ts
import { Easing } from "remotion";

// Broadcast staples
export const EASE_OUT_EXPO = Easing.bezier(0.16, 1, 0.3, 1);
export const EASE_IN_EXPO = Easing.bezier(0.7, 0, 0.84, 0);
export const EASE_IN_OUT_QUART = Easing.bezier(0.76, 0, 0.24, 1);
export const EASE_OUT_BACK = Easing.bezier(0.34, 1.56, 0.64, 1); // slight overshoot
export const EASE_SNAP = Easing.bezier(0.25, 0.46, 0.45, 0.94);

// Utility: clamp interpolate to [0,1]
export function clamp01(v: number): number {
  return Math.min(1, Math.max(0, v));
}
```

### Visual Hierarchy Rules

1. **One primary element per frame** — never compete with two animating things at once
2. **Safe zones** — keep text within 10% margin from edges (broadcast safe)
3. **Z-layering** — background < texture < graphic element < text < accent
4. **Color contrast** — 4.5:1 minimum for on-screen legibility at broadcast quality
5. **Motion direction** — entries from left/bottom feel natural; exits to right/top feel resolved

### Brand Consistency Helper

```ts
export interface BrandSystem {
  primary: string;
  secondary: string;
  accent: string;
  textLight: string;
  textDark: string;
  fontDisplay: string;
  fontBody: string;
  cornerRadius: number;
  logoUrl?: string;
}

export const DEFAULT_BRAND: BrandSystem = {
  primary: "#0A0A0A",
  secondary: "#1A1A2E",
  accent: "#E94560",
  textLight: "#FFFFFF",
  textDark: "#0A0A0A",
  fontDisplay: "Inter",
  fontBody: "Inter",
  cornerRadius: 4,
};
```

---

## 2. Lower Third Component

Three style variants: **Classic** (solid bar), **News** (split-color bar with accent line), **Modern** (glassmorphism).

```tsx
import React from "react";
import {
  useCurrentFrame,
  useVideoConfig,
  interpolate,
  spring,
  AbsoluteFill,
} from "remotion";
import { EASE_OUT_EXPO, EASE_IN_EXPO, BrandSystem, DEFAULT_BRAND } from "./brand";

export type LowerThirdVariant = "classic" | "news" | "modern";

interface LowerThirdProps {
  name: string;
  title: string;
  variant?: LowerThirdVariant;
  brand?: BrandSystem;
  inFrame?: number;
  holdFrames?: number;
  outFrame?: number;
}

const ClassicLowerThird: React.FC<{
  name: string; title: string; brand: BrandSystem;
  progress: number; outProgress: number;
}> = ({ name, title, brand, progress, outProgress }) => {
  const slideX = interpolate(progress, [0, 1], [-120, 0], { extrapolateRight: "clamp", easing: EASE_OUT_EXPO });
  const slideXOut = interpolate(outProgress, [0, 1], [0, -120], { extrapolateRight: "clamp", easing: EASE_IN_EXPO });
  const opacity = interpolate(progress, [0, 0.3], [0, 1], { extrapolateRight: "clamp" });
  const opacityOut = interpolate(outProgress, [0.7, 1], [1, 0], { extrapolateRight: "clamp" });
  return (
    <div style={{ position: "absolute", bottom: 120, left: 80, transform: `translateX(${slideX + slideXOut}%)`, opacity: opacity * (1 - opacityOut) }}>
      <div style={{ width: 4, height: "100%", background: brand.accent, position: "absolute", left: 0, top: 0 }} />
      <div style={{ background: brand.primary, paddingLeft: 20, paddingRight: 32, paddingTop: 12, paddingBottom: 12, borderLeft: `4px solid ${brand.accent}` }}>
        <div style={{ fontFamily: brand.fontDisplay, fontWeight: 700, fontSize: 32, color: brand.textLight, letterSpacing: "-0.02em", lineHeight: 1.1 }}>{name}</div>
        <div style={{ fontFamily: brand.fontBody, fontWeight: 400, fontSize: 18, color: brand.accent, marginTop: 4, letterSpacing: "0.04em", textTransform: "uppercase" }}>{title}</div>
      </div>
    </div>
  );
};

const NewsLowerThird: React.FC<{
  name: string; title: string; brand: BrandSystem;
  progress: number; outProgress: number;
}> = ({ name, title, brand, progress, outProgress }) => {
  const scaleX = interpolate(progress, [0, 1], [0, 1], { extrapolateRight: "clamp", easing: EASE_OUT_EXPO });
  const scaleXOut = interpolate(outProgress, [0, 1], [1, 0], { extrapolateRight: "clamp", easing: EASE_IN_EXPO });
  const textOpacity = interpolate(progress, [0.4, 1], [0, 1], { extrapolateRight: "clamp" });
  const textOpacityOut = interpolate(outProgress, [0, 0.4], [1, 0], { extrapolateRight: "clamp" });
  const effectiveScale = scaleX * scaleXOut;
  return (
    <div style={{ position: "absolute", bottom: 100, left: 0, right: 0, transformOrigin: "left center" }}>
      <div style={{ height: 4, background: brand.accent, width: "100%", transform: `scaleX(${effectiveScale})`, transformOrigin: "left center", position: "absolute", bottom: 0 }} />
      <div style={{ display: "flex", alignItems: "stretch", transform: `scaleX(${effectiveScale})`, transformOrigin: "left center", opacity: textOpacity * textOpacityOut }}>
        <div style={{ background: brand.accent, padding: "14px 24px", fontFamily: brand.fontDisplay, fontWeight: 800, fontSize: 28, color: brand.textLight, letterSpacing: "-0.01em", whiteSpace: "nowrap" }}>{name}</div>
        <div style={{ background: brand.secondary, padding: "14px 28px", fontFamily: brand.fontBody, fontWeight: 400, fontSize: 22, color: brand.textLight, display: "flex", alignItems: "center", whiteSpace: "nowrap" }}>{title}</div>
      </div>
    </div>
  );
};

const ModernLowerThird: React.FC<{
  name: string; title: string; brand: BrandSystem;
  progress: number; outProgress: number;
}> = ({ name, title, brand, progress, outProgress }) => {
  const translateY = interpolate(progress, [0, 1], [60, 0], { extrapolateRight: "clamp", easing: EASE_OUT_EXPO });
  const translateYOut = interpolate(outProgress, [0, 1], [0, 60], { extrapolateRight: "clamp", easing: EASE_IN_EXPO });
  const opacity = interpolate(progress, [0, 0.5], [0, 1], { extrapolateRight: "clamp" });
  const opacityOut = interpolate(outProgress, [0.5, 1], [1, 0], { extrapolateRight: "clamp" });
  return (
    <div style={{ position: "absolute", bottom: 100, left: 80, transform: `translateY(${translateY + translateYOut}px)`, opacity: opacity * opacityOut }}>
      <div style={{ background: "rgba(255,255,255,0.08)", backdropFilter: "blur(20px)", WebkitBackdropFilter: "blur(20px)", border: "1px solid rgba(255,255,255,0.15)", borderRadius: brand.cornerRadius * 2, padding: "16px 28px", borderLeft: `3px solid ${brand.accent}` }}>
        <div style={{ fontFamily: brand.fontDisplay, fontWeight: 700, fontSize: 30, color: brand.textLight, letterSpacing: "-0.02em" }}>{name}</div>
        <div style={{ fontFamily: brand.fontBody, fontWeight: 300, fontSize: 17, color: "rgba(255,255,255,0.7)", marginTop: 6, letterSpacing: "0.06em", textTransform: "uppercase" }}>{title}</div>
      </div>
    </div>
  );
};

export const LowerThird: React.FC<LowerThirdProps> = ({
  name, title, variant = "classic", brand = DEFAULT_BRAND, inFrame = 0, holdFrames = 90, outFrame,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const resolvedOutFrame = outFrame ?? inFrame + holdFrames + 20;
  const outDuration = 20;
  const progress = spring({ frame: frame - inFrame, fps, config: { damping: 20, stiffness: 80, mass: 0.8 } });
  const outProgress = interpolate(frame, [resolvedOutFrame, resolvedOutFrame + outDuration], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
  const commonProps = { name, title, brand, progress, outProgress };
  if (variant === "news") return <NewsLowerThird {...commonProps} />;
  if (variant === "modern") return <ModernLowerThird {...commonProps} />;
  return <ClassicLowerThird {...commonProps} />;
};
```

---

## 3. Kinetic Typography

Four animation modes: **bounce**, **wave**, **typewriter**, **slam**.

```tsx
import React from "react";
import { useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";
import { EASE_OUT_EXPO } from "./brand";

type KineticMode = "bounce" | "wave" | "typewriter" | "slam";

interface KineticTypographyProps {
  text: string;
  mode?: KineticMode;
  startFrame?: number;
  staggerFrames?: number;
  fontSize?: number;
  color?: string;
  fontFamily?: string;
  fontWeight?: number;
}

// Bounce: each character springs up from below
const BounceChar: React.FC<{ char: string; index: number; startFrame: number; staggerFrames: number; fontSize: number; color: string; fontFamily: string; fontWeight: number }> = ({ char, index, startFrame, staggerFrames, fontSize, color, fontFamily, fontWeight }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const progress = spring({ frame: frame - startFrame - index * staggerFrames, fps, config: { damping: 10, stiffness: 200, mass: 0.5 } });
  const translateY = interpolate(progress, [0, 1], [80, 0]);
  const opacity = interpolate(progress, [0, 0.1], [0, 1], { extrapolateRight: "clamp" });
  return <span style={{ display: "inline-block", transform: `translateY(${translateY}px)`, opacity, fontFamily, fontWeight, fontSize, color, lineHeight: 1.1 }}>{char === " " ? " " : char}</span>;
};

// Wave: characters oscillate in a sine wave pattern
const WaveChar: React.FC<{ char: string; index: number; startFrame: number; staggerFrames: number; fontSize: number; color: string; fontFamily: string; fontWeight: number }> = ({ char, index, startFrame, staggerFrames, fontSize, color, fontFamily, fontWeight }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const entryProgress = spring({ frame: frame - startFrame - index * staggerFrames, fps, config: { damping: 18, stiffness: 150 } });
  const waveOffset = Math.sin((frame / fps) * Math.PI * 2 + index * 0.5) * 8;
  const opacity = interpolate(entryProgress, [0, 0.3], [0, 1], { extrapolateRight: "clamp" });
  const translateY = interpolate(entryProgress, [0, 1], [40, 0]) + (entryProgress >= 0.99 ? waveOffset : 0);
  return <span style={{ display: "inline-block", transform: `translateY(${translateY}px)`, opacity, fontFamily, fontWeight, fontSize, color }}>{char === " " ? " " : char}</span>;
};

// Typewriter: characters appear one at a time with cursor blink
const TypewriterText: React.FC<{ text: string; startFrame: number; staggerFrames: number; fontSize: number; color: string; fontFamily: string; fontWeight: number }> = ({ text, startFrame, staggerFrames, fontSize, color, fontFamily, fontWeight }) => {
  const frame = useCurrentFrame();
  const charsVisible = Math.max(0, Math.floor((frame - startFrame) / staggerFrames));
  const showCursor = frame >= startFrame;
  const cursorBlink = Math.floor(frame / 15) % 2 === 0;
  const allVisible = charsVisible >= text.length;
  return (
    <span style={{ fontFamily, fontWeight, fontSize, color }}>
      {text.slice(0, charsVisible)}
      {showCursor && (!allVisible || cursorBlink) && <span style={{ opacity: cursorBlink ? 1 : 0, marginLeft: 2 }}>|</span>}
    </span>
  );
};

// Slam: word slams in large, scales down to normal
const SlamWord: React.FC<{ word: string; index: number; startFrame: number; staggerFrames: number; fontSize: number; color: string; fontFamily: string; fontWeight: number }> = ({ word, index, startFrame, staggerFrames, fontSize, color }) => {
  const frame = useCurrentFrame();
  const elapsed = frame - startFrame - index * staggerFrames;
  const scale = interpolate(elapsed, [0, 4, 12], [3, 1.15, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp", easing: EASE_OUT_EXPO });
  const opacity = interpolate(elapsed, [0, 3], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
  const flashOpacity = interpolate(elapsed, [0, 2, 6], [1, 0.3, 0], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
  return (
    <span style={{ display: "inline-block", transform: `scale(${scale})`, opacity, fontFamily: "Inter", fontWeight: 900, fontSize, color, marginRight: 8, position: "relative" }}>
      {word}
      <span style={{ position: "absolute", inset: 0, color: "#FFFFFF", opacity: flashOpacity, pointerEvents: "none" }}>{word}</span>
    </span>
  );
};

export const KineticTypography: React.FC<KineticTypographyProps> = ({
  text, mode = "bounce", startFrame = 0, staggerFrames = 3,
  fontSize = 72, color = "#FFFFFF", fontFamily = "Inter", fontWeight = 800,
}) => {
  const commonProps = { startFrame, staggerFrames, fontSize, color, fontFamily, fontWeight };
  if (mode === "typewriter") return <TypewriterText text={text} {...commonProps} />;
  if (mode === "slam") {
    const words = text.split(" ");
    return <div style={{ display: "flex", flexWrap: "wrap" }}>{words.map((word, i) => <SlamWord key={i} word={word} index={i} {...commonProps} />)}</div>;
  }
  const chars = text.split("");
  if (mode === "wave") return <div style={{ display: "inline-block" }}>{chars.map((char, i) => <WaveChar key={i} char={char} index={i} {...commonProps} />)}</div>;
  return <div style={{ display: "inline-block", overflow: "hidden" }}>{chars.map((char, i) => <BounceChar key={i} char={char} index={i} {...commonProps} />)}</div>;
};
```

---

## 4. Logo Animation

Four reveal sequences: **draw** (stroke trace), **fade** (opacity bloom), **scale** (punch-in), **morph** (shape shift).

```tsx
import React from "react";
import { useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";
import { EASE_OUT_EXPO, EASE_OUT_BACK } from "./brand";

type LogoRevealMode = "draw" | "fade" | "scale" | "morph";
interface LogoAnimationProps {
  svgPath: string; viewBox?: string; width?: number; height?: number;
  color?: string; mode?: LogoRevealMode; startFrame?: number; strokeWidth?: number;
}

export const LogoAnimation: React.FC<LogoAnimationProps> = ({
  svgPath, viewBox = "0 0 200 200", width = 200, height = 200,
  color = "#FFFFFF", mode = "draw", startFrame = 0, strokeWidth = 3,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const progress = spring({ frame: frame - startFrame, fps, config: { damping: 20, stiffness: 60, mass: 1 } });
  const ESTIMATED_PATH_LENGTH = 1200;

  if (mode === "draw") {
    const dashOffset = interpolate(progress, [0, 0.7], [ESTIMATED_PATH_LENGTH, 0], { extrapolateRight: "clamp", easing: EASE_OUT_EXPO });
    const fillOpacity = interpolate(progress, [0.5, 1], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
    return (
      <svg viewBox={viewBox} width={width} height={height}>
        <path d={svgPath} fill={color} fillOpacity={fillOpacity} />
        <path d={svgPath} fill="none" stroke={color} strokeWidth={strokeWidth} strokeDasharray={ESTIMATED_PATH_LENGTH} strokeDashoffset={dashOffset} strokeLinecap="round" strokeLinejoin="round" />
      </svg>
    );
  }

  if (mode === "fade") {
    const opacity = interpolate(progress, [0, 0.6], [0, 1], { extrapolateRight: "clamp", easing: EASE_OUT_EXPO });
    const glowOpacity = interpolate(progress, [0.1, 0.5], [0.8, 0], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
    const glowScale = interpolate(progress, [0, 0.5], [1.4, 1], { extrapolateRight: "clamp" });
    return (
      <svg viewBox={viewBox} width={width} height={height}>
        <defs><filter id="glow"><feGaussianBlur stdDeviation="8" result="blur" /><feMerge><feMergeNode in="blur" /><feMergeNode in="SourceGraphic" /></feMerge></filter></defs>
        <path d={svgPath} fill={color} opacity={glowOpacity} transform={`scale(${glowScale})`} transformOrigin="center" filter="url(#glow)" />
        <path d={svgPath} fill={color} opacity={opacity} />
      </svg>
    );
  }

  if (mode === "scale") {
    const scale = interpolate(progress, [0, 1], [0, 1], { extrapolateRight: "clamp", easing: EASE_OUT_BACK });
    const opacity = interpolate(progress, [0, 0.2], [0, 1], { extrapolateRight: "clamp" });
    return <svg viewBox={viewBox} width={width} height={height} style={{ transform: `scale(${scale})`, opacity }}><path d={svgPath} fill={color} /></svg>;
  }

  // morph: scale in with color shift
  const scaleX = interpolate(progress, [0, 0.5], [0.1, 1], { extrapolateRight: "clamp", easing: EASE_OUT_EXPO });
  const scaleY = interpolate(progress, [0.3, 1], [0.1, 1], { extrapolateRight: "clamp", easing: EASE_OUT_EXPO });
  const r = Math.round(interpolate(progress, [0, 1], [233, parseInt(color.slice(1, 3), 16)]));
  const g = Math.round(interpolate(progress, [0, 1], [69, parseInt(color.slice(3, 5), 16)]));
  const b = Math.round(interpolate(progress, [0, 1], [96, parseInt(color.slice(5, 7), 16)]));
  return (
    <svg viewBox={viewBox} width={width} height={height}>
      <path d={svgPath} fill={`rgb(${r},${g},${b})`} transform={`scale(${scaleX}, ${scaleY})`} transformOrigin="center" />
    </svg>
  );
};
```

---

## 5. Title Card

Full-screen title sequence with animated background, subtitle, and staggered entry.

```tsx
import React from "react";
import { useCurrentFrame, useVideoConfig, interpolate, spring, AbsoluteFill } from "remotion";
import { EASE_OUT_EXPO, BrandSystem, DEFAULT_BRAND } from "./brand";

interface TitleCardProps {
  title: string; subtitle?: string; eyebrow?: string;
  brand?: BrandSystem; backgroundStyle?: "solid" | "gradient" | "geometric";
  inFrame?: number; outFrame?: number;
}

export const TitleCard: React.FC<TitleCardProps> = ({
  title, subtitle, eyebrow, brand = DEFAULT_BRAND,
  backgroundStyle = "geometric", inFrame = 0, outFrame,
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  const resolvedOutFrame = outFrame ?? durationInFrames - 30;

  const entryProgress = spring({ frame: frame - inFrame, fps, config: { damping: 22, stiffness: 70 } });
  const exitProgress = interpolate(frame, [resolvedOutFrame, resolvedOutFrame + 20], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });

  const eyebrowY = interpolate(entryProgress, [0, 1], [30, 0], { extrapolateRight: "clamp", easing: EASE_OUT_EXPO });
  const eyebrowOpacity = interpolate(entryProgress, [0, 0.4], [0, 1], { extrapolateRight: "clamp" });

  const titleProgress = spring({ frame: Math.max(0, frame - inFrame - 8), fps, config: { damping: 20, stiffness: 60 } });
  const titleY = interpolate(titleProgress, [0, 1], [50, 0], { extrapolateRight: "clamp", easing: EASE_OUT_EXPO });
  const titleOpacity = interpolate(titleProgress, [0, 0.3], [0, 1], { extrapolateRight: "clamp" });

  const subtitleProgress = spring({ frame: Math.max(0, frame - inFrame - 18), fps, config: { damping: 24, stiffness: 80 } });
  const subtitleOpacity = interpolate(subtitleProgress, [0, 0.5], [0, 1], { extrapolateRight: "clamp" });

  const lineScale = interpolate(entryProgress, [0.2, 0.8], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp", easing: EASE_OUT_EXPO });
  const overallOpacity = interpolate(exitProgress, [0, 1], [1, 0], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });

  const bgStyle = backgroundStyle === "gradient"
    ? { background: `linear-gradient(135deg, ${brand.primary} 0%, ${brand.secondary} 50%, ${brand.accent}22 100%)` }
    : { background: brand.primary };

  return (
    <AbsoluteFill style={{ opacity: overallOpacity }}>
      <AbsoluteFill style={bgStyle}>
        {backgroundStyle === "geometric" && (
          <svg width="100%" height="100%" viewBox="0 0 1920 1080" preserveAspectRatio="xMidYMid slice" style={{ position: "absolute", inset: 0 }}>
            {[0, 1, 2, 3].map((i) => {
              const lp = interpolate(entryProgress, [i * 0.05, i * 0.05 + 0.4], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp", easing: EASE_OUT_EXPO });
              return <line key={i} x1={-200 + i * 500} y1={0} x2={i * 500 + 400} y2={1080} stroke={brand.accent} strokeWidth={1.5} opacity={0.2 * lp} strokeDasharray="1920" strokeDashoffset={(1 - lp) * 1920} />;
            })}
            <circle cx={1600} cy={200} r={300} fill="none" stroke={brand.accent} strokeWidth={2} opacity={0.1 * entryProgress} transform={`rotate(${frame * 0.2}, 1600, 200)`} strokeDasharray="20 10" />
            <rect x={0} y={0} width={interpolate(entryProgress, [0, 1], [0, 8], { extrapolateRight: "clamp" })} height={1080} fill={brand.accent} />
          </svg>
        )}
      </AbsoluteFill>

      <AbsoluteFill style={{ display: "flex", flexDirection: "column", justifyContent: "center", alignItems: "flex-start", padding: "0 120px" }}>
        {eyebrow && (
          <div style={{ fontFamily: brand.fontBody, fontWeight: 500, fontSize: 18, color: brand.accent, letterSpacing: "0.2em", textTransform: "uppercase", transform: `translateY(${eyebrowY}px)`, opacity: eyebrowOpacity, marginBottom: 20 }}>
            {eyebrow}
          </div>
        )}
        <div style={{ width: 80, height: 4, background: brand.accent, transform: `scaleX(${lineScale})`, transformOrigin: "left center", marginBottom: 28 }} />
        <div style={{ fontFamily: brand.fontDisplay, fontWeight: 900, fontSize: 96, color: brand.textLight, letterSpacing: "-0.04em", lineHeight: 0.95, transform: `translateY(${titleY}px)`, opacity: titleOpacity, maxWidth: 1200 }}>
          {title}
        </div>
        {subtitle && (
          <div style={{ fontFamily: brand.fontBody, fontWeight: 300, fontSize: 32, color: "rgba(255,255,255,0.65)", marginTop: 32, opacity: subtitleOpacity, maxWidth: 800, lineHeight: 1.4 }}>
            {subtitle}
          </div>
        )}
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```

---

## 6. Score / Stats Bug

Persistent overlay showing live-updating numbers.

```tsx
import React from "react";
import { useCurrentFrame, interpolate, spring, useVideoConfig } from "remotion";
import { EASE_OUT_EXPO, BrandSystem, DEFAULT_BRAND } from "./brand";

interface ScoreEntry { label: string; value: number | string; accentColor?: string; }
interface ScoreBugProps {
  entries: ScoreEntry[];
  position?: "top-left" | "top-right" | "bottom-left" | "bottom-right";
  brand?: BrandSystem; showFrom?: number; liveIndicator?: boolean;
}

const AnimatedNumber: React.FC<{ value: number; prevValue?: number; startFrame: number; color: string; fontSize: number; fontFamily: string }> = ({ value, prevValue = 0, startFrame, color, fontSize, fontFamily }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const rollProgress = spring({ frame: frame - startFrame, fps, config: { damping: 16, stiffness: 120 } });
  const displayValue = Math.round(interpolate(rollProgress, [0, 1], [prevValue, value]));
  const bounceScale = interpolate(frame - startFrame, [0, 4, 10], [1, 1.3, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp", easing: EASE_OUT_EXPO });
  return <span style={{ fontFamily, fontWeight: 900, fontSize, color, display: "inline-block", transform: `scale(${bounceScale})`, minWidth: "2ch", textAlign: "center" }}>{displayValue}</span>;
};

const LiveBadge: React.FC<{ color: string }> = ({ color }) => {
  const frame = useCurrentFrame();
  const pulseOpacity = 0.4 + 0.6 * Math.abs(Math.sin((frame / 30) * Math.PI));
  return (
    <div style={{ display: "flex", alignItems: "center", gap: 6, marginBottom: 8 }}>
      <div style={{ width: 8, height: 8, borderRadius: "50%", background: "#FF0000", opacity: pulseOpacity }} />
      <span style={{ fontWeight: 700, fontSize: 11, color: "#FFFFFF", letterSpacing: "0.15em", textTransform: "uppercase", opacity: 0.9 }}>LIVE</span>
    </div>
  );
};

const POSITION_STYLES: Record<string, React.CSSProperties> = {
  "top-left": { top: 24, left: 24 }, "top-right": { top: 24, right: 24 },
  "bottom-left": { bottom: 24, left: 24 }, "bottom-right": { bottom: 24, right: 24 },
};

export const ScoreBug: React.FC<ScoreBugProps> = ({
  entries, position = "top-right", brand = DEFAULT_BRAND, showFrom = 0, liveIndicator = true,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const entryProgress = spring({ frame: frame - showFrom, fps, config: { damping: 20, stiffness: 100 } });
  const translateX = interpolate(entryProgress, [0, 1], [position.includes("right") ? 120 : -120, 0], { extrapolateRight: "clamp" });
  const opacity = interpolate(entryProgress, [0, 0.3], [0, 1], { extrapolateRight: "clamp" });
  return (
    <div style={{ position: "absolute", ...POSITION_STYLES[position], transform: `translateX(${translateX}px)`, opacity, zIndex: 100 }}>
      <div style={{ background: "rgba(0,0,0,0.85)", backdropFilter: "blur(12px)", borderRadius: brand.cornerRadius * 2, padding: "14px 20px", border: `1px solid rgba(255,255,255,0.1)`, borderTop: `3px solid ${brand.accent}`, minWidth: 160 }}>
        {liveIndicator && <LiveBadge color={brand.accent} />}
        {entries.map((entry, i) => (
          <div key={i} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", gap: 24, padding: "6px 0", borderBottom: i < entries.length - 1 ? "1px solid rgba(255,255,255,0.08)" : "none" }}>
            <span style={{ fontFamily: brand.fontBody, fontWeight: 500, fontSize: 14, color: "rgba(255,255,255,0.6)", textTransform: "uppercase", letterSpacing: "0.08em" }}>{entry.label}</span>
            {typeof entry.value === "number"
              ? <AnimatedNumber value={entry.value} startFrame={showFrom} color={entry.accentColor ?? brand.accent} fontSize={22} fontFamily={brand.fontDisplay} />
              : <span style={{ fontFamily: brand.fontDisplay, fontWeight: 900, fontSize: 22, color: entry.accentColor ?? brand.accent }}>{entry.value}</span>
            }
          </div>
        ))}
      </div>
    </div>
  );
};
```

---

## 7. Transition Wipes

```tsx
import React from "react";
import { AbsoluteFill, useCurrentFrame, interpolate } from "remotion";
import { EASE_IN_OUT_QUART, BrandSystem, DEFAULT_BRAND } from "./brand";

type WipeDirection = "horizontal" | "diagonal";
interface TransitionWipeProps {
  direction?: WipeDirection; brand?: BrandSystem;
  durationFrames?: number; startFrame?: number; color?: string;
}

export const TransitionWipe: React.FC<TransitionWipeProps> = ({
  direction = "horizontal", brand = DEFAULT_BRAND,
  durationFrames = 30, startFrame = 0, color,
}) => {
  const frame = useCurrentFrame();
  const resolvedColor = color ?? brand.accent;
  const progress = interpolate(frame, [startFrame, startFrame + durationFrames], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
  const inP = interpolate(progress, [0, 0.45], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp", easing: EASE_IN_OUT_QUART });
  const outP = interpolate(progress, [0.55, 1], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp", easing: EASE_IN_OUT_QUART });

  if (direction === "diagonal") {
    const skew = 15;
    const inX = interpolate(inP, [0, 1], [-skew, 100 + skew]);
    const outX = interpolate(outP, [0, 1], [0, 100 + skew * 2]);
    return (
      <AbsoluteFill>
        <div style={{ position: "absolute", inset: 0, background: resolvedColor, clipPath: `polygon(${outX - skew}% 0, ${inX}% 0, ${inX - skew}% 100%, ${outX - skew * 2}% 100%)` }} />
      </AbsoluteFill>
    );
  }

  const clipRight = interpolate(inP, [0, 1], [0, 100]);
  const clipLeft = interpolate(outP, [0, 1], [0, 100]);
  return (
    <AbsoluteFill>
      <div style={{ position: "absolute", inset: 0, background: resolvedColor, clipPath: `polygon(${clipLeft}% 0, ${clipRight}% 0, ${clipRight}% 100%, ${clipLeft}% 100%)` }} />
    </AbsoluteFill>
  );
};
```

---

## 8. Animated Charts

```tsx
import React from "react";
import { useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";
import { EASE_OUT_EXPO, BrandSystem, DEFAULT_BRAND } from "./brand";

interface BarDatum { label: string; value: number; color?: string; }

export const AnimatedBarChart: React.FC<{ data: BarDatum[]; width?: number; height?: number; brand?: BrandSystem; startFrame?: number; staggerFrames?: number; showValues?: boolean }> = ({
  data, width = 800, height = 400, brand = DEFAULT_BRAND, startFrame = 0, staggerFrames = 6, showValues = true,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const maxValue = Math.max(...data.map((d) => d.value));
  const barWidth = (width - 40) / data.length - 12;
  const chartHeight = height - 60;
  return (
    <svg width={width} height={height} viewBox={`0 0 ${width} ${height}`}>
      <line x1={20} y1={chartHeight + 10} x2={width - 20} y2={chartHeight + 10} stroke="rgba(255,255,255,0.2)" strokeWidth={1} />
      {data.map((datum, i) => {
        const barProgress = spring({ frame: frame - startFrame - i * staggerFrames, fps, config: { damping: 18, stiffness: 100 } });
        const barHeight = (datum.value / maxValue) * chartHeight * barProgress;
        const x = 20 + i * (barWidth + 12);
        const y = chartHeight + 10 - barHeight;
        const barColor = datum.color ?? brand.accent;
        const valueOpacity = interpolate(barProgress, [0.7, 1], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
        return (
          <g key={i}>
            <rect x={x} y={y} width={barWidth} height={barHeight} fill={barColor} rx={brand.cornerRadius} />
            {showValues && <text x={x + barWidth / 2} y={y - 8} textAnchor="middle" fill="white" fontSize={14} fontWeight={700} opacity={valueOpacity}>{datum.value}</text>}
            <text x={x + barWidth / 2} y={chartHeight + 28} textAnchor="middle" fill="rgba(255,255,255,0.6)" fontSize={12}>{datum.label}</text>
          </g>
        );
      })}
    </svg>
  );
};

function polarToCartesian(cx: number, cy: number, r: number, angleDeg: number) {
  const rad = ((angleDeg - 90) * Math.PI) / 180;
  return { x: cx + r * Math.cos(rad), y: cy + r * Math.sin(rad) };
}
function describeArc(cx: number, cy: number, r: number, startAngle: number, endAngle: number) {
  const start = polarToCartesian(cx, cy, r, endAngle);
  const end = polarToCartesian(cx, cy, r, startAngle);
  const largeArc = endAngle - startAngle <= 180 ? 0 : 1;
  return `M ${cx} ${cy} L ${start.x} ${start.y} A ${r} ${r} 0 ${largeArc} 0 ${end.x} ${end.y} Z`;
}

export const AnimatedPieChart: React.FC<{ data: { label: string; value: number; color: string }[]; radius?: number; brand?: BrandSystem; startFrame?: number }> = ({
  data, radius = 180, brand = DEFAULT_BRAND, startFrame = 0,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const drawProgress = spring({ frame: frame - startFrame, fps, config: { damping: 22, stiffness: 60 } });
  const total = data.reduce((sum, d) => sum + d.value, 0);
  let currentAngle = 0;
  const cx = radius + 20, cy = radius + 20, size = (radius + 20) * 2;
  return (
    <svg width={size} height={size}>
      {data.map((datum, i) => {
        const sliceAngle = (datum.value / total) * 360 * drawProgress;
        const startAngle = currentAngle;
        const endAngle = currentAngle + sliceAngle;
        currentAngle += (datum.value / total) * 360;
        if (sliceAngle <= 0) return null;
        const midAngle = startAngle + sliceAngle / 2;
        const labelPos = polarToCartesian(cx, cy, radius * 0.65, midAngle);
        const labelOpacity = interpolate(drawProgress, [0.6, 1], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
        return (
          <g key={i}>
            <path d={describeArc(cx, cy, radius, startAngle, endAngle)} fill={datum.color} stroke="rgba(0,0,0,0.3)" strokeWidth={2} />
            <text x={labelPos.x} y={labelPos.y} textAnchor="middle" dominantBaseline="middle" fill="white" fontSize={13} fontWeight={700} opacity={labelOpacity}>{Math.round((datum.value / total) * 100)}%</text>
          </g>
        );
      })}
    </svg>
  );
};

export const AnimatedLineChart: React.FC<{ data: number[]; width?: number; height?: number; brand?: BrandSystem; startFrame?: number; color?: string }> = ({
  data, width = 800, height = 300, brand = DEFAULT_BRAND, startFrame = 0, color,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const traceProgress = spring({ frame: frame - startFrame, fps, config: { damping: 26, stiffness: 50 } });
  const lineColor = color ?? brand.accent;
  const padX = 40, padY = 30, plotW = width - padX * 2, plotH = height - padY * 2;
  const maxVal = Math.max(...data), minVal = Math.min(...data), range = maxVal - minVal || 1;
  const toX = (i: number) => padX + (i / (data.length - 1)) * plotW;
  const toY = (v: number) => padY + plotH - ((v - minVal) / range) * plotH;
  const points = data.map((v, i) => `${toX(i)},${toY(v)}`).join(" ");
  const clipWidth = interpolate(traceProgress, [0, 1], [0, width], { extrapolateRight: "clamp" });
  return (
    <svg width={width} height={height}>
      <defs>
        <clipPath id="line-trace"><rect x={0} y={0} width={clipWidth} height={height} /></clipPath>
        <linearGradient id="line-fill" x1="0" x2="0" y1="0" y2="1">
          <stop offset="0%" stopColor={lineColor} stopOpacity={0.3} />
          <stop offset="100%" stopColor={lineColor} stopOpacity={0} />
        </linearGradient>
      </defs>
      {[0, 0.25, 0.5, 0.75, 1].map((t) => <line key={t} x1={padX} y1={padY + t * plotH} x2={width - padX} y2={padY + t * plotH} stroke="rgba(255,255,255,0.08)" strokeWidth={1} />)}
      <polygon points={`${toX(0)},${padY + plotH} ${points} ${toX(data.length - 1)},${padY + plotH}`} fill="url(#line-fill)" clipPath="url(#line-trace)" />
      <polyline points={points} fill="none" stroke={lineColor} strokeWidth={3} strokeLinecap="round" strokeLinejoin="round" clipPath="url(#line-trace)" />
      {data.map((v, i) => toX(i) <= clipWidth ? <circle key={i} cx={toX(i)} cy={toY(v)} r={5} fill={lineColor} stroke="white" strokeWidth={2} /> : null)}
    </svg>
  );
};
```

---

## 9. Particle / Confetti System

```tsx
import React, { useMemo } from "react";
import { useCurrentFrame, useVideoConfig, AbsoluteFill, interpolate } from "remotion";

interface Particle { id: number; x: number; startY: number; size: number; color: string; rotationSpeed: number; fallSpeed: number; shape: "rect" | "circle" | "triangle"; delay: number; swayAmount: number; swaySpeed: number; }

function seededRandom(seed: number): () => number {
  let s = seed;
  return () => { s = (s * 16807 + 0) % 2147483647; return (s - 1) / 2147483646; };
}

function generateParticles(count: number, colors: string[]): Particle[] {
  const rand = seededRandom(42);
  const shapes: Particle["shape"][] = ["rect", "circle", "triangle"];
  return Array.from({ length: count }, (_, i) => ({
    id: i, x: rand() * 100, startY: -10 - rand() * 20, size: 8 + rand() * 14,
    color: colors[Math.floor(rand() * colors.length)], rotationSpeed: (rand() - 0.5) * 6,
    fallSpeed: 1.5 + rand() * 3, shape: shapes[Math.floor(rand() * shapes.length)],
    delay: Math.floor(rand() * 40), swayAmount: 20 + rand() * 40, swaySpeed: 0.5 + rand() * 1.5,
  }));
}

export const ParticleSystem: React.FC<{ count?: number; colors?: string[]; startFrame?: number; gravity?: number }> = ({
  count = 80, colors = ["#FF6B6B", "#FFE66D", "#4ECDC4", "#45B7D1", "#96CEB4", "#FFEAA7"],
  startFrame = 0, gravity = 0.3,
}) => {
  const frame = useCurrentFrame();
  const { height } = useVideoConfig();
  const particles = useMemo(() => generateParticles(count, colors), [count, colors]);
  const elapsed = frame - startFrame;
  return (
    <AbsoluteFill style={{ pointerEvents: "none" }}>
      {particles.map((p) => {
        const pElapsed = elapsed - p.delay;
        if (pElapsed <= 0) return null;
        const fallPx = p.fallSpeed * pElapsed + 0.5 * gravity * pElapsed * pElapsed;
        const yPercent = p.startY + (fallPx / height) * 100;
        if (yPercent > 115) return null;
        const opacity = interpolate(yPercent, [80, 110], [1, 0], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
        const swayX = Math.sin(frame * p.swaySpeed * 0.05 + p.id) * p.swayAmount;
        const baseStyle: React.CSSProperties = { position: "absolute", left: `${p.x}%`, top: `${yPercent}%`, transform: `translate(${swayX}px, 0) rotate(${pElapsed * p.rotationSpeed}deg)`, opacity };
        if (p.shape === "circle") return <div key={p.id} style={{ ...baseStyle, width: p.size, height: p.size, borderRadius: "50%", background: p.color }} />;
        if (p.shape === "triangle") return <div key={p.id} style={{ ...baseStyle, width: 0, height: 0, borderLeft: `${p.size / 2}px solid transparent`, borderRight: `${p.size / 2}px solid transparent`, borderBottom: `${p.size}px solid ${p.color}` }} />;
        return <div key={p.id} style={{ ...baseStyle, width: p.size, height: p.size * 0.5, background: p.color, borderRadius: 2 }} />;
      })}
    </AbsoluteFill>
  );
};
```

---

## 10. Reusable Animation Hooks

```tsx
import { useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";
import { EASE_OUT_EXPO, EASE_IN_EXPO } from "./brand";

export function useSlideIn({ startFrame = 0, direction = "left" as "left"|"right"|"up"|"down", distance = 80, damping = 20, stiffness = 80, mass = 0.8 } = {}) {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const progress = spring({ frame: frame - startFrame, fps, config: { damping, stiffness, mass } });
  const offset = interpolate(progress, [0, 1], [distance, 0]);
  const opacity = interpolate(progress, [0, 0.3], [0, 1], { extrapolateRight: "clamp" });
  const translateX = direction === "left" ? -offset : direction === "right" ? offset : 0;
  const translateY = direction === "up" ? -offset : direction === "down" ? offset : 0;
  return { translateX, translateY, opacity, style: { transform: `translate(${translateX}px, ${translateY}px)`, opacity } as React.CSSProperties };
}

export function useFadeSequence(entries: { startFrame: number; endFrame?: number; fadeDuration?: number }[]): number[] {
  const frame = useCurrentFrame();
  return entries.map(({ startFrame, endFrame, fadeDuration = 15 }) => {
    const fadeIn = interpolate(frame, [startFrame, startFrame + fadeDuration], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp", easing: EASE_OUT_EXPO });
    if (endFrame !== undefined) {
      const fadeOut = interpolate(frame, [endFrame - fadeDuration, endFrame], [1, 0], { extrapolateLeft: "clamp", extrapolateRight: "clamp", easing: EASE_IN_EXPO });
      return Math.min(fadeIn, fadeOut);
    }
    return fadeIn;
  });
}

export function useStagger({ count, startFrame = 0, staggerFrames = 5, config = { damping: 20, stiffness: 80 } }: { count: number; startFrame?: number; staggerFrames?: number; config?: { damping?: number; stiffness?: number } }): number[] {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  return Array.from({ length: count }, (_, i) => spring({ frame: frame - startFrame - i * staggerFrames, fps, config }));
}

export function usePulse({ periodFrames = 60, minScale = 0.95, maxScale = 1.05, startFrame = 0 } = {}): number {
  const frame = useCurrentFrame();
  const t = (Math.max(0, frame - startFrame) % periodFrames) / periodFrames;
  return minScale + (maxScale - minScale) * (0.5 + 0.5 * Math.sin(t * Math.PI * 2));
}

export function useCountUp({ from = 0, to, startFrame = 0 }: { from?: number; to: number; startFrame?: number }): number {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const progress = spring({ frame: frame - startFrame, fps, config: { damping: 26, stiffness: 40 } });
  return Math.round(interpolate(progress, [0, 1], [from, to]));
}

export function useTypewriter(text: string, startFrame: number, framesPerChar = 2): string {
  const frame = useCurrentFrame();
  return text.slice(0, Math.min(text.length, Math.floor(Math.max(0, frame - startFrame) / framesPerChar)));
}
```

---

## 11. Timing Reference

### Lower Thirds

| Phase | Frames (30fps) | Frames (60fps) | Notes |
|---|---|---|---|
| Slide in | 15-20 | 30-40 | Spring-based, not linear |
| Hold minimum | 90 | 180 | 3 seconds |
| Hold standard | 120-150 | 240-300 | 4-5 seconds |
| Hold feature guest | 180-210 | 360-420 | 6-7 seconds |
| Slide out | 15-20 | 30-40 | Mirror of in |

### Title Cards

| Phase | Frames (30fps) | Frames (60fps) | Notes |
|---|---|---|---|
| Entry animation | 20-30 | 40-60 | Staggered elements |
| Hold section title | 60-90 | 120-180 | 2-3 seconds |
| Hold opening title | 90-150 | 180-300 | 3-5 seconds |
| Exit | 15-20 | 30-40 | Fade or wipe |

### Transitions

| Type | Frames (30fps) | Notes |
|---|---|---|
| Hard cut | 0 | No animation |
| Snap wipe | 8-12 | Fast branded wipe |
| Standard wipe | 20-30 | Mid-show transitions |
| Logo reveal | 45-60 | Act breaks, openings |
| Crossfade | 15-20 | Soft cut between scenes |

### Kinetic Typography

| Mode | Per-character stagger | Notes |
|---|---|---|
| Bounce | 2-4 frames | Energetic, casual |
| Wave | 3-5 frames | Music/entertainment |
| Typewriter | 1-3 frames | News, documentary |
| Slam | 8-12 frames per word | High impact, sports |

### Chart Builds

| Type | Entry duration | Notes |
|---|---|---|
| Bar chart (per bar) | 8-15 frames stagger | Spring-based for bounce |
| Pie chart (full) | 60-90 frames | Single sweep |
| Line chart trace | 60-120 frames | Clip-path sweep |

### Score Bug

| Phase | Frames | Notes |
|---|---|---|
| Entry slide | 15-20 | Slides from edge |
| Hold (persistent) | Indefinite | On-screen for full segment |
| Number update flash | 4-8 | Brief scale pulse on change |
| Exit | 12-15 | Faster than entry |

### Rule of Thumb

```
Entry <= Exit duration
Hold >= 3x Entry duration
Never overlap more than 2 animating elements simultaneously
Motion should always resolve before narration begins
```

### Sample Composition Timeline (30fps)

```
Frame   0: Scene begins
Frame   0: Score bug slides in
Frame  20: Lower third slides in
Frame 140: Lower third slides out
Frame 150: Kinetic headline slams in
Frame 210: Kinetic headline fades
Frame 240: Transition wipe begins (30 frames)
Frame 270: New scene begins
```
