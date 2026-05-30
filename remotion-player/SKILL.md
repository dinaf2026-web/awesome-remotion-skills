---
name: remotion-player
description: Embed Remotion videos in React/Next.js websites — Player component setup, controls, event handling, thumbnails, interactive previews, and lazy loading.
origin: community
tags: [remotion, player, embed, nextjs, react, interactive, preview]
---

# remotion-player

Embed Remotion compositions in React/Next.js as interactive, frame-accurate video players — without rendering to a video file.

## 1. When to Use the Player

| Scenario | Use Player | Use Render Pipeline |
|---|---|---|
| Preview during editing | Yes | No |
| Interactive / parameterized embed | Yes | No |
| Final video file (MP4/WebM) | No | Yes (`npx remotion render`) |
| Server-side thumbnail (poster image) | No | Yes (`renderStill`) |
| Blog embed, landing page demo | Yes | Either |
| Email attachment, social upload | No | Yes |

The `@remotion/player` package runs your Remotion composition inside a `<canvas>`-backed React component in the browser. It uses the same composition code as the CLI renderer, so what you see in the Player matches the final render exactly.

## 2. Basic Player Setup

```bash
npm install @remotion/player remotion
```

```tsx
// components/VideoPlayer.tsx
"use client";

import { Player, PlayerRef } from "@remotion/player";
import { useRef } from "react";
import { MyComposition } from "@/remotion/MyComposition";

const COMPOSITION_WIDTH = 1920;
const COMPOSITION_HEIGHT = 1080;
const COMPOSITION_FPS = 30;
const COMPOSITION_DURATION_FRAMES = 300; // 10 seconds at 30fps

export function VideoPlayer() {
  const playerRef = useRef<PlayerRef>(null);

  return (
    <Player
      ref={playerRef}
      component={MyComposition}
      inputProps={{
        title: "Hello World",
        accentColor: "#3b82f6",
      }}
      durationInFrames={COMPOSITION_DURATION_FRAMES}
      fps={COMPOSITION_FPS}
      compositionWidth={COMPOSITION_WIDTH}
      compositionHeight={COMPOSITION_HEIGHT}
      style={{ width: "100%", borderRadius: 8 }}
      controls
      autoPlay={false}
      loop
      clickToPlay={false}
      showVolumeControls
      initiallyShowControls={3000}
    />
  );
}
```

### PlayerRef methods

```tsx
const playerRef = useRef<PlayerRef>(null);

playerRef.current?.play();
playerRef.current?.pause();
playerRef.current?.toggle();
playerRef.current?.seekTo(90);
playerRef.current?.getCurrentFrame();
playerRef.current?.isPlaying();
playerRef.current?.getVolume();
playerRef.current?.setVolume(0.5);
playerRef.current?.isMuted();
playerRef.current?.mute();
playerRef.current?.unmute();
playerRef.current?.requestFullscreen();
playerRef.current?.exitFullscreen();
playerRef.current?.isFullscreen();
```

## 3. Player in Next.js App Router

The Player uses browser APIs and cannot render on the server. Always use `dynamic` with `ssr: false`.

```tsx
// app/demo/page.tsx
import dynamic from "next/dynamic";

const VideoPlayer = dynamic(
  () => import("@/components/VideoPlayer").then((m) => m.VideoPlayer),
  {
    ssr: false,
    loading: () => (
      <div
        style={{ aspectRatio: "16/9", background: "#0f0f0f", borderRadius: 8 }}
        aria-label="Video loading..."
      />
    ),
  }
);

export default function DemoPage() {
  return (
    <main className="max-w-4xl mx-auto px-4 py-12">
      <h1 className="text-3xl font-bold mb-6">Product Demo</h1>
      <VideoPlayer />
    </main>
  );
}
```

If your composition imports heavy libraries (e.g. Three.js, D3), wrap those in dynamic inside the composition itself so they don't bloat the server bundle:

```tsx
// remotion/MyComposition.tsx
import dynamic from "next/dynamic";

const HeavyChart = dynamic(() => import("./HeavyChart"), { ssr: false });
```

## 4. Custom Controls

Hide the built-in controls (`controls={false}`) and build your own using `PlayerRef`.

