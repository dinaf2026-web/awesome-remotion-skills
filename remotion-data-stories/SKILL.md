---
name: remotion-data-stories
description: Animate data and charts in Remotion — bar chart races, line chart draws, stat counters, map animations, and data-driven video storytelling patterns.
origin: community
tags: [remotion, data-visualization, charts, animation, bar-race, storytelling, data]
---

# remotion-data-stories

Animate data and charts in Remotion. Covers bar chart races, line chart draws, stat counters, map animations, and data-driven video storytelling patterns.

---

## 1. Data Story Principles

### Pacing

Each data beat needs breathing room. A reveal that fires and cuts immediately loses the audience. Structure sequences as: **enter → hold → exit**.

| Phase | Duration guideline |
|---|---|
| Enter (animate in) | 20–40 frames |
| Hold (readable) | 60–90 frames |
| Exit or transition | 15–20 frames |

A 30 fps video at 3 seconds per beat = 90 frames. Budget accordingly.

### Focus

One insight per scene. Do not animate six bars simultaneously and expect viewers to absorb rankings. Sequence reveals: show the lowest bar first, build to the winner, hold on the winner.

### Annotation Timing

Annotations should appear **after** the data element is fully visible — never during the enter animation. Wait until the spring settles (frame >= enterStart + 35) before fading in callout text.

### Narration Sync

Map your script word-by-word to frames before building. A standard voiceover pace is 2.5-3 words per second = ~15 frames per word at 30 fps. Build a `NARRATION_CUES` array and drive all reveals from it.

```ts
// narration-cues.ts
export const NARRATION_CUES = [
  { frame: 0,   label: "In 2023..." },
  { frame: 45,  label: "Revenue hit a record..." },
  { frame: 90,  label: "But costs rose faster." },
  { frame: 150, label: "The gap tells the real story." },
] as const;
```

---

## 2. Animated Bar Chart

Bars grow from zero, labels fade in, value counters tick up simultaneously.

```tsx
// AnimatedBarChart.tsx
import { useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";

interface Bar {
  label: string;
  value: number;
  color: string;
}

interface AnimatedBarChartProps {
  bars: Bar[];
  maxValue: number;
  startFrame?: number;
  chartHeight?: number;
}

const BAR_WIDTH = 80;
const BAR_GAP = 24;

export const AnimatedBarChart: React.FC<AnimatedBarChartProps> = ({
  bars,
  maxValue,
  startFrame = 0,
  chartHeight = 300,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <div style={{ display: "flex", alignItems: "flex-end", gap: BAR_GAP, height: chartHeight + 60 }}>
      {bars.map((bar, i) => {
        const barStart = startFrame + i * 8;
        const localFrame = Math.max(0, frame - barStart);

        const progress = spring({
          frame: localFrame,
          fps,
          config: { damping: 14, stiffness: 80, mass: 1 },
          durationInFrames: 45,
        });

        const barHeight = (bar.value / maxValue) * chartHeight * progress;

        const labelOpacity = interpolate(localFrame, [30, 45], [0, 1], {
          extrapolateLeft: "clamp",
          extrapolateRight: "clamp",
        });

        const displayValue = Math.round(bar.value * progress);

        return (
          <div key={bar.label} style={{ display: "flex", flexDirection: "column", alignItems: "center" }}>
            <div style={{ opacity: labelOpacity, fontSize: 18, fontWeight: 700, color: bar.color, marginBottom: 6, fontVariantNumeric: "tabular-nums" }}>
              {displayValue.toLocaleString()}
            </div>
            <div style={{ width: BAR_WIDTH, height: barHeight, backgroundColor: bar.color, borderRadius: "6px 6px 0 0", transition: "none" }} />
            <div style={{ opacity: labelOpacity, marginTop: 8, fontSize: 14, color: "#555", textAlign: "center", width: BAR_WIDTH }}>
              {bar.label}
            </div>
          </div>
        );
      })}
    </div>
  );
};
```

