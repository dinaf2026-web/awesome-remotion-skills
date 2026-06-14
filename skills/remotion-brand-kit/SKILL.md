# SKILL.md — remotion-brand-kit

```markdown
---
name: remotion-brand-kit
description: Build a consistent brand video system in Remotion — reusable intro/outro, color tokens, typography system, logo component, watermark, and a brand config that applies across all your videos.
origin: community
tags: [remotion, branding, brand-kit, design-system, intro, outro, consistency]
---

# Remotion Brand Kit

Build a production-grade brand video system in Remotion. The `BrandConfig` object is the single source of truth — every component reads from it, so changing your brand means changing one file.

---

## 1. Brand Config System

`src/brand/brand.config.ts` — import this everywhere.

```typescript
import { BrandConfig } from './brand.types';

export const BRAND: BrandConfig = {
  identity: {
    name: 'Acme Studio',
    tagline: 'Build things that matter.',
    logoUrl: '/logo.png',          // put in public/
    logoAspectRatio: 4 / 1,        // width / height
    faviconUrl: '/favicon.png',
  },

  colors: {
    primary: '#0F172A',            // near-black navy
    secondary: '#6366F1',          // indigo accent
    accent: '#F59E0B',             // amber highlight
    background: '#FFFFFF',
    backgroundDark: '#0F172A',
    surface: '#F8FAFC',
    text: '#0F172A',
    textMuted: '#64748B',
    textOnDark: '#F8FAFC',
    textOnAccent: '#FFFFFF',
    gradient: {
      from: '#6366F1',
      to: '#8B5CF6',
      angle: 135,
    },
  },

  typography: {
    headingFamily: 'Sora',
    bodyFamily: 'Inter',
    monoFamily: 'Fira Code',
    weights: {
      light: 300,
      regular: 400,
      medium: 500,
      semibold: 600,
      bold: 700,
      extrabold: 800,
    },
    scale: {
      hero: 96,
      h1: 64,
      h2: 48,
      h3: 32,
      h4: 24,
      body: 18,
      caption: 14,
      label: 12,
    },
  },

  timing: {
    introDuration: 90,             // frames at 30fps = 3s
    outroDuration: 150,            // frames = 5s
    logoRevealFrames: 30,
    textFadeFrames: 20,
    transitionFrames: 15,
    musicStingFrame: 0,            // frame where audio sting peaks
  },

  social: {
    handles: {
      youtube: '@acmestudio',
      instagram: '@acmestudio',
      twitter: '@acmestudio',
      tiktok: '@acmestudio',
    },
    cta: 'Subscribe for more ->',
    website: 'acmestudio.io',
  },

  watermark: {
    position: 'bottom-right',
    opacity: 0.6,
    size: 48,                      // px height
    padding: 24,
  },

  animation: {
    easing: 'ease-out',
    springConfig: {
      damping: 14,
      stiffness: 120,
      mass: 1,
    },
  },

  audio: {
    musicStingUrl: '/audio/sting.mp3',
    outroMusicUrl: '/audio/outro.mp3',
    defaultVolume: 0.8,
  },
};
```

---

## 2. Brand Token Types

`src/brand/brand.types.ts`

```typescript
export type WatermarkPosition =
  | 'top-left'
  | 'top-right'
  | 'bottom-left'
  | 'bottom-right'
  | 'top-center'
  | 'bottom-center';

export type LogoAnimationVariant = 'fade' | 'slide-up' | 'scale-pop';

export type BackgroundVariant =
  | 'solid'
  | 'gradient'
  | 'video-loop'
  | 'animated-pattern';

export interface ColorTokens {
  primary: string;
  secondary: string;
  accent: string;
  background: string;
  backgroundDark: string;
  surface: string;
  text: string;
  textMuted: string;
  textOnDark: string;
  textOnAccent: string;
  gradient: {
    from: string;
    to: string;
    angle: number;
  };
}

export interface TypographyTokens {
  headingFamily: string;
  bodyFamily: string;
  monoFamily: string;
  weights: {
    light: number;
    regular: number;
    medium: number;
    semibold: number;
    bold: number;
    extrabold: number;
  };
  scale: {
    hero: number;
    h1: number;
    h2: number;
    h3: number;
    h4: number;
    body: number;
    caption: number;
    label: number;
  };
}