```tsx
// components/CustomControls.tsx
"use client";

import { Player, PlayerRef } from "@remotion/player";
import { useCallback, useEffect, useRef, useState } from "react";
import { MyComposition } from "@/remotion/MyComposition";

const DURATION = 300;
const FPS = 30;

export function CustomPlayerWithControls() {
  const playerRef = useRef<PlayerRef>(null);
  const [playing, setPlaying] = useState(false);
  const [frame, setFrame] = useState(0);
  const [volume, setVolume] = useState(1);
  const [muted, setMuted] = useState(false);
  const [fullscreen, setFullscreen] = useState(false);

  useEffect(() => {
    const player = playerRef.current;
    if (!player) return;

    const onPlay = () => setPlaying(true);
    const onPause = () => setPlaying(false);
    const onEnded = () => setPlaying(false);
    const onTimeUpdate = ({ detail }: { detail: { frame: number } }) =>
      setFrame(detail.frame);
    const onFullscreenChange = ({ detail }: { detail: { isFullscreen: boolean } }) =>
      setFullscreen(detail.isFullscreen);

    player.addEventListener("play", onPlay);
    player.addEventListener("pause", onPause);
    player.addEventListener("ended", onEnded);
    player.addEventListener("timeupdate", onTimeUpdate);
    player.addEventListener("fullscreenchange", onFullscreenChange);

    return () => {
      player.removeEventListener("play", onPlay);
      player.removeEventListener("pause", onPause);
      player.removeEventListener("ended", onEnded);
      player.removeEventListener("timeupdate", onTimeUpdate);
      player.removeEventListener("fullscreenchange", onFullscreenChange);
    };
  }, []);

  const togglePlay = useCallback(() => { playerRef.current?.toggle(); }, []);

  const handleScrub = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    const f = Number(e.target.value);
    playerRef.current?.seekTo(f);
    setFrame(f);
  }, []);

  const handleVolume = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    const v = Number(e.target.value);
    playerRef.current?.setVolume(v);
    setVolume(v);
    if (v > 0 && muted) { playerRef.current?.unmute(); setMuted(false); }
  }, [muted]);

  const toggleMute = useCallback(() => {
    if (muted) { playerRef.current?.unmute(); } else { playerRef.current?.mute(); }
    setMuted((m) => !m);
  }, [muted]);

  const toggleFullscreen = useCallback(() => {
    if (fullscreen) { playerRef.current?.exitFullscreen(); } else { playerRef.current?.requestFullscreen(); }
  }, [fullscreen]);

  const timeLabel = `${Math.floor(frame / FPS)}s / ${Math.floor(DURATION / FPS)}s`;

  return (
    <div className="relative rounded-xl overflow-hidden bg-black">
      <Player
        ref={playerRef}
        component={MyComposition}
        inputProps={{ title: "Demo" }}
        durationInFrames={DURATION}
        fps={FPS}
        compositionWidth={1920}
        compositionHeight={1080}
        style={{ width: "100%" }}
        controls={false}
        clickToPlay
        loop
      />
      <div className="absolute bottom-0 left-0 right-0 bg-gradient-to-t from-black/80 to-transparent p-4">
        <input
          type="range" min={0} max={DURATION - 1} value={frame}
          onChange={handleScrub}
          className="w-full h-1 accent-blue-500 mb-3 cursor-pointer"
          aria-label="Seek"
        />
        <div className="flex items-center gap-3">
          <button onClick={togglePlay} aria-label={playing ? "Pause" : "Play"} className="w-8 h-8 text-white">
            {playing ? "||" : ">"}
          </button>
          <span className="text-white text-sm tabular-nums">{timeLabel}</span>
          <div className="flex-1" />
          <button onClick={toggleMute} aria-label={muted ? "Unmute" : "Mute"} className="text-white">
            {muted || volume === 0 ? "M" : "V"}
          </button>
          <input
            type="range" min={0} max={1} step={0.05} value={muted ? 0 : volume}
            onChange={handleVolume}
            className="w-20 h-1 accent-blue-500 cursor-pointer"
            aria-label="Volume"
          />
          <button onClick={toggleFullscreen} aria-label={fullscreen ? "Exit fullscreen" : "Fullscreen"} className="text-white">
            FS
          </button>
        </div>
      </div>
    </div>
  );
}
```