---

## 3. Bar Chart Race

Bars reorder over time as rankings change.

```tsx
// BarChartRace.tsx
import { useCurrentFrame, useVideoConfig } from "remotion";

interface RaceEntry {
  id: string;
  label: string;
  color: string;
  keyframes: Record<number, number>;
}

function interpolateValue(entry: RaceEntry, frame: number): number {
  const keyframeTimes = Object.keys(entry.keyframes).map(Number).sort((a, b) => a - b);
  if (frame <= keyframeTimes[0]) return entry.keyframes[keyframeTimes[0]];
  if (frame >= keyframeTimes[keyframeTimes.length - 1]) return entry.keyframes[keyframeTimes[keyframeTimes.length - 1]];
  const prevTime = keyframeTimes.filter((t) => t <= frame).at(-1)!;
  const nextTime = keyframeTimes.find((t) => t > frame)!;
  const t = (frame - prevTime) / (nextTime - prevTime);
  return entry.keyframes[prevTime] + (entry.keyframes[nextTime] - entry.keyframes[prevTime]) * t;
}

export const BarChartRace: React.FC<{ entries: RaceEntry[]; maxBars?: number; barHeight?: number; barGap?: number; maxWidth?: number }> = ({
  entries, maxBars = 8, barHeight = 48, barGap = 12, maxWidth = 600,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const currentValues = entries.map((e) => ({ ...e, currentValue: interpolateValue(e, frame) }));
  const sorted = [...currentValues].sort((a, b) => b.currentValue - a.currentValue).slice(0, maxBars);
  const maxValue = sorted[0]?.currentValue ?? 1;

  return (
    <div style={{ position: "relative", height: maxBars * (barHeight + barGap) }}>
      {sorted.map((entry, rank) => {
        const targetY = rank * (barHeight + barGap);
        const barWidth = (entry.currentValue / maxValue) * maxWidth;
        return (
          <div key={entry.id} style={{ position: "absolute", top: targetY, left: 0, width: "100%", display: "flex", alignItems: "center", gap: 10, transition: `top ${(1 / fps) * 8 * 1000}ms cubic-bezier(0.4,0,0.2,1)` }}>
            <div style={{ width: 28, textAlign: "right", fontSize: 14, fontWeight: 700, color: "#999", flexShrink: 0 }}>#{rank + 1}</div>
            <div style={{ height: barHeight, width: barWidth, backgroundColor: entry.color, borderRadius: "0 6px 6px 0", transition: `width ${(1 / fps) * 2 * 1000}ms linear`, display: "flex", alignItems: "center", paddingLeft: 10 }}>
              <span style={{ color: "#fff", fontWeight: 700, fontSize: 14 }}>{entry.label}</span>
            </div>
            <div style={{ fontSize: 16, fontWeight: 700, color: entry.color, fontVariantNumeric: "tabular-nums" }}>{Math.round(entry.currentValue).toLocaleString()}</div>
          </div>
        );
      })}
    </div>
  );
};
```

---

## 4. Line Chart Draw

The line traces across the screen in sync with the timeline.