export interface TimingTokens {
  introDuration: number;
  outroDuration: number;
  logoRevealFrames: number;
  textFadeFrames: number;
  transitionFrames: number;
  musicStingFrame: number;
}

export interface SocialTokens {
  handles: {
    youtube?: string;
    instagram?: string;
    twitter?: string;
    tiktok?: string;
    linkedin?: string;
  };
  cta: string;
  website: string;
}

export interface WatermarkTokens {
  position: WatermarkPosition;
  opacity: number;
  size: number;
  padding: number;
}

export interface AnimationTokens {
  easing: string;
  springConfig: {
    damping: number;
    stiffness: number;
    mass: number;
  };
}

export interface AudioTokens {
  musicStingUrl?: string;
  outroMusicUrl?: string;
  defaultVolume: number;
}

export interface IdentityTokens {
  name: string;
  tagline: string;
  logoUrl: string;
  logoAspectRatio: number;
  faviconUrl?: string;
}

export interface BrandConfig {
  identity: IdentityTokens;
  colors: ColorTokens;
  typography: TypographyTokens;
  timing: TimingTokens;
  social: SocialTokens;
  watermark: WatermarkTokens;
  animation: AnimationTokens;
  audio: AudioTokens;
}

export type BrandOverride = DeepPartial<BrandConfig>;

type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

export function mergeBrand(
  base: BrandConfig,
  override: BrandOverride
): BrandConfig {
  return {
    ...base,
    ...override,
    colors: { ...base.colors, ...(override.colors ?? {}) },
    typography: { ...base.typography, ...(override.typography ?? {}) },
    timing: { ...base.timing, ...(override.timing ?? {}) },
    social: { ...base.social, ...(override.social ?? {}) },
    watermark: { ...base.watermark, ...(override.watermark ?? {}) },
    animation: { ...base.animation, ...(override.animation ?? {}) },
    audio: { ...base.audio, ...(override.audio ?? {}) },
    identity: { ...base.identity, ...(override.identity ?? {}) },
  };
}
```

---

## 3. Animated Logo Component

`src/brand/AnimatedLogo.tsx`

Three animation variants: `fade`, `slide-up`, `scale-pop`.

```typescript
import {
  useCurrentFrame,
  useVideoConfig,
  spring,
  interpolate,
  Img,
  Easing,
} from 'remotion';
import { BrandConfig, LogoAnimationVariant } from './brand.types';

interface AnimatedLogoProps {
  brand: BrandConfig;
  variant?: LogoAnimationVariant;
  height?: number;
  startFrame?: number;
  style?: React.CSSProperties;
}