## 5. Event Handling

The `PlayerRef` exposes a typed `addEventListener` / `removeEventListener` API. All events fire on the ref object, not the DOM element.

```tsx
useEffect(() => {
  const player = playerRef.current;
  if (!player) return;

  const onPlay = () => analytics.track("video_play");
  const onPause = () => console.log("paused at frame", player.getCurrentFrame());
  const onEnded = () => analytics.track("video_complete");
  const onTimeUpdate = ({ detail }: { detail: { frame: number } }) => {
    if (detail.frame === Math.floor(300 * 0.5)) analytics.track("video_50_percent");
  };
  const onError = ({ detail }: { detail: { error: Error } }) => {
    Sentry.captureException(detail.error);
  };
  const onSeeked = ({ detail }: { detail: { frame: number } }) => {
    console.log("seeked to", detail.frame);
  };
  const onFullscreenChange = ({ detail }: { detail: { isFullscreen: boolean } }) => {
    console.log("fullscreen:", detail.isFullscreen);
  };
  const onVolumeChange = ({ detail }: { detail: { volume: number } }) => {
    console.log("volume:", detail.volume);
  };

  player.addEventListener("play", onPlay);
  player.addEventListener("pause", onPause);
  player.addEventListener("ended", onEnded);
  player.addEventListener("timeupdate", onTimeUpdate);
  player.addEventListener("error", onError);
  player.addEventListener("seeked", onSeeked);
  player.addEventListener("fullscreenchange", onFullscreenChange);
  player.addEventListener("volumechange", onVolumeChange);

  return () => {
    player.removeEventListener("play", onPlay);
    player.removeEventListener("pause", onPause);
    player.removeEventListener("ended", onEnded);
    player.removeEventListener("timeupdate", onTimeUpdate);
    player.removeEventListener("error", onError);
    player.removeEventListener("seeked", onSeeked);
    player.removeEventListener("fullscreenchange", onFullscreenChange);
    player.removeEventListener("volumechange", onVolumeChange);
  };
}, []);
```

## 6. Thumbnail Generation

Use `@remotion/renderer`'s `renderStill` on the server (Node.js route handler or build script) to generate a poster image from a specific frame. Never call `renderStill` in a Client Component.

```ts
// app/api/thumbnail/route.ts
import { renderStill } from "@remotion/renderer";
import { bundle } from "@remotion/bundler";
import path from "path";
import { NextRequest, NextResponse } from "next/server";

let bundled: string | null = null;

async function getBundle() {
  if (bundled) return bundled;
  bundled = await bundle({ entryPoint: path.join(process.cwd(), "remotion/index.ts") });
  return bundled;
}

export async function GET(req: NextRequest) {
  const { searchParams } = new URL(req.url);
  const frame = Number(searchParams.get("frame") ?? "0");
  const title = searchParams.get("title") ?? "Untitled";
  const serveUrl = await getBundle();

  const { buffer } = await renderStill({
    composition: {
      id: "MyComposition",
      width: 1920, height: 1080, fps: 30, durationInFrames: 300,
      defaultProps: { title: "", accentColor: "#3b82f6" },
      props: { title, accentColor: "#3b82f6" },
    },
    serveUrl,
    frame,
    output: null,
    imageFormat: "png",
    inputProps: { title, accentColor: "#3b82f6" },
  });

  return new NextResponse(buffer, {
    headers: {
      "Content-Type": "image/png",
      "Cache-Control": "public, max-age=86400, immutable",
    },
  });
}
```

Use the thumbnail as a poster:

```tsx
// components/PlayerWithPoster.tsx
"use client";

import { Player } from "@remotion/player";
import { MyComposition } from "@/remotion/MyComposition";

export function PlayerWithPoster({ title, posterFrame = 0 }: { title: string; posterFrame?: number }) {
  const posterUrl = `/api/thumbnail?frame=${posterFrame}&title=${encodeURIComponent(title)}`;

  return (
    <Player
      component={MyComposition}
      inputProps={{ title, accentColor: "#3b82f6" }}
      durationInFrames={300}
      fps={30}
      compositionWidth={1920}
      compositionHeight={1080}
      style={{ width: "100%" }}
      controls
      renderPoster={() => (
        <img src={posterUrl} alt={`${title} preview`} style={{ width: "100%", height: "100%", objectFit: "cover" }} />
      )}
      showPosterWhenUnplayed
      showPosterWhenEnded
      showPosterWhenPaused={false}
    />
  );
}
```

## 7. Interactive Video

Pass live `inputProps` to re-render the composition as the user changes parameters. The Player re-renders immediately without re-mounting.

```tsx
// components/InteractivePlayer.tsx
"use client";

import { Player } from "@remotion/player";
import { useMemo, useState } from "react";
import { MyComposition } from "@/remotion/MyComposition";

const PRESETS = [
  { label: "Blue", accentColor: "#3b82f6" },
  { label: "Purple", accentColor: "#8b5cf6" },
  { label: "Green", accentColor: "#10b981" },
  { label: "Orange", accentColor: "#f59e0b" },
];

export function InteractivePlayer() {
  const [title, setTitle] = useState("Hello World");
  const [accentColor, setAccentColor] = useState("#3b82f6");

  const inputProps = useMemo(() => ({ title, accentColor }), [title, accentColor]);

  return (
    <div className="space-y-6">
      <Player
        component={MyComposition}
        inputProps={inputProps}
        durationInFrames={300}
        fps={30}
        compositionWidth={1920}
        compositionHeight={1080}
        style={{ width: "100%" }}
        controls loop autoPlay muted
      />
      <div className="space-y-4 p-4 bg-gray-50 rounded-xl">
        <div>
          <label htmlFor="title-input" className="block text-sm font-medium mb-1">Title</label>
          <input
            id="title-input" type="text" value={title}
            onChange={(e) => setTitle(e.target.value)} maxLength={60}
            className="w-full border rounded-lg px-3 py-2 text-sm"
          />
        </div>
        <div>
          <p className="text-sm font-medium mb-2">Accent color</p>
          <div className="flex gap-2">
            {PRESETS.map((p) => (
              <button key={p.label} onClick={() => setAccentColor(p.accentColor)} aria-label={p.label}
                style={{ background: p.accentColor }}
                className={`w-8 h-8 rounded-full border-2 transition-transform ${accentColor === p.accentColor ? "border-gray-900 scale-110" : "border-transparent"}`}
              />
            ))}
            <input type="color" value={accentColor} onChange={(e) => setAccentColor(e.target.value)}
              className="w-8 h-8 rounded-full cursor-pointer border-0" aria-label="Custom color" />
          </div>
        </div>
      </div>
    </div>
  );
}
```

## 8. Responsive Player

`compositionWidth` / `compositionHeight` define the internal canvas resolution. Display size is set with `style`. Use CSS `aspect-ratio` to prevent layout shift.

```tsx
// components/ResponsivePlayer.tsx
"use client";

import { Player } from "@remotion/player";
import { MyComposition } from "@/remotion/MyComposition";

export function ResponsivePlayer({ compositionWidth = 1920, compositionHeight = 1080 }) {
  return (
    <div style={{ aspectRatio: `${compositionWidth} / ${compositionHeight}`, width: "100%", background: "#000", borderRadius: 8, overflow: "hidden" }}>
      <Player
        component={MyComposition}
        inputProps={{ title: "Responsive" }}
        durationInFrames={300} fps={30}
        compositionWidth={compositionWidth}
        compositionHeight={compositionHeight}
        style={{ width: "100%", height: "100%" }}
        controls
      />
    </div>
  );
}

// Vertical short: <ResponsivePlayer compositionWidth={1080} compositionHeight={1920} />
// Square:         <ResponsivePlayer compositionWidth={1080} compositionHeight={1080} />
```

## 9. Lazy Loading

Load the Player only when it enters the viewport to avoid loading Remotion's JS bundle for off-screen content.