```tsx
// LineChartDraw.tsx
import { useCurrentFrame, interpolate } from "remotion";

interface Point { x: number; y: number; }

export const LineChartDraw: React.FC<{
  points: Point[];
  startFrame: number;
  durationFrames: number;
  width?: number;
  height?: number;
  strokeColor?: string;
  strokeWidth?: number;
  showDots?: boolean;
  fillArea?: boolean;
}> = ({ points, startFrame, durationFrames, width = 600, height = 300, strokeColor = "#4F8EF7", strokeWidth = 3, showDots = true, fillArea = false }) => {
  const frame = useCurrentFrame();
  const localFrame = Math.max(0, frame - startFrame);
  const progress = interpolate(localFrame, [0, durationFrames], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });

  const mapped = points.map((p) => ({ x: p.x * width, y: height - p.y * height }));
  const path = mapped.map((p, i) => `${i === 0 ? "M" : "L"} ${p.x.toFixed(2)} ${p.y.toFixed(2)}`).join(" ");
  const estimatedLength = width * 1.5;
  const dashOffset = estimatedLength * (1 - progress);

  return (
    <svg width={width} height={height} style={{ overflow: "visible" }}>
      {fillArea && <path d={path + ` L ${points[points.length - 1].x * width} ${height} L 0 ${height} Z`} fill={strokeColor} fillOpacity={0.15} style={{ clipPath: `inset(0 ${(1 - progress) * 100}% 0 0)` }} />}
      <path d={path} fill="none" stroke={strokeColor} strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" strokeDasharray={estimatedLength} strokeDashoffset={dashOffset} />
      {showDots && points.map((p, i) => (
        <circle key={i} cx={p.x * width} cy={height - p.y * height} r={5} fill="#fff" stroke={strokeColor} strokeWidth={2} opacity={progress >= i / (points.length - 1) ? 1 : 0} />
      ))}
    </svg>
  );
};
```

---

## 5. Stat Counter

Large number counts up from 0 to its final value with easing.

```tsx
// StatCounter.tsx
import { useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";

export const StatCounter: React.FC<{
  value: number;
  prefix?: string;
  suffix?: string;
  startFrame?: number;
  durationFrames?: number;
  decimals?: number;
  fontSize?: number;
  color?: string;
  label?: string;
}> = ({ value, prefix = "", suffix = "", startFrame = 0, durationFrames = 60, decimals = 0, fontSize = 96, color = "#1a1a1a", label }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const localFrame = Math.max(0, frame - startFrame);
  const progress = spring({ frame: localFrame, fps, config: { damping: 20, stiffness: 60, mass: 1 }, durationInFrames: durationFrames });
  const displayValue = value * progress;
  const labelOpacity = interpolate(localFrame, [0, 20], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });

  return (
    <div style={{ textAlign: "center" }}>
      <div style={{ fontSize, fontWeight: 900, color, fontVariantNumeric: "tabular-nums", lineHeight: 1, letterSpacing: "-0.02em" }}>
        {prefix}{displayValue.toLocaleString(undefined, { minimumFractionDigits: decimals, maximumFractionDigits: decimals })}{suffix}
      </div>
      {label && <div style={{ marginTop: 12, fontSize: fontSize * 0.2, color: "#777", fontWeight: 500, opacity: labelOpacity, textTransform: "uppercase", letterSpacing: "0.08em" }}>{label}</div>}
    </div>
  );
};
```

**Usage:**

```tsx
<StatCounter value={2_847_391} suffix=" users" startFrame={45} durationFrames={90} fontSize={120} color="#4F8EF7" label="Active Monthly Users" />
```

---

## 6. Comparison Split

Two stats animate side by side; the winner highlights with a scale pop.