export const AnimatedLogo: React.FC<AnimatedLogoProps> = ({
  brand,
  variant = 'slide-up',
  height = 64,
  startFrame = 0,
  style,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const relativeFrame = frame - startFrame;
  const { springConfig } = brand.animation;

  const fadeOpacity = interpolate(
    relativeFrame,
    [0, brand.timing.logoRevealFrames],
    [0, 1],
    {
      easing: Easing.out(Easing.cubic),
      extrapolateRight: 'clamp',
      extrapolateLeft: 'clamp',
    }
  );

  const slideProgress = spring({
    frame: relativeFrame,
    fps,
    config: springConfig,
    durationInFrames: brand.timing.logoRevealFrames,
  });
  const slideY = interpolate(slideProgress, [0, 1], [40, 0]);
  const slideOpacity = interpolate(slideProgress, [0, 0.4], [0, 1], {
    extrapolateRight: 'clamp',
  });

  const popProgress = spring({
    frame: relativeFrame,
    fps,
    config: { damping: 10, stiffness: 200, mass: 0.8 },
    durationInFrames: brand.timing.logoRevealFrames,
  });
  const popScale = interpolate(popProgress, [0, 1], [0.4, 1]);
  const popOpacity = interpolate(popProgress, [0, 0.3], [0, 1], {
    extrapolateRight: 'clamp',
  });

  const variantStyles: Record<LogoAnimationVariant, React.CSSProperties> = {
    fade: { opacity: fadeOpacity },
    'slide-up': { opacity: slideOpacity, transform: `translateY(${slideY}px)` },
    'scale-pop': { opacity: popOpacity, transform: `scale(${popScale})` },
  };

  const width = height * brand.identity.logoAspectRatio;

  return (
    <div style={{ display: 'flex', alignItems: 'center', justifyContent: 'center', ...variantStyles[variant], ...style }}>
      <Img src={brand.identity.logoUrl} style={{ height, width, objectFit: 'contain' }} />
    </div>
  );
};
```

---

## 4. Intro Sequence

`src/brand/IntroSequence.tsx`

3-second branded opener. Logo reveal, then title slides up underneath.

```typescript
import { AbsoluteFill, Audio, interpolate, spring, useCurrentFrame, useVideoConfig, Easing } from 'remotion';
import { BrandConfig } from './brand.types';
import { AnimatedLogo } from './AnimatedLogo';
import { BrandText } from './BrandText';

interface IntroSequenceProps {
  brand: BrandConfig;
  title?: string;
  subtitle?: string;
  backgroundVariant?: 'dark' | 'light' | 'gradient';
}

export const IntroSequence: React.FC<IntroSequenceProps> = ({
  brand, title, subtitle, backgroundVariant = 'dark',
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const { timing, colors } = brand;

  const titleStartFrame = Math.floor(timing.logoRevealFrames * 0.6);

  const titleProgress = spring({ frame: frame - titleStartFrame, fps, config: brand.animation.springConfig, durationInFrames: timing.textFadeFrames });
  const titleOpacity = interpolate(titleProgress, [0, 1], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });
  const titleY = interpolate(titleProgress, [0, 1], [20, 0]);

  const subtitleStartFrame = titleStartFrame + timing.textFadeFrames - 5;
  const subtitleProgress = spring({ frame: frame - subtitleStartFrame, fps, config: brand.animation.springConfig, durationInFrames: timing.textFadeFrames });
  const subtitleOpacity = interpolate(subtitleProgress, [0, 1], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });

  const fadeOutStart = timing.introDuration - 10;
  const globalOpacity = interpolate(frame, [fadeOutStart, timing.introDuration], [1, 0], { easing: Easing.in(Easing.cubic), extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });

  const backgrounds: Record<string, React.CSSProperties> = {
    dark: { background: colors.backgroundDark },
    light: { background: colors.background },
    gradient: { background: `linear-gradient(${colors.gradient.angle}deg, ${colors.gradient.from}, ${colors.gradient.to})` },
  };

  const textColor = backgroundVariant === 'light' ? colors.text : colors.textOnDark;

  return (
    <AbsoluteFill style={{ ...backgrounds[backgroundVariant], display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center', gap: 24, opacity: globalOpacity }}>
      {brand.audio.musicStingUrl && <Audio src={brand.audio.musicStingUrl} startFrom={0} volume={brand.audio.defaultVolume} />}
      <AnimatedLogo brand={brand} variant="slide-up" height={80} startFrame={0} />
      {title && (
        <div style={{ opacity: titleOpacity, transform: `translateY(${titleY}px)` }}>
          <BrandText brand={brand} variant="h2" color={textColor} align="center" weight="bold">{title}</BrandText>
        </div>
      )}
      {subtitle && (
        <div style={{ opacity: subtitleOpacity }}>
          <BrandText brand={brand} variant="body" color={colors.textMuted} align="center" weight="regular">{subtitle}</BrandText>
        </div>
      )}
      <div style={{ position: 'absolute', bottom: 0, left: 0, height: 4, width: `${interpolate(titleProgress, [0, 1], [0, 100])}%`, background: colors.accent, opacity: titleOpacity }} />
    </AbsoluteFill>
  );
};
```

---

## 5. Outro Sequence

`src/brand/OutroSequence.tsx`

5-second branded end card with CTA, social handles, and subscribe prompt.

```typescript
import { AbsoluteFill, Audio, interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';
import { BrandConfig } from './brand.types';
import { AnimatedLogo } from './AnimatedLogo';
import { BrandText } from './BrandText';

interface OutroSequenceProps {
  brand: BrandConfig;
  showSocialHandles?: boolean;
  showSubscribe?: boolean;
  customCta?: string;
}

export const OutroSequence: React.FC<OutroSequenceProps> = ({
  brand, showSocialHandles = true, showSubscribe = true, customCta,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const { timing, colors, social } = brand;

  const makeSpring = (startFrame: number) =>
    spring({ frame: frame - startFrame, fps, config: brand.animation.springConfig, durationInFrames: timing.textFadeFrames });

  const logoProgress = makeSpring(0);
  const ctaProgress = makeSpring(20);
  const socialProgress = makeSpring(35);
  const subscribeProgress = makeSpring(55);

  const animated = (progress: number, yOffset = 24) => ({
    opacity: interpolate(progress, [0, 1], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' }),
    translateY: interpolate(progress, [0, 1], [yOffset, 0]),
  });

  const logo = animated(logoProgress);
  const cta = animated(ctaProgress);
  const socialAnim = animated(socialProgress);
  const subscribe = animated(subscribeProgress);

  const handles = Object.entries(social.handles).filter(([, val]) => val !== undefined);

  return (
    <AbsoluteFill style={{ background: colors.backgroundDark, display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center', gap: 32 }}>
      {brand.audio.outroMusicUrl && <Audio src={brand.audio.outroMusicUrl} startFrom={0} volume={brand.audio.defaultVolume * 0.6} />}
      <div style={{ opacity: logo.opacity, transform: `translateY(${logo.translateY}px)` }}>
        <AnimatedLogo brand={brand} variant="fade" height={56} startFrame={0} />
      </div>
      <div style={{ width: 48, height: 3, background: colors.accent, borderRadius: 2, opacity: logo.opacity }} />
      <div style={{ opacity: cta.opacity, transform: `translateY(${cta.translateY}px)`, textAlign: 'center' }}>
        <BrandText brand={brand} variant="h3" color={colors.textOnDark} weight="bold" align="center">{customCta ?? social.cta}</BrandText>
        <BrandText brand={brand} variant="body" color={colors.textMuted} align="center" style={{ marginTop: 8 }}>{social.website}</BrandText>
      </div>
      {showSocialHandles && handles.length > 0 && (
        <div style={{ display: 'flex', gap: 24, flexWrap: 'wrap', justifyContent: 'center' }}>
          {handles.map(([platform, handle]) => (
            <div key={platform} style={{ display: 'flex', gap: 8, alignItems: 'center', opacity: socialAnim.opacity, transform: `translateY(${socialAnim.translateY}px)` }}>
              <span style={{ color: colors.secondary, fontWeight: 600, fontSize: 14 }}>{platform.toUpperCase()}</span>
              <span style={{ color: '#94A3B8', fontSize: 14 }}>{handle}</span>
            </div>
          ))}
        </div>
      )}
      {showSubscribe && (
        <div style={{ opacity: subscribe.opacity, transform: `translateY(${subscribe.translateY}px)`, background: colors.secondary, paddingInline: 32, paddingBlock: 14, borderRadius: 999 }}>
          <BrandText brand={brand} variant="body" color={colors.textOnAccent} weight="semibold" align="center">Subscribe — it's free</BrandText>
        </div>
      )}
    </AbsoluteFill>
  );
};
```

---

## 6. Watermark Component

`src/brand/Watermark.tsx`

```typescript
import { AbsoluteFill, Img, interpolate, useCurrentFrame } from 'remotion';
import { BrandConfig, WatermarkPosition } from './brand.types';

interface WatermarkProps {
  brand: BrandConfig;
  position?: WatermarkPosition;
  opacity?: number;
  size?: number;
  padding?: number;
  fadeInFrames?: number;
}

function positionToStyles(position: WatermarkPosition, padding: number): React.CSSProperties {
  const p = padding;
  const map: Record<WatermarkPosition, React.CSSProperties> = {
    'top-left':      { top: p, left: p },
    'top-right':     { top: p, right: p },
    'bottom-left':   { bottom: p, left: p },
    'bottom-right':  { bottom: p, right: p },
    'top-center':    { top: p, left: '50%', transform: 'translateX(-50%)' },
    'bottom-center': { bottom: p, left: '50%', transform: 'translateX(-50%)' },
  };
  return map[position];
}

export const Watermark: React.FC<WatermarkProps> = ({ brand, position, opacity, size, padding, fadeInFrames = 10 }) => {
  const frame = useCurrentFrame();
  const wm = brand.watermark;
  const resolvedPosition = position ?? wm.position;
  const resolvedOpacity = opacity ?? wm.opacity;
  const resolvedSize = size ?? wm.size;
  const resolvedPadding = padding ?? wm.padding;

  const fadeOpacity = interpolate(frame, [0, fadeInFrames], [0, resolvedOpacity], { extrapolateRight: 'clamp', extrapolateLeft: 'clamp' });

  return (
    <AbsoluteFill style={{ pointerEvents: 'none' }}>
      <div style={{ position: 'absolute', opacity: fadeOpacity, ...positionToStyles(resolvedPosition, resolvedPadding) }}>
        <Img src={brand.identity.logoUrl} style={{ height: resolvedSize, width: resolvedSize * brand.identity.logoAspectRatio, objectFit: 'contain' }} />
      </div>
    </AbsoluteFill>
  );
};
```

---

## 7. Lower Third Brand Component

`src/brand/LowerThird.tsx`

```typescript
import { AbsoluteFill, interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';
import { BrandConfig } from './brand.types';
import { BrandText } from './BrandText';

interface LowerThirdProps {
  brand: BrandConfig;
  name: string;
  title?: string;
  startFrame?: number;
  durationInFrames?: number;
  position?: 'bottom-left' | 'bottom-right' | 'top-left' | 'top-right';
  theme?: 'dark' | 'accent' | 'glass';
}

export const LowerThird: React.FC<LowerThirdProps> = ({
  brand, name, title, startFrame = 0, durationInFrames = 90, position = 'bottom-left', theme = 'dark',
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const relativeFrame = frame - startFrame;
  const { colors } = brand;

  const isVisible = relativeFrame >= 0 && relativeFrame <= durationInFrames;
  const enterProgress = spring({ frame: relativeFrame, fps, config: { damping: 18, stiffness: 150, mass: 1 }, durationInFrames: 20 });
  const exitStart = durationInFrames - 20;
  const exitProgress = spring({ frame: relativeFrame - exitStart, fps, config: { damping: 18, stiffness: 200, mass: 1 }, durationInFrames: 20 });

  const slideX = isVisible
    ? relativeFrame <= exitStart
      ? interpolate(enterProgress, [0, 1], [-300, 0])
      : interpolate(exitProgress, [0, 1], [0, -300])
    : -300;

  const backgrounds: Record<string, string> = {
    dark: colors.backgroundDark,
    accent: colors.secondary,
    glass: 'rgba(15, 23, 42, 0.75)',
  };

  const isLeft = position.includes('left');
  const isBottom = position.includes('bottom');

  return (
    <AbsoluteFill style={{ pointerEvents: 'none' }}>
      <div style={{ position: 'absolute', [isBottom ? 'bottom' : 'top']: 64, [isLeft ? 'left' : 'right']: 64, transform: `translateX(${isLeft ? slideX : -slideX}px)` }}>
        <div style={{ width: 4, background: colors.accent, borderRadius: 2, height: title ? 56 : 36, position: 'absolute', left: 0, top: 0 }} />
        <div style={{ background: backgrounds[theme], paddingInline: 20, paddingBlock: 10, marginLeft: 12, borderRadius: 4, backdropFilter: theme === 'glass' ? 'blur(12px)' : undefined, minWidth: 240 }}>
          <BrandText brand={brand} variant="h4" color={colors.textOnDark} weight="semibold">{name}</BrandText>
          {title && <BrandText brand={brand} variant="caption" color={colors.textMuted} weight="regular" style={{ marginTop: 2 }}>{title}</BrandText>}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

---

## 8. Background System

`src/brand/BrandBackground.tsx`

```typescript
import { AbsoluteFill, OffthreadVideo, useCurrentFrame } from 'remotion';
import { BrandConfig, BackgroundVariant } from './brand.types';

interface BrandBackgroundProps {
  brand: BrandConfig;
  variant: BackgroundVariant;
  dark?: boolean;
  videoSrc?: string;
  patternColor?: string;
}

const AnimatedPattern: React.FC<{ primaryColor: string; secondaryColor: string; frame: number }> = ({ primaryColor, secondaryColor, frame }) => {
  const offset = (frame * 0.5) % 40;
  return (
    <AbsoluteFill style={{ background: secondaryColor, backgroundImage: `radial-gradient(circle, ${primaryColor}22 1px, transparent 1px)`, backgroundSize: '40px 40px', backgroundPosition: `${offset}px ${offset}px` }} />
  );
};

export const BrandBackground: React.FC<BrandBackgroundProps> = ({ brand, variant, dark = false, videoSrc, patternColor }) => {
  const frame = useCurrentFrame();
  const { colors } = brand;
  const baseColor = dark ? colors.backgroundDark : colors.background;

  if (variant === 'solid') return <AbsoluteFill style={{ background: baseColor }} />;
  if (variant === 'gradient') return <AbsoluteFill style={{ background: `linear-gradient(${colors.gradient.angle}deg, ${colors.gradient.from}, ${colors.gradient.to})` }} />;
  if (variant === 'video-loop') {
    if (!videoSrc) return <AbsoluteFill style={{ background: baseColor }} />;
    return (
      <AbsoluteFill>
        <OffthreadVideo src={videoSrc} style={{ width: '100%', height: '100%', objectFit: 'cover', opacity: 0.85 }} />
        <AbsoluteFill style={{ background: `${colors.backgroundDark}55` }} />
      </AbsoluteFill>
    );
  }
  if (variant === 'animated-pattern') return <AnimatedPattern frame={frame} primaryColor={patternColor ?? colors.secondary} secondaryColor={baseColor} />;
  return <AbsoluteFill style={{ background: baseColor }} />;
};
```

---

## 9. Typography Component

`src/brand/BrandText.tsx`

```typescript
import { BrandConfig } from './brand.types';

type TextVariant = 'hero' | 'h1' | 'h2' | 'h3' | 'h4' | 'body' | 'caption' | 'label';
type FontWeight = keyof BrandConfig['typography']['weights'];
type TextAlign = 'left' | 'center' | 'right';

interface BrandTextProps {
  brand: BrandConfig;
  variant: TextVariant;
  children: React.ReactNode;
  color?: string;
  weight?: FontWeight;
  align?: TextAlign;
  mono?: boolean;
  uppercase?: boolean;
  letterSpacing?: number;
  lineHeight?: number;
  style?: React.CSSProperties;
  as?: keyof JSX.IntrinsicElements;
}

const isHeading = (v: TextVariant) => ['hero', 'h1', 'h2', 'h3', 'h4'].includes(v);

export const BrandText: React.FC<BrandTextProps> = ({ brand, variant, children, color, weight, align = 'left', mono = false, uppercase = false, letterSpacing, lineHeight, style, as }) => {
  const { typography, colors } = brand;
  const fontSize = typography.scale[variant];
  const resolvedWeight = weight !== undefined ? typography.weights[weight] : isHeading(variant) ? typography.weights.bold : typography.weights.regular;
  const fontFamily = mono ? typography.monoFamily : isHeading(variant) ? typography.headingFamily : typography.bodyFamily;
  const resolvedColor = color ?? colors.text;

  const defaultLineHeights: Record<TextVariant, number> = { hero: 1.05, h1: 1.1, h2: 1.15, h3: 1.2, h4: 1.25, body: 1.6, caption: 1.5, label: 1.4 };
  const tagMap: Record<TextVariant, keyof JSX.IntrinsicElements> = { hero: 'h1', h1: 'h1', h2: 'h2', h3: 'h3', h4: 'h4', body: 'p', caption: 'span', label: 'span' };
  const Tag = (as ?? tagMap[variant]) as React.ElementType;

  return (
    <Tag style={{ fontFamily, fontSize, fontWeight: resolvedWeight, color: resolvedColor, textAlign: align, lineHeight: lineHeight ?? defaultLineHeights[variant], letterSpacing: letterSpacing ?? (isHeading(variant) ? -0.02 * fontSize : 0), textTransform: uppercase ? 'uppercase' : 'none', margin: 0, padding: 0, ...style }}>
      {children}
    </Tag>
  );
};
```

---

## 10. Scene Wrapper (HOC + BrandShell)

`src/brand/withBrand.tsx`

Wrap any composition to automatically prepend intro, append outro, and overlay watermark.

```typescript
import { AbsoluteFill, Sequence, useVideoConfig } from 'remotion';
import { BrandConfig } from './brand.types';
import { IntroSequence } from './IntroSequence';
import { OutroSequence } from './OutroSequence';
import { Watermark } from './Watermark';

interface BrandShellProps {
  brand: BrandConfig;
  children: React.ReactNode;
  showIntro?: boolean;
  showOutro?: boolean;
  showWatermark?: boolean;
  introTitle?: string;
  introSubtitle?: string;
  outroCta?: string;
}

export const BrandShell: React.FC<BrandShellProps> = ({
  brand, children, showIntro = true, showOutro = true, showWatermark = true, introTitle, introSubtitle, outroCta,
}) => {
  const { durationInFrames } = useVideoConfig();
  const { timing } = brand;
  const introDuration = showIntro ? timing.introDuration : 0;
  const outroDuration = showOutro ? timing.outroDuration : 0;
  const contentDuration = durationInFrames - introDuration - outroDuration;

  return (
    <AbsoluteFill>
      {showIntro && <Sequence from={0} durationInFrames={introDuration}><IntroSequence brand={brand} title={introTitle} subtitle={introSubtitle} /></Sequence>}
      <Sequence from={introDuration} durationInFrames={contentDuration}>{children}</Sequence>
      {showOutro && <Sequence from={introDuration + contentDuration} durationInFrames={outroDuration}><OutroSequence brand={brand} customCta={outroCta} /></Sequence>}
      {showWatermark && <Sequence from={introDuration} durationInFrames={contentDuration}><Watermark brand={brand} /></Sequence>}
    </AbsoluteFill>
  );
};
```

---

## 11. Multi-Video Consistency

### Project layout

```
src/
├── brand/
│   ├── brand.config.ts       <- single source of truth
│   ├── brand.types.ts
│   ├── AnimatedLogo.tsx
│   ├── BrandBackground.tsx
│   ├── BrandText.tsx
│   ├── IntroSequence.tsx
│   ├── OutroSequence.tsx
│   ├── LowerThird.tsx
│   ├── Watermark.tsx
│   └── withBrand.tsx
├── compositions/
│   ├── TutorialVideo.tsx
│   ├── ProductDemo.tsx
│   └── CourseIntro.tsx
└── Root.tsx
```

### Root.tsx — register all compositions

```typescript
import { Composition } from 'remotion';
import { BRAND } from './brand/brand.config';
import { TutorialVideo } from './compositions/TutorialVideo';

const fps = 30;
const introDur = BRAND.timing.introDuration;
const outroDur = BRAND.timing.outroDuration;

export const RemotionRoot: React.FC = () => (
  <>
    <Composition
      id="TutorialVideo"
      component={TutorialVideo}
      durationInFrames={introDur + 600 + outroDur}
      fps={fps}
      width={1920}
      height={1080}
      defaultProps={{ brand: BRAND }}
    />
  </>
);
```

### Per-video brand overrides

```typescript
import { BRAND } from '../brand/brand.config';
import { mergeBrand } from '../brand/brand.types';

const SPECIAL_BRAND = mergeBrand(BRAND, {
  colors: { accent: '#F59E0B', secondary: '#D97706' },
  identity: { tagline: 'Special Edition' },
});

export const SpecialVideo: React.FC = () => (
  <BrandShell brand={SPECIAL_BRAND} introTitle="Special Edition Drop">
    {/* ... */}
  </BrandShell>
);
```

---

## 12. Brand Checklist

### Colors
- [ ] Primary color matches brand hex exactly (no #000 defaults)
- [ ] Accent color appears in intro, outro, and lower thirds
- [ ] Background contrast passes WCAG AA for all on-screen text
- [ ] Gradient angle is consistent across all gradient backgrounds

### Typography
- [ ] Heading font loaded via staticFile or web font import
- [ ] BrandText used everywhere — no raw p/span with hardcoded sizes
- [ ] Scale applied consistently: h2 for titles, body for descriptions, caption for meta

### Logo and Watermark
- [ ] Logo file is in /public/ and loads without 404
- [ ] logoAspectRatio matches the actual image ratio
- [ ] Watermark does not overlap critical content

### Intro / Outro
- [ ] introDuration in config matches the actual intro length you want
- [ ] outroDuration in config matches the actual outro length you want
- [ ] Total durationInFrames = introDuration + contentDuration + outroDuration
- [ ] Outro CTA text is correct for this video's goal

### Consistency Across Videos
- [ ] All compositions import from brand.config.ts
- [ ] mergeBrand is the only mechanism for per-video overrides
- [ ] New videos registered in Root.tsx with correct duration math

### Final Render Check
- [ ] Studio preview plays intro -> content -> outro without gaps or flicker
- [ ] Watermark is present during the content segment only
- [ ] Audio does not overpower voiceover (test at 0.8 volume)
- [ ] Lower thirds enter and exit cleanly
```