```tsx
// components/LazyPlayer.tsx
"use client";

import { useEffect, useRef, useState } from "react";
import dynamic from "next/dynamic";
import { MyComposition } from "@/remotion/MyComposition";

const Player = dynamic(() => import("@remotion/player").then((m) => m.Player), { ssr: false });

export function LazyPlayer({ title }: { title: string }) {
  const containerRef = useRef<HTMLDivElement>(null);
  const [isVisible, setIsVisible] = useState(false);
  const [hasBeenVisible, setHasBeenVisible] = useState(false);

  useEffect(() => {
    const el = containerRef.current;
    if (!el) return;
    const observer = new IntersectionObserver(
      ([entry]) => {
        setIsVisible(entry.isIntersecting);
        if (entry.isIntersecting) setHasBeenVisible(true);
      },
      { rootMargin: "200px 0px", threshold: 0 }
    );
    observer.observe(el);
    return () => observer.disconnect();
  }, []);

  return (
    <div ref={containerRef} style={{ aspectRatio: "16/9", background: "#0f0f0f", borderRadius: 8 }}>
      {hasBeenVisible ? (
        <Player
          component={MyComposition}
          inputProps={{ title }}
          durationInFrames={300} fps={30}
          compositionWidth={1920} compositionHeight={1080}
          style={{ width: "100%", height: "100%" }}
          controls autoPlay={isVisible} muted loop
        />
      ) : (
        <div style={{ width: "100%", height: "100%" }} aria-label="Video loading..." />
      )}
    </div>
  );
}
```

## 10. Multiple Players

When multiple players exist on one page, pause all others when one starts playing.

```tsx
// components/PlayerGrid.tsx
"use client";

import { Player, PlayerRef } from "@remotion/player";
import { useCallback, useEffect, useRef } from "react";
import { MyComposition } from "@/remotion/MyComposition";

const VIDEOS = [
  { id: "a", title: "Episode 1", accentColor: "#3b82f6" },
  { id: "b", title: "Episode 2", accentColor: "#8b5cf6" },
  { id: "c", title: "Episode 3", accentColor: "#10b981" },
];

export function PlayerGrid() {
  const playerRefs = useRef<Map<string, PlayerRef>>(new Map());

  const setRef = useCallback((id: string) => (ref: PlayerRef | null) => {
    if (ref) { playerRefs.current.set(id, ref); } else { playerRefs.current.delete(id); }
  }, []);

  const handlePlay = useCallback((activeId: string) => {
    playerRefs.current.forEach((player, id) => {
      if (id !== activeId && player.isPlaying()) player.pause();
    });
  }, []);

  return (
    <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
      {VIDEOS.map((video) => (
        <PlayerCard key={video.id} video={video} refCallback={setRef(video.id)} onPlay={() => handlePlay(video.id)} />
      ))}
    </div>
  );
}

function PlayerCard({ video, refCallback, onPlay }: { video: typeof VIDEOS[0]; refCallback: (ref: PlayerRef | null) => void; onPlay: () => void }) {
  const playerRef = useRef<PlayerRef>(null);

  const combinedRef = useCallback((ref: PlayerRef | null) => {
    (playerRef as React.MutableRefObject<PlayerRef | null>).current = ref;
    refCallback(ref);
  }, [refCallback]);

  useEffect(() => {
    const player = playerRef.current;
    if (!player) return;
    player.addEventListener("play", onPlay);
    return () => player.removeEventListener("play", onPlay);
  }, [onPlay]);

  return (
    <div className="rounded-xl overflow-hidden shadow-md">
      <Player
        ref={combinedRef}
        component={MyComposition}
        inputProps={{ title: video.title, accentColor: video.accentColor }}
        durationInFrames={300} fps={30}
        compositionWidth={1920} compositionHeight={1080}
        style={{ width: "100%" }}
        controls clickToPlay
      />
      <p className="text-sm font-medium p-3">{video.title}</p>
    </div>
  );
}
```

## 11. Mobile Considerations

### Autoplay restrictions

Browsers block autoplay with audio. Always use `muted` for autoplay:

```tsx
// Safe
<Player autoPlay muted loop ... />

// WRONG — browser will block on mobile
// <Player autoPlay loop ... />
```

### Touch-friendly controls

Ensure tap targets are at least 44x44 px. Use `pointer-events: none` on the canvas if tap-to-play conflicts with custom buttons.