```tsx
// ComparisonSplit.tsx
import { useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";

interface ComparisonStat { label: string; value: number; color: string; }

export const ComparisonSplit: React.FC<{
  left: ComparisonStat;
  right: ComparisonStat;
  prefix?: string;
  suffix?: string;
  startFrame?: number;
  highlightWinner?: boolean;
}> = ({ left, right, prefix = "", suffix = "", startFrame = 0, highlightWinner = true }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const localFrame = Math.max(0, frame - startFrame);
  const countProgress = spring({ frame: localFrame, fps, config: { damping: 18, stiffness: 60 }, durationInFrames: 60 });
  const winnerFrame = Math.max(0, frame - (startFrame + 70));
  const winnerScale = spring({ frame: winnerFrame, fps, config: { damping: 10, stiffness: 200 }, durationInFrames: 20 });
  const isLeftWinner = left.value >= right.value;
  const dividerOpacity = interpolate(localFrame, [20, 40], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });

  const renderStat = (stat: ComparisonStat, isWinner: boolean) => {
    const scale = highlightWinner && isWinner ? 1 + 0.08 * winnerScale : 1;
    return (
      <div style={{ flex: 1, display: "flex", flexDirection: "column", alignItems: "center", transform: `scale(${scale})` }}>
        <div style={{ fontSize: 88, fontWeight: 900, color: stat.color, fontVariantNumeric: "tabular-nums", lineHeight: 1, opacity: highlightWinner && !isWinner ? 0.45 : 1 }}>
          {prefix}{Math.round(stat.value * countProgress).toLocaleString()}{suffix}
        </div>
        <div style={{ marginTop: 10, fontSize: 18, color: "#555", fontWeight: 500, textAlign: "center" }}>{stat.label}</div>
        {highlightWinner && isWinner && <div style={{ marginTop: 12, fontSize: 13, color: stat.color, fontWeight: 700, textTransform: "uppercase", letterSpacing: "0.1em", opacity: winnerScale }}>Winner</div>}
      </div>
    );
  };

  return (
    <div style={{ display: "flex", alignItems: "center", gap: 40 }}>
      {renderStat(left, isLeftWinner)}
      <div style={{ width: 2, height: 120, backgroundColor: "#ddd", opacity: dividerOpacity, flexShrink: 0 }} />
      {renderStat(right, !isLeftWinner)}
    </div>
  );
};
```

---

## 7. Annotation System

Callout arrows and labels appear at specific frames, positioned relative to data elements.

```tsx
// AnnotationLayer.tsx
import { useCurrentFrame, useVideoConfig, interpolate, spring } from "remotion";

interface Annotation { id: string; frame: number; x: number; y: number; dx: number; dy: number; text: string; subtext?: string; color?: string; }

export const AnnotationLayer: React.FC<{ annotations: Annotation[]; containerWidth: number; containerHeight: number }> = ({ annotations, containerWidth, containerHeight }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const DEFAULT_COLOR = "#F7914F";

  return (
    <svg width={containerWidth} height={containerHeight} style={{ position: "absolute", top: 0, left: 0, pointerEvents: "none" }}>
      {annotations.map((ann) => {
        const localFrame = Math.max(0, frame - ann.frame);
        const color = ann.color ?? DEFAULT_COLOR;
        const progress = spring({ frame: localFrame, fps, config: { damping: 14, stiffness: 120 }, durationInFrames: 25 });
        const textOpacity = interpolate(localFrame, [20, 35], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
        const lineEndX = ann.x + ann.dx * progress;
        const lineEndY = ann.y + ann.dy * progress;
        const labelX = ann.x + ann.dx;
        const labelY = ann.y + ann.dy;

        return (
          <g key={ann.id}>
            <circle cx={ann.x} cy={ann.y} r={5 * progress} fill={color} />
            <line x1={ann.x} y1={ann.y} x2={lineEndX} y2={lineEndY} stroke={color} strokeWidth={2} strokeDasharray="4 3" />
            {progress > 0.9 && <circle cx={lineEndX} cy={lineEndY} r={3} fill={color} />}
            <foreignObject x={ann.dx > 0 ? labelX : labelX - 180} y={labelY - 30} width={180} height={60} opacity={textOpacity}>
              <div style={{ backgroundColor: "#fff", border: `2px solid ${color}`, borderRadius: 8, padding: "6px 10px", boxShadow: "0 2px 12px rgba(0,0,0,0.1)" }}>
                <div style={{ fontSize: 13, fontWeight: 700, color }}>{ann.text}</div>
                {ann.subtext && <div style={{ fontSize: 11, color: "#666", marginTop: 2 }}>{ann.subtext}</div>}
              </div>
            </foreignObject>
          </g>
        );
      })}
    </svg>
  );
};
```

---

## 8. Color Schemes for Data