```tsx
// components/MobilePlayer.tsx
"use client";

import { Player, PlayerRef } from "@remotion/player";
import { useEffect, useRef, useState } from "react";
import { MyComposition } from "@/remotion/MyComposition";

export function MobilePlayer() {
  const playerRef = useRef<PlayerRef>(null);
  const [userInteracted, setUserInteracted] = useState(false);

  const handleFirstTap = () => {
    if (!userInteracted) { setUserInteracted(true); playerRef.current?.unmute(); }
  };

  return (
    <div onClick={handleFirstTap} onTouchStart={handleFirstTap} style={{ position: "relative" }}>
      <Player
        ref={playerRef}
        component={MyComposition}
        inputProps={{ title: "Mobile Demo" }}
        durationInFrames={300} fps={30}
        compositionWidth={1080} compositionHeight={1920}
        style={{ width: "100%" }}
        controls muted={!userInteracted} autoPlay loop clickToPlay={false}
      />
      {!userInteracted && (
        <div style={{ position: "absolute", bottom: 60, left: "50%", transform: "translateX(-50%)", background: "rgba(0,0,0,0.7)", color: "#fff", padding: "6px 14px", borderRadius: 999, fontSize: 13, pointerEvents: "none" }}>
          Tap to unmute
        </div>
      )}
    </div>
  );
}
```

### Reduced motion

Respect `prefers-reduced-motion` inside the composition:

```tsx
function useReducedMotion() {
  if (typeof window === "undefined") return false;
  return window.matchMedia("(prefers-reduced-motion: reduce)").matches;
}

export function MyComposition({ title }: { title: string }) {
  const frame = useCurrentFrame();
  const reducedMotion = useReducedMotion();
  const opacity = reducedMotion ? 1 : interpolate(frame, [0, 20], [0, 1], { extrapolateRight: "clamp" });
  return <div style={{ opacity, fontFamily: "sans-serif", fontSize: 80 }}>{title}</div>;
}
```

## 12. Anti-Patterns

### Never: SSR the Player

```tsx
// WRONG
import { Player } from "@remotion/player"; // will throw "window is not defined"

// CORRECT
const Player = dynamic(() => import("@remotion/player").then((m) => m.Player), { ssr: false });
```

### Never: Skip a loading state

```tsx
// WRONG — causes layout shift
const PlayerComponent = dynamic(() => import("@/components/VideoPlayer"), { ssr: false });

// CORRECT — reserve space
const PlayerComponent = dynamic(() => import("@/components/VideoPlayer"), {
  ssr: false,
  loading: () => <div style={{ aspectRatio: "16/9", background: "#111", borderRadius: 8 }} />,
});
```

### Never: Mutate inputProps on every render without useMemo

```tsx
// WRONG — new object reference on every render
function BadPlayer({ title }: { title: string }) {
  return <Player inputProps={{ title, accentColor: "#3b82f6" }} ... />;
}

// CORRECT — stable reference
function GoodPlayer({ title }: { title: string }) {
  const inputProps = useMemo(() => ({ title, accentColor: "#3b82f6" }), [title]);
  return <Player inputProps={inputProps} ... />;
}
```

### Never: Call renderStill in a Client Component

```tsx
// WRONG
"use client";
import { renderStill } from "@remotion/renderer"; // crashes — uses Node.js APIs

// CORRECT — only in a Route Handler or build script (see Section 6)
```

### Never: Forget to clean up event listeners

```tsx
// WRONG — memory leak
useEffect(() => {
  playerRef.current?.addEventListener("play", handler);
}, []);

// CORRECT
useEffect(() => {
  const player = playerRef.current;
  if (!player) return;
  player.addEventListener("play", handler);
  return () => player.removeEventListener("play", handler);
}, []);
```

### Never: Set compositionWidth/Height to display size

```tsx
// WRONG — low-res canvas scaled up looks blurry
<Player compositionWidth={640} compositionHeight={360} style={{ width: 1280 }} ... />

// CORRECT — native resolution; CSS handles display scaling
<Player compositionWidth={1920} compositionHeight={1080} style={{ width: "100%" }} ... />
```