```ts
// color-schemes.ts

export const CATEGORICAL_PALETTE = [
  "#4F8EF7", "#F7914F", "#52C97E", "#F75F5F",
  "#A78BFA", "#F7D050", "#38BDF8", "#FB7185",
] as const;

export const CATEGORICAL_MUTED = CATEGORICAL_PALETTE.map((c) => c + "66");

export const SEQUENTIAL_BLUE = ["#dbeafe", "#93c5fd", "#60a5fa", "#2563eb", "#1e3a8a"] as const;
export const SEQUENTIAL_GREEN = ["#dcfce7", "#86efac", "#4ade80", "#16a34a", "#14532d"] as const;
export const SEQUENTIAL_ORANGE = ["#ffedd5", "#fdba74", "#fb923c", "#ea580c", "#7c2d12"] as const;
export const DIVERGING_RED_BLUE = ["#ef4444", "#fca5a5", "#e5e7eb", "#93c5fd", "#2563eb"] as const;

export const SEMANTIC = {
  positive: "#52C97E",
  negative: "#F75F5F",
  neutral: "#94a3b8",
  highlight: "#F7914F",
  comparison_a: "#4F8EF7",
  comparison_b: "#F7914F",
} as const;

export function categoricalColor(index: number): string {
  return CATEGORICAL_PALETTE[index % CATEGORICAL_PALETTE.length];
}

export function sequentialColor(value: number, min: number, max: number, palette: readonly string[] = SEQUENTIAL_BLUE): string {
  const t = Math.max(0, Math.min(1, (value - min) / (max - min)));
  return palette[Math.round(t * (palette.length - 1))];
}

export function divergingColor(value: number, min: number, max: number, palette: readonly string[] = DIVERGING_RED_BLUE): string {
  const t = Math.max(0, Math.min(1, (value - min) / (max - min)));
  return palette[Math.round(t * (palette.length - 1))];
}
```

```ts
// theme.ts
export const THEMES = {
  light: { background: "#ffffff", surface: "#f8fafc", text: "#1a1a1a", textMuted: "#64748b", border: "#e2e8f0", axisLine: "#cbd5e1" },
  dark: { background: "#0f172a", surface: "#1e293b", text: "#f1f5f9", textMuted: "#94a3b8", border: "#334155", axisLine: "#475569" },
} as const;

export type Theme = keyof typeof THEMES;
```

---

## 9. Data-Driven Props Pattern

Pass CSV/JSON data as Remotion props. Auto-generate sequences from structured data.

```ts
// data-props.ts
import { z } from "zod";

export const DataStorySchema = z.object({
  title: z.string(),
  theme: z.enum(["light", "dark"]),
  bars: z.array(z.object({ label: z.string(), value: z.number(), color: z.string().optional() })),
  linePoints: z.array(z.object({ x: z.number(), y: z.number() })).optional(),
  statValue: z.number().optional(),
  statLabel: z.string().optional(),
});

export type DataStoryProps = z.infer<typeof DataStorySchema>;
```

```ts
// auto-color.ts
import { CATEGORICAL_PALETTE } from "./color-schemes";

export function assignColors<T extends { color?: string }>(items: T[]): (T & { color: string })[] {
  return items.map((item, i) => ({ ...item, color: item.color ?? CATEGORICAL_PALETTE[i % CATEGORICAL_PALETTE.length] }));
}
```

---

## Quick-Start Checklist

- [ ] Install: `npm install remotion @remotion/renderer zod`
- [ ] Create `src/compositions/` directory for each data story
- [ ] Define your `DataStorySchema` with Zod before building components
- [ ] Map voiceover script to frame numbers before building reveals
- [ ] Use `spring()` for all entrances — never raw `interpolate()` alone
- [ ] Stagger related elements by 8-12 frames to avoid simultaneous pops
- [ ] Hold every data beat for at least 60 frames (2 s at 30 fps)
- [ ] Annotations appear after `revealFrame + 30` minimum
- [ ] Assign colors from `CATEGORICAL_PALETTE` — never hardcode ad-hoc hex values
- [ ] Test at 1x and 0.5x playback speed; pacing should feel right at both
