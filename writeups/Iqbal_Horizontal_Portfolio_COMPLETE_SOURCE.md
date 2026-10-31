# IQBAL//SEC — Complete Project Source
This document contains the complete contents of every text/code file in the rebuilt project. The binary CV is included in the ZIP at `public/Muhammad-Iqbal-Latif-CV.pdf`. The previous unused `components/TiltCard.tsx` file was removed.

## `.env.example`

```bash
# Optional: set the deployed origin so Open Graph metadata resolves to the production domain.
NEXT_PUBLIC_SITE_URL=https://your-domain.example
```

## `.gitignore`

```gitignore
node_modules/
.next/
out/
*.log
.DS_Store
.env*
!.env.example
```

## `README.md`

```markdown
# IQBAL//SEC — Horizontal Cyber Portfolio

A chaptered, purple-native cyber-security portfolio built with Next.js 16, React 19, TypeScript, Motion, React Three Fiber, and plain CSS.

## Motion architecture

The desktop experience uses Motion's `useScroll`, `useSpring`, and `useTransform` to map a tall vertical scroll region onto a sticky horizontal track. No GSAP dependency was added because Motion already provides the required scroll progress, transforms, spring smoothing, reduced-motion integration, and React-native lifecycle behavior. On narrow screens, coarse pointers, and reduced-motion configurations, the same panels render as an intentional vertical document without pinning.

## Features

- Six horizontal case-file chapters on desktop
- Deliberate vertical fallback for touch and mobile devices
- Skippable secure-session boot sequence
- Scroll-reactive R3F cyber bloom, packet rings, and particle field
- 3D chapter entry transitions and project parallax
- Expandable project case files
- Scroll-driven tool orbit
- Custom reticle cursor and magnetic calls to action
- Violet gradient mesh, scan effects, and glitch text reveals
- Responsive navigation with chapter markers
- Keyboard-accessible controls and reduced-motion support
- No Tailwind and no additional runtime dependency

## Run locally

```bash
npm install
npm run dev
```

Open `http://localhost:3000`.

## Validate

```bash
npm run lint
npm run typecheck
npm run build
```

## Deployment

Set `NEXT_PUBLIC_SITE_URL` to the deployed origin and deploy to Vercel. The App Router metadata, manifest, local fonts, Open Graph image, and downloadable CV are already included.
```

## `app/globals.css`

```css
:root {
  --void: #0b0615;
  --deep-violet: #150a2e;
  --indigo-core: #2e1065;
  --royal-violet: #5b21b6;
  --electric-violet: #7c3aed;
  --neon-violet: #9333ea;
  --plasma-lilac: #a855f7;
  --ultraviolet-glow: #c084fc;
  --lavender-mist: #e9d5ff;
  --ink: #f5f3ff;
  --ink-muted: #b9a8d9;
  --font-display: "Space Grotesk Variable", "Arial Narrow", sans-serif;
  --font-mono: "JetBrains Mono Variable", monospace;
  --page-gutter: clamp(1.1rem, 3.4vw, 4.8rem);
  --nav-height: 68px;
  --panel-pad-top: clamp(7.5rem, 11vh, 9.5rem);
  --panel-pad-bottom: clamp(4rem, 7vh, 6rem);
  --border-soft: color-mix(in srgb, var(--ultraviolet-glow) 22%, transparent);
  --border-strong: color-mix(in srgb, var(--ultraviolet-glow) 48%, transparent);
  --surface: color-mix(in srgb, var(--deep-violet) 78%, transparent);
  --surface-strong: color-mix(in srgb, var(--indigo-core) 72%, var(--void));
}

* {
  box-sizing: border-box;
}

html {
  background: var(--void);
  color-scheme: dark;
  scroll-behavior: smooth;
  scrollbar-width: thin;
  scrollbar-color: var(--electric-violet) var(--void);
}

body {
  margin: 0;
  min-width: 320px;
  overflow-x: hidden;
  background: var(--void);
  color: var(--ink);
  font-family: var(--font-display);
  font-size: 16px;
  font-weight: 470;
  line-height: 1.55;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
}

::selection {
  background: var(--electric-violet);
  color: var(--ink);
}

button,
a {
  color: inherit;
  font: inherit;
}

a {
  text-decoration: none;
}

button {
  border: 0;
}

button:focus-visible,
a:focus-visible {
  outline: 2px solid var(--ultraviolet-glow);
  outline-offset: 5px;
}

.has-custom-cursor,
.has-custom-cursor * {
  cursor: none !important;
}

.portfolio-shell {
  position: relative;
  min-height: 100vh;
  isolation: isolate;
  background: var(--void);
}

.noise-layer {
  position: fixed;
  inset: 0;
  z-index: 80;
  pointer-events: none;
  opacity: 0.035;
  mix-blend-mode: screen;
  background-image:
    repeating-linear-gradient(
      0deg,
      transparent 0 2px,
      color-mix(in srgb, var(--ultraviolet-glow) 28%, transparent) 3px
    ),
    repeating-linear-gradient(
      90deg,
      transparent 0 5px,
      color-mix(in srgb, var(--electric-violet) 20%, transparent) 6px
    );
  background-size: 9px 9px, 13px 13px;
}

.background-field {
  position: fixed;
  inset: 0;
  z-index: -2;
  overflow: hidden;
  pointer-events: none;
  background:
    radial-gradient(circle at 50% 40%, color-mix(in srgb, var(--deep-violet) 84%, transparent), transparent 56%),
    var(--void);
}

.gradient-mesh {
  position: absolute;
  width: min(64vw, 880px);
  aspect-ratio: 1;
  border-radius: 42% 58% 66% 34% / 52% 34% 66% 48%;
  filter: blur(105px);
  opacity: 0.26;
  will-change: transform;
}

.gradient-mesh--one {
  top: -30%;
  left: -12%;
  background:
    radial-gradient(circle at 35% 36%, var(--plasma-lilac), transparent 38%),
    radial-gradient(circle at 70% 65%, var(--electric-violet), transparent 52%),
    var(--royal-violet);
  animation: mesh-breathe 14s ease-in-out infinite alternate;
}

.gradient-mesh--two {
  right: -16%;
  bottom: -38%;
  background:
    radial-gradient(circle at 35% 65%, var(--neon-violet), transparent 40%),
    radial-gradient(circle at 68% 35%, var(--indigo-core), transparent 55%),
    var(--electric-violet);
  opacity: 0.2;
  animation: mesh-breathe 19s ease-in-out infinite alternate-reverse;
}

.background-field__grid {
  position: absolute;
  inset: -20%;
  opacity: 0.14;
  background-image:
    linear-gradient(color-mix(in srgb, var(--ultraviolet-glow) 19%, transparent) 1px, transparent 1px),
    linear-gradient(90deg, color-mix(in srgb, var(--ultraviolet-glow) 19%, transparent) 1px, transparent 1px);
  background-size: 68px 68px;
  transform: perspective(1200px) rotateX(70deg) translateY(18%);
  transform-origin: center bottom;
  mask-image: linear-gradient(to bottom, transparent 10%, var(--void) 45%, transparent 90%);
}

.background-field__scan {
  position: absolute;
  inset: 0;
  opacity: 0.08;
  background: linear-gradient(
    180deg,
    transparent 0%,
    color-mix(in srgb, var(--ultraviolet-glow) 55%, transparent) 50%,
    transparent 100%
  );
  background-size: 100% 220px;
  animation: field-scan 9s linear infinite;
}

.chapter-progress {
  position: fixed;
  z-index: 150;
  top: 0;
  right: 0;
  left: 0;
  height: 3px;
  transform-origin: left center;
  background: linear-gradient(
    90deg,
    var(--royal-violet),
    var(--electric-violet),
    var(--plasma-lilac),
    var(--ultraviolet-glow)
  );
  box-shadow: 0 0 24px var(--electric-violet);
}

.site-nav-shell {
  position: fixed;
  z-index: 130;
  top: 1rem;
  right: var(--page-gutter);
  left: var(--page-gutter);
}

.site-nav {
  display: grid;
  grid-template-columns: auto minmax(0, 1fr) auto;
  align-items: center;
  min-height: var(--nav-height);
  border: 1px solid var(--border-soft);
  background: color-mix(in srgb, var(--void) 76%, transparent);
  box-shadow:
    0 18px 60px color-mix(in srgb, var(--void) 78%, transparent),
    inset 0 1px color-mix(in srgb, var(--lavender-mist) 8%, transparent);
  backdrop-filter: blur(20px) saturate(130%);
}

.site-mark {
  display: inline-flex;
  align-items: center;
  gap: 0.75rem;
  height: 100%;
  padding: 0 1.05rem;
  background: transparent;
  color: var(--lavender-mist);
  font-family: var(--font-mono);
  font-size: 0.72rem;
  font-weight: 760;
  letter-spacing: 0.12em;
  cursor: pointer;
}

.site-mark em {
  color: var(--plasma-lilac);
  font-style: normal;
}

.site-mark__glyph {
  position: relative;
  display: block;
  width: 24px;
  height: 24px;
}

.site-mark__glyph i {
  position: absolute;
  inset: 4px;
  border: 1px solid var(--ultraviolet-glow);
  border-radius: 50%;
  box-shadow: 0 0 12px color-mix(in srgb, var(--ultraviolet-glow) 60%, transparent);
}

.site-mark__glyph i:nth-child(2) {
  inset: 8px -2px;
  border-radius: 50%;
  transform: rotate(45deg);
}

.site-mark__glyph i:nth-child(3) {
  inset: -2px 8px;
  border-radius: 50%;
  transform: rotate(-45deg);
}

.site-nav__desktop {
  display: flex;
  align-items: stretch;
  justify-content: center;
  height: var(--nav-height);
}

.site-nav__desktop button {
  position: relative;
  display: inline-flex;
  align-items: center;
  gap: 0.55rem;
  padding: 0 0.85rem;
  overflow: hidden;
  background: transparent;
  color: var(--ink-muted);
  font-family: var(--font-mono);
  font-size: 0.65rem;
  font-weight: 680;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  cursor: pointer;
  transition: color 180ms ease, background 180ms ease;
}

.site-nav__desktop button small {
  color: var(--royal-violet);
  font-size: 0.58rem;
  transition: color 180ms ease;
}

.site-nav__desktop button i {
  position: absolute;
  right: 18%;
  bottom: 0;
  left: 18%;
  height: 2px;
  transform: scaleX(0);
  transform-origin: center;
  background: var(--ultraviolet-glow);
  box-shadow: 0 0 16px var(--plasma-lilac);
  transition: transform 240ms ease;
}

.site-nav__desktop button:hover,
.site-nav__desktop button.is-active {
  color: var(--ink);
  background: color-mix(in srgb, var(--royal-violet) 12%, transparent);
}

.site-nav__desktop button:hover small,
.site-nav__desktop button.is-active small {
  color: var(--ultraviolet-glow);
}

.site-nav__desktop button.is-active i {
  transform: scaleX(1);
}

.site-nav__status {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  min-width: 126px;
  height: 100%;
  padding: 0 1rem;
  border-left: 1px solid var(--border-soft);
  font-family: var(--font-mono);
}

.site-nav__status span {
  color: var(--ultraviolet-glow);
  font-size: 0.72rem;
  font-weight: 800;
}

.site-nav__status p {
  margin: 0;
  color: var(--ink-muted);
  font-size: 0.61rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.nav-toggle {
  display: none;
  place-items: center;
  width: 48px;
  height: 48px;
  margin-right: 0.55rem;
  background: color-mix(in srgb, var(--royal-violet) 20%, transparent);
  color: var(--lavender-mist);
  cursor: pointer;
}

.mobile-nav {
  position: fixed;
  z-index: 125;
  inset: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 7rem var(--page-gutter) 3rem;
  background:
    linear-gradient(140deg, color-mix(in srgb, var(--indigo-core) 82%, transparent), var(--void)),
    var(--void);
  transform-origin: top;
}

.mobile-nav__label {
  margin-bottom: 1.5rem;
  color: var(--plasma-lilac);
  font-family: var(--font-mono);
  font-size: 0.65rem;
  letter-spacing: 0.16em;
}

.mobile-nav button {
  display: flex;
  align-items: baseline;
  gap: 1.1rem;
  width: 100%;
  padding: 0.55rem 0;
  border-bottom: 1px solid var(--border-soft);
  background: transparent;
  color: var(--ink-muted);
  font-size: clamp(2rem, 10vw, 4.5rem);
  font-weight: 700;
  letter-spacing: -0.05em;
  text-align: left;
}

.mobile-nav button span {
  color: var(--royal-violet);
  font-family: var(--font-mono);
  font-size: 0.68rem;
  letter-spacing: 0;
}

.mobile-nav button.is-active {
  color: var(--ink);
}

.mobile-nav button.is-active span {
  color: var(--ultraviolet-glow);
}

.chapter-rail {
  position: fixed;
  z-index: 118;
  top: 50%;
  right: 1.35rem;
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
  transform: translateY(-50%);
}

.chapter-rail button {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 0.5rem;
  width: 42px;
  height: 28px;
  background: transparent;
  color: var(--ink-muted);
  font-family: var(--font-mono);
  font-size: 0.54rem;
  cursor: pointer;
}

.chapter-rail button span {
  opacity: 0;
  transform: translateX(5px);
  transition: opacity 180ms ease, transform 180ms ease, color 180ms ease;
}

.chapter-rail button i {
  width: 13px;
  height: 1px;
  background: var(--ink-muted);
  transition: width 220ms ease, background 220ms ease, box-shadow 220ms ease;
}

.chapter-rail button:hover span,
.chapter-rail button.is-active span {
  opacity: 1;
  transform: translateX(0);
  color: var(--ultraviolet-glow);
}

.chapter-rail button.is-active i {
  width: 27px;
  background: var(--ultraviolet-glow);
  box-shadow: 0 0 12px var(--plasma-lilac);
}

.horizontal-scroll-root {
  position: relative;
  z-index: 3;
}

.is-horizontal .horizontal-sticky-viewport {
  position: sticky;
  top: 0;
  width: 100%;
  height: 100svh;
  overflow: hidden;
  perspective: 1600px;
}

.horizontal-track {
  position: relative;
  display: flex;
  width: max-content;
  min-height: 100svh;
  transform-style: preserve-3d;
  will-change: transform;
}

.is-vertical .horizontal-track {
  display: block;
  width: 100%;
}

.chapter-panel {
  position: relative;
  flex: 0 0 100vw;
  width: 100vw;
  min-height: 100svh;
  overflow: hidden;
  border-right: 1px solid var(--border-soft);
  transform-style: preserve-3d;
  backface-visibility: hidden;
}

.is-vertical .chapter-panel {
  width: 100%;
  min-height: auto;
  overflow: clip;
  border-right: 0;
  border-bottom: 1px solid var(--border-soft);
  scroll-margin-top: calc(var(--nav-height) + 1.5rem);
}

.chapter-panel::before {
  content: "";
  position: absolute;
  inset: 0;
  z-index: -1;
  opacity: 0;
  background: radial-gradient(
    circle at 65% 45%,
    color-mix(in srgb, var(--electric-violet) 13%, transparent),
    transparent 58%
  );
  transition: opacity 700ms ease;
}

.chapter-panel.is-active::before {
  opacity: 1;
}

.chapter-panel__index {
  position: absolute;
  right: 3.5vw;
  bottom: -7vh;
  z-index: -1;
  color: color-mix(in srgb, var(--ultraviolet-glow) 5%, transparent);
  font-family: var(--font-mono);
  font-size: clamp(9rem, 22vw, 24rem);
  font-weight: 800;
  letter-spacing: -0.13em;
  line-height: 0.8;
  pointer-events: none;
  user-select: none;
}

.panel-inner {
  width: 100%;
  max-width: 1680px;
  height: 100svh;
  margin: 0 auto;
  padding: var(--panel-pad-top) clamp(4.3rem, 6vw, 7rem) var(--panel-pad-bottom) var(--page-gutter);
}

.is-vertical .panel-inner {
  height: auto;
  min-height: 100svh;
  padding-right: var(--page-gutter);
}

.section-heading {
  position: relative;
  z-index: 3;
  max-width: 980px;
  transform-style: preserve-3d;
}

.section-heading--right {
  margin-left: auto;
  text-align: right;
}

.section-heading__meta {
  display: flex;
  align-items: center;
  gap: 0.85rem;
  margin-bottom: 0.8rem;
  color: var(--plasma-lilac);
  font-family: var(--font-mono);
  font-size: 0.64rem;
  font-weight: 680;
  letter-spacing: 0.13em;
  text-transform: uppercase;
}

.section-heading__meta span {
  color: var(--ultraviolet-glow);
}

.section-heading__meta i {
  width: 42px;
  height: 1px;
  background: var(--royal-violet);
}

.section-heading__meta p {
  margin: 0;
}

.section-heading h2 {
  max-width: 1120px;
  margin: 0;
  color: var(--ink);
  font-size: clamp(2.5rem, 5.1vw, 6.4rem);
  font-weight: 680;
  letter-spacing: -0.065em;
  line-height: 0.92;
  text-wrap: balance;
}

.glitch-reveal {
  position: relative;
}

.glitch-reveal::before,
.glitch-reveal::after {
  content: attr(data-text);
  position: absolute;
  inset: 0;
  pointer-events: none;
  opacity: 0;
}

.glitch-reveal::before {
  color: var(--ultraviolet-glow);
  clip-path: inset(12% 0 63% 0);
}

.glitch-reveal::after {
  color: var(--electric-violet);
  clip-path: inset(62% 0 12% 0);
}

.glitch-reveal.is-active::before {
  animation: glitch-top 580ms steps(2, end) 1;
}

.glitch-reveal.is-active::after {
  animation: glitch-bottom 580ms steps(2, end) 1;
}

.boot-sequence {
  position: fixed;
  inset: 0;
  z-index: 1000;
  display: grid;
  place-items: center;
  overflow: hidden;
  background: var(--void);
}

.boot-sequence::before {
  content: "";
  position: absolute;
  inset: 0;
  opacity: 0.42;
  background-image:
    linear-gradient(color-mix(in srgb, var(--royal-violet) 26%, transparent) 1px, transparent 1px),
    linear-gradient(90deg, color-mix(in srgb, var(--royal-violet) 26%, transparent) 1px, transparent 1px);
  background-size: 46px 46px;
  mask-image: radial-gradient(circle at center, var(--void), transparent 80%);
}

.boot-sequence__scan {
  position: absolute;
  inset: -50% 0;
  background: linear-gradient(
    180deg,
    transparent 42%,
    color-mix(in srgb, var(--ultraviolet-glow) 18%, transparent) 50%,
    transparent 58%
  );
  animation: boot-scan 1.4s linear infinite;
}

.boot-sequence__terminal {
  position: relative;
  width: min(760px, calc(100vw - 2rem));
  min-height: 390px;
  padding: clamp(1.3rem, 4vw, 2.5rem);
  border: 1px solid var(--border-strong);
  background: color-mix(in srgb, var(--deep-violet) 82%, transparent);
  box-shadow:
    18px 18px 0 color-mix(in srgb, var(--indigo-core) 70%, transparent),
    0 0 80px color-mix(in srgb, var(--electric-violet) 18%, transparent);
  backdrop-filter: blur(12px);
}

.boot-sequence__terminal::before,
.boot-sequence__terminal::after {
  content: "";
  position: absolute;
  width: 46px;
  height: 46px;
  border-color: var(--ultraviolet-glow);
}

.boot-sequence__terminal::before {
  top: -8px;
  left: -8px;
  border-top: 2px solid;
  border-left: 2px solid;
}

.boot-sequence__terminal::after {
  right: -8px;
  bottom: -8px;
  border-right: 2px solid;
  border-bottom: 2px solid;
}

.boot-sequence__topline {
  display: flex;
  justify-content: space-between;
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--border-soft);
  color: var(--plasma-lilac);
  font-family: var(--font-mono);
  font-size: 0.62rem;
  letter-spacing: 0.12em;
}

.boot-sequence__output {
  min-height: 210px;
  padding: 2.2rem 0;
  font-family: var(--font-mono);
}

.boot-sequence__output p {
  display: flex;
  align-items: center;
  gap: 0.8rem;
  margin: 0 0 0.7rem;
  color: var(--lavender-mist);
  font-size: clamp(0.72rem, 2vw, 0.88rem);
}

.boot-sequence__output p span {
  color: var(--royal-violet);
}

.boot-sequence__output p i {
  width: 8px;
  height: 1.1em;
  background: var(--ultraviolet-glow);
  animation: cursor-blink 650ms steps(1) infinite;
}

.boot-sequence__skip {
  position: absolute;
  right: 1.2rem;
  bottom: 1.2rem;
  display: inline-flex;
  align-items: center;
  gap: 0.65rem;
  padding: 0.65rem 0.8rem;
  border: 1px solid var(--border-soft);
  background: transparent;
  color: var(--ink-muted);
  font-family: var(--font-mono);
  font-size: 0.58rem;
  letter-spacing: 0.1em;
  cursor: pointer;
}

.boot-sequence__skip span {
  padding: 0.2rem 0.35rem;
  background: var(--indigo-core);
  color: var(--lavender-mist);
}

.cursor-aura {
  position: fixed;
  z-index: 90;
  top: -130px;
  left: -130px;
  width: 260px;
  height: 260px;
  border-radius: 50%;
  pointer-events: none;
  background: radial-gradient(
    circle,
    color-mix(in srgb, var(--plasma-lilac) 15%, transparent),
    transparent 68%
  );
  mix-blend-mode: screen;
}

.cursor-reticle {
  position: fixed;
  z-index: 400;
  top: -18px;
  left: -18px;
  width: 36px;
  height: 36px;
  pointer-events: none;
  transition: width 180ms ease, height 180ms ease, top 180ms ease, left 180ms ease;
}

.cursor-reticle span {
  position: absolute;
  width: 9px;
  height: 9px;
  border-color: var(--ultraviolet-glow);
  transition: transform 180ms ease, border-color 180ms ease;
}

.cursor-reticle span:nth-child(1) {
  top: 0;
  left: 0;
  border-top: 1px solid;
  border-left: 1px solid;
}

.cursor-reticle span:nth-child(2) {
  top: 0;
  right: 0;
  border-top: 1px solid;
  border-right: 1px solid;
}

.cursor-reticle span:nth-child(3) {
  right: 0;
  bottom: 0;
  border-right: 1px solid;
  border-bottom: 1px solid;
}

.cursor-reticle span:nth-child(4) {
  bottom: 0;
  left: 0;
  border-bottom: 1px solid;
  border-left: 1px solid;
}

.cursor-reticle i {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 3px;
  height: 3px;
  border-radius: 50%;
  background: var(--ultraviolet-glow);
  box-shadow: 0 0 8px var(--plasma-lilac);
  transform: translate(-50%, -50%);
}

.cursor-reticle--action span:nth-child(1),
.cursor-reticle--card span:nth-child(1) {
  transform: translate(-5px, -5px);
}

.cursor-reticle--action span:nth-child(2),
.cursor-reticle--card span:nth-child(2) {
  transform: translate(5px, -5px);
}

.cursor-reticle--action span:nth-child(3),
.cursor-reticle--card span:nth-child(3) {
  transform: translate(5px, 5px);
}

.cursor-reticle--action span:nth-child(4),
.cursor-reticle--card span:nth-child(4) {
  transform: translate(-5px, 5px);
}

.cursor-reticle--action span {
  border-color: var(--lavender-mist);
}

.cursor-reticle--card {
  top: -23px;
  left: -23px;
  width: 46px;
  height: 46px;
}

.mode-indicator {
  position: fixed;
  z-index: 110;
  right: 1.3rem;
  bottom: 1.1rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  color: var(--ink-muted);
  font-family: var(--font-mono);
  font-size: 0.55rem;
  letter-spacing: 0.08em;
  pointer-events: none;
}

.mode-indicator svg:first-child {
  color: var(--plasma-lilac);
}

.hero-panel {
  background:
    linear-gradient(90deg, color-mix(in srgb, var(--void) 82%, transparent), transparent 60%),
    transparent;
}

.hero-panel__layout {
  position: relative;
  display: grid;
  grid-template-columns: minmax(0, 1.02fr) minmax(420px, 0.98fr);
  align-items: center;
  width: 100%;
  height: 100svh;
  padding: calc(var(--panel-pad-top) - 0.5rem) clamp(4.3rem, 6vw, 7rem) 5.6rem var(--page-gutter);
}

.hero-copy {
  position: relative;
  z-index: 4;
  max-width: 900px;
  transform-style: preserve-3d;
}

.status-pill {
  display: inline-flex;
  align-items: center;
  gap: 0.65rem;
  padding: 0.58rem 0.72rem;
  border: 1px solid var(--border-soft);
  background: color-mix(in srgb, var(--deep-violet) 60%, transparent);
  color: var(--lavender-mist);
  font-family: var(--font-mono);
  font-size: 0.57rem;
  font-weight: 680;
  letter-spacing: 0.11em;
}

.status-pill span {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: var(--ultraviolet-glow);
  box-shadow: 0 0 14px var(--plasma-lilac);
  animation: status-pulse 1.6s ease-in-out infinite;
}

.hero-copy__meta {
  display: flex;
  justify-content: space-between;
  max-width: 700px;
  margin: 1.5rem 0 0.85rem;
  color: var(--ink-muted);
  font-family: var(--font-mono);
  font-size: 0.58rem;
  letter-spacing: 0.12em;
}

.hero-title {
  max-width: 920px;
  margin: 0;
  color: var(--ink);
  font-size: clamp(4.4rem, 8vw, 9.5rem);
  font-weight: 720;
  letter-spacing: -0.085em;
  line-height: 0.77;
}

.hero-title > span {
  display: block;
}

.hero-title__accent {
  color: var(--ultraviolet-glow);
  text-shadow: 0 0 38px color-mix(in srgb, var(--electric-violet) 38%, transparent);
}

.hero-copy__intro {
  max-width: 670px;
  margin: 1.6rem 0 0;
  color: var(--ink-muted);
  font-size: clamp(0.95rem, 1.25vw, 1.14rem);
  line-height: 1.65;
}

.hero-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.8rem;
  margin-top: 1.8rem;
}

.action-link {
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  min-height: 52px;
  padding: 0.85rem 1.05rem;
  border: 1px solid var(--border-strong);
  font-family: var(--font-mono);
  font-size: 0.62rem;
  font-weight: 720;
  letter-spacing: 0.09em;
  overflow: hidden;
  transition: border-color 180ms ease, color 180ms ease, box-shadow 180ms ease;
}

.action-link::before {
  content: "";
  position: absolute;
  inset: 0;
  z-index: -1;
  transform: scaleX(0);
  transform-origin: right;
  background: var(--electric-violet);
  transition: transform 320ms cubic-bezier(0.16, 1, 0.3, 1);
}

.action-link:hover::before {
  transform: scaleX(1);
  transform-origin: left;
}

.action-link--primary {
  background: var(--royal-violet);
  color: var(--ink);
  box-shadow: 0 12px 34px color-mix(in srgb, var(--royal-violet) 28%, transparent);
}

.action-link--ghost {
  background: color-mix(in srgb, var(--deep-violet) 56%, transparent);
  color: var(--lavender-mist);
}

.hero-copy__footnote {
  display: flex;
  align-items: center;
  gap: 0.7rem;
  margin-top: 1.5rem;
  color: color-mix(in srgb, var(--ink-muted) 68%, transparent);
  font-family: var(--font-mono);
  font-size: 0.52rem;
  letter-spacing: 0.12em;
}

.hero-copy__footnote i {
  width: 22px;
  height: 1px;
  background: var(--royal-violet);
}

.hero-scene-shell {
  position: relative;
  z-index: 2;
  width: min(47vw, 760px);
  aspect-ratio: 1;
  justify-self: center;
  transform-style: preserve-3d;
}

.hero-scene-shell::before,
.hero-scene-shell::after {
  content: "";
  position: absolute;
  pointer-events: none;
}

.hero-scene-shell::before {
  inset: 4%;
  border: 1px solid var(--border-soft);
  border-radius: 50%;
  box-shadow:
    inset 0 0 55px color-mix(in srgb, var(--electric-violet) 12%, transparent),
    0 0 90px color-mix(in srgb, var(--royal-violet) 17%, transparent);
}

.hero-scene-shell::after {
  inset: 12%;
  border: 1px dashed color-mix(in srgb, var(--ultraviolet-glow) 28%, transparent);
  border-radius: 50%;
  animation: shell-rotate 28s linear infinite;
}

.cyber-scene {
  position: absolute;
  inset: 0;
}

.cyber-scene canvas {
  touch-action: pan-y;
}

.cyber-scene__hud {
  position: absolute;
  right: 8%;
  left: 8%;
  display: flex;
  align-items: center;
  gap: 0.65rem;
  color: var(--ink-muted);
  font-family: var(--font-mono);
  font-size: 0.5rem;
  letter-spacing: 0.12em;
  pointer-events: none;
}

.cyber-scene__hud--top {
  top: 8%;
}

.cyber-scene__hud--bottom {
  bottom: 8%;
  justify-content: space-between;
}

.cyber-scene__hud i {
  flex: 1;
  height: 1px;
  background: linear-gradient(90deg, var(--royal-violet), transparent);
}

.hero-scene-shell__caption {
  position: absolute;
  right: 1.2rem;
  bottom: 1.2rem;
  padding: 0.6rem 0.75rem;
  border-left: 1px solid var(--ultraviolet-glow);
  background: color-mix(in srgb, var(--void) 72%, transparent);
  font-family: var(--font-mono);
  backdrop-filter: blur(10px);
}

.hero-scene-shell__caption span {
  display: block;
  color: var(--ultraviolet-glow);
  font-size: 0.58rem;
  letter-spacing: 0.12em;
}

.hero-scene-shell__caption p {
  margin: 0.15rem 0 0;
  color: var(--ink-muted);
  font-size: 0.48rem;
  letter-spacing: 0.08em;
}

.scene-loader {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.55rem;
}

.scene-loader span {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: var(--ultraviolet-glow);
  animation: loader-pulse 900ms ease-in-out infinite alternate;
}

.scene-loader span:nth-child(2) {
  animation-delay: 180ms;
}

.scene-loader span:nth-child(3) {
  animation-delay: 360ms;
}

.signal-strip {
  position: absolute;
  right: 0;
  bottom: 0;
  left: 0;
  overflow: hidden;
  border-top: 1px solid var(--border-soft);
  border-bottom: 1px solid var(--border-soft);
  background: color-mix(in srgb, var(--deep-violet) 66%, transparent);
  white-space: nowrap;
}

.signal-strip__track {
  display: flex;
  width: max-content;
  padding: 0.72rem 0;
}

.signal-strip__track span {
  display: inline-flex;
  align-items: center;
  gap: 0.7rem;
  padding: 0 1.7rem;
  color: var(--ink-muted);
  font-family: var(--font-mono);
  font-size: 0.54rem;
  letter-spacing: 0.13em;
}

.signal-strip__track i {
  width: 5px;
  height: 5px;
  border-radius: 50%;
  background: var(--plasma-lilac);
  box-shadow: 0 0 10px var(--plasma-lilac);
}

.about-panel {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.about-panel__grid {
  display: grid;
  grid-template-columns: minmax(0, 1.1fr) minmax(300px, 0.82fr) minmax(280px, 0.75fr);
  gap: clamp(1rem, 2vw, 2rem);
  align-items: stretch;
  margin-top: clamp(2rem, 4.5vh, 4rem);
  perspective: 1300px;
}

.about-manifesto,
.education-file,
.metric-cell {
  border: 1px solid var(--border-soft);
  background: var(--surface);
  box-shadow: inset 0 1px color-mix(in srgb, var(--lavender-mist) 6%, transparent);
  backdrop-filter: blur(16px);
}

.about-manifesto {
  padding: clamp(1.4rem, 2.4vw, 2.4rem);
  transform-origin: left center;
}

.about-manifesto__lead {
  margin: 0;
  color: var(--lavender-mist);
  font-size: clamp(1.25rem, 2vw, 2rem);
  font-weight: 560;
  letter-spacing: -0.035em;
  line-height: 1.25;
}

.about-manifesto > p:not(.about-manifesto__lead) {
  margin: 1.35rem 0 0;
  color: var(--ink-muted);
}

.about-manifesto__tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.45rem;
  margin-top: 1.6rem;
}

.about-manifesto__tags span,
.skill-group__tags span,
.project-case__stack span {
  padding: 0.36rem 0.48rem;
  border: 1px solid var(--border-soft);
  background: color-mix(in srgb, var(--royal-violet) 11%, transparent);
  color: var(--lavender-mist);
  font-family: var(--font-mono);
  font-size: 0.48rem;
  letter-spacing: 0.07em;
}

.metric-matrix {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 0.75rem;
  perspective: 1200px;
}

.metric-cell {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  min-height: 145px;
  padding: 1.15rem;
  overflow: hidden;
  transform-style: preserve-3d;
}

.metric-cell > span {
  color: var(--ultraviolet-glow);
  font-family: var(--font-mono);
  font-size: clamp(1.55rem, 2.3vw, 2.7rem);
  font-weight: 760;
  letter-spacing: -0.06em;
}

.metric-cell p {
  margin: 0.25rem 0 0;
  color: var(--ink-muted);
  font-family: var(--font-mono);
  font-size: 0.52rem;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.metric-cell i {
  position: absolute;
  top: -32px;
  right: -24px;
  width: 90px;
  height: 90px;
  border: 1px solid var(--royal-violet);
  border-radius: 50%;
  opacity: 0.45;
}

.education-file {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: clamp(1.4rem, 2vw, 2rem);
  overflow: hidden;
}

.education-file::before {
  content: "";
  position: absolute;
  top: -36%;
  right: -25%;
  width: 230px;
  height: 230px;
  border-radius: 50%;
  background: var(--royal-violet);
  filter: blur(80px);
  opacity: 0.26;
}

.education-file__topline,
.project-case__topline {
  display: flex;
  justify-content: space-between;
  color: var(--plasma-lilac);
  font-family: var(--font-mono);
  font-size: 0.53rem;
  letter-spacing: 0.09em;
}

.education-file h3 {
  margin: 2.2rem 0 0.2rem;
  color: var(--ink);
  font-size: clamp(1.6rem, 2.5vw, 2.8rem);
  letter-spacing: -0.055em;
  line-height: 1;
}

.education-file > p {
  margin: 0;
  color: var(--ink-muted);
}

.education-file__details {
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
  margin-top: 1.5rem;
  color: var(--ultraviolet-glow);
  font-family: var(--font-mono);
  font-size: 0.58rem;
  letter-spacing: 0.08em;
}

.education-file small {
  margin-top: 1.2rem;
  color: var(--ink-muted);
  font-family: var(--font-mono);
  font-size: 0.52rem;
  line-height: 1.7;
}

.projects-panel {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.projects-stage {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: clamp(0.8rem, 1.3vw, 1.25rem);
  align-items: start;
  margin-top: clamp(1.7rem, 3.2vh, 3rem);
  perspective: 1600px;
  transform-style: preserve-3d;
}

.project-case {
  position: relative;
  min-height: 400px;
  border: 1px solid var(--border-soft);
  background: color-mix(in srgb, var(--deep-violet) 80%, transparent);
  box-shadow:
    inset 0 1px color-mix(in srgb, var(--lavender-mist) 7%, transparent),
    0 20px 70px color-mix(in srgb, var(--void) 45%, transparent);
  overflow: hidden;
  transform-style: preserve-3d;
  backdrop-filter: blur(18px);
  transition: border-color 220ms ease, box-shadow 220ms ease;
}

.project-case::before {
  content: "";
  position: absolute;
  inset: -40% 15% 65% -15%;
  border-radius: 50%;
  background: var(--royal-violet);
  filter: blur(58px);
  opacity: 0.22;
  transition: opacity 220ms ease, transform 500ms ease;
}

.project-case:hover,
.project-case.is-expanded {
  border-color: var(--border-strong);
  box-shadow:
    inset 0 1px color-mix(in srgb, var(--lavender-mist) 10%, transparent),
    0 24px 90px color-mix(in srgb, var(--royal-violet) 22%, transparent);
}

.project-case:hover::before,
.project-case.is-expanded::before {
  opacity: 0.38;
  transform: translate(16%, 12%);
}

.project-case__trigger {
  position: relative;
  z-index: 2;
  width: 100%;
  min-height: 400px;
  padding: clamp(1.15rem, 1.7vw, 1.65rem);
  background: transparent;
  color: inherit;
  text-align: left;
  cursor: pointer;
}

.project-case__icon {
  display: grid;
  place-items: center;
  width: 48px;
  height: 48px;
  margin-top: 1.4rem;
  border: 1px solid var(--border-strong);
  background: color-mix(in srgb, var(--royal-violet) 18%, transparent);
  color: var(--ultraviolet-glow);
  box-shadow: 0 0 24px color-mix(in srgb, var(--electric-violet) 14%, transparent);
}

.project-case__type {
  margin: 1.25rem 0 0.45rem;
  color: var(--plasma-lilac);
  font-family: var(--font-mono);
  font-size: 0.52rem;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.project-case h3 {
  margin: 0;
  color: var(--ink);
  font-size: clamp(1.25rem, 1.75vw, 2rem);
  letter-spacing: -0.045em;
  line-height: 1.05;
}

.project-case__summary {
  margin: 1rem 0 0;
  color: var(--ink-muted);
  font-size: 0.82rem;
  line-height: 1.55;
}

.project-case__toggle {
  position: absolute;
  right: 1.3rem;
  bottom: 1.25rem;
  left: 1.3rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-top: 0.75rem;
  border-top: 1px solid var(--border-soft);
  color: var(--lavender-mist);
  font-family: var(--font-mono);
  font-size: 0.52rem;
  letter-spacing: 0.08em;
}

.project-case__toggle svg {
  transition: transform 260ms ease;
}

.project-case.is-expanded .project-case__toggle svg {
  transform: rotate(180deg);
}

.project-case__details {
  position: relative;
  z-index: 2;
  display: grid;
  grid-template-columns: 1fr;
  gap: 1.1rem;
  overflow: hidden;
  border-top: 1px solid var(--border-soft);
  background: color-mix(in srgb, var(--indigo-core) 42%, transparent);
}

.project-case__details > div {
  padding: 1rem 1.3rem 0;
}

.project-case__details > div:last-child {
  padding-bottom: 1.3rem;
}

.project-case__details p {
  margin: 0 0 0.55rem;
  color: var(--plasma-lilac);
  font-family: var(--font-mono);
  font-size: 0.5rem;
  letter-spacing: 0.1em;
}

.project-case__details ul {
  display: grid;
  gap: 0.4rem;
  margin: 0;
  padding: 0;
  list-style: none;
  color: var(--ink-muted);
  font-size: 0.75rem;
}

.project-case__details li::before {
  content: "//";
  margin-right: 0.45rem;
  color: var(--ultraviolet-glow);
  font-family: var(--font-mono);
}

.project-case__stack {
  display: flex;
  flex-wrap: wrap;
  gap: 0.35rem;
}

.projects-panel__note {
  display: flex;
  align-items: center;
  gap: 0.55rem;
  margin-top: 1.3rem;
  color: var(--ink-muted);
  font-family: var(--font-mono);
  font-size: 0.52rem;
  letter-spacing: 0.08em;
}

.projects-panel__note svg {
  color: var(--plasma-lilac);
}

.arsenal-panel {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.arsenal-panel__layout {
  display: grid;
  grid-template-columns: minmax(430px, 0.88fr) minmax(0, 1.12fr);
  gap: clamp(1.5rem, 3vw, 4rem);
  align-items: center;
  margin-top: clamp(1.8rem, 3vh, 3rem);
}

.tool-orbit-shell {
  position: relative;
  width: min(42vw, 570px);
  aspect-ratio: 1;
  justify-self: center;
  overflow: hidden;
  border: 1px solid var(--border-soft);
  border-radius: 50%;
  background: radial-gradient(
    circle,
    color-mix(in srgb, var(--indigo-core) 50%, transparent) 0 28%,
    transparent 62%
  );
  box-shadow:
    inset 0 0 90px color-mix(in srgb, var(--electric-violet) 14%, transparent),
    0 0 80px color-mix(in srgb, var(--royal-violet) 12%, transparent);
}

.tool-orbit-shell__grid {
  position: absolute;
  inset: 0;
  border-radius: 50%;
  opacity: 0.2;
  background-image:
    linear-gradient(color-mix(in srgb, var(--ultraviolet-glow) 28%, transparent) 1px, transparent 1px),
    linear-gradient(90deg, color-mix(in srgb, var(--ultraviolet-glow) 28%, transparent) 1px, transparent 1px);
  background-size: 40px 40px;
  mask-image: radial-gradient(circle, var(--void), transparent 78%);
}

.tool-orbit {
  position: absolute;
  inset: 10%;
  border-radius: 50%;
  transform-style: preserve-3d;
}

.tool-orbit__ring {
  position: absolute;
  inset: 0;
  border: 1px dashed var(--border-strong);
  border-radius: 50%;
  box-shadow: 0 0 35px color-mix(in srgb, var(--electric-violet) 14%, transparent);
}

.tool-orbit__ring::before,
.tool-orbit__ring::after {
  content: "";
  position: absolute;
  inset: 16%;
  border: 1px solid var(--border-soft);
  border-radius: 50%;
}

.tool-orbit__ring::after {
  inset: 34%;
  border-color: var(--royal-violet);
}

.tool-node-position {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 1px;
  height: 1px;
  transform: rotate(var(--node-angle)) translateX(clamp(142px, 14.4vw, 206px));
  transform-origin: center;
}

.tool-node {
  position: absolute;
  top: -28px;
  left: -34px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.22rem;
  width: 68px;
  height: 56px;
  border: 1px solid var(--border-soft);
  background: color-mix(in srgb, var(--deep-violet) 90%, transparent);
  color: var(--ultraviolet-glow);
  box-shadow: 0 8px 22px color-mix(in srgb, var(--void) 60%, transparent);
  backdrop-filter: blur(10px);
  transition: border-color 180ms ease, background 180ms ease, transform 180ms ease;
}

.tool-node:hover {
  border-color: var(--ultraviolet-glow);
  background: var(--indigo-core);
  transform: scale(1.08);
}

.tool-node span {
  color: var(--lavender-mist);
  font-family: var(--font-mono);
  font-size: 0.43rem;
  letter-spacing: 0.06em;
}

.tool-orbit__core {
  position: absolute;
  top: 50%;
  left: 50%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 136px;
  height: 136px;
  border: 1px solid var(--border-strong);
  border-radius: 50%;
  background: radial-gradient(circle at 35% 30%, var(--electric-violet), var(--indigo-core));
  color: var(--lavender-mist);
  box-shadow:
    0 0 38px color-mix(in srgb, var(--electric-violet) 52%, transparent),
    inset 0 0 24px color-mix(in srgb, var(--ultraviolet-glow) 20%, transparent);
  transform: translate(-50%, -50%);
}

.tool-orbit__core span {
  margin-top: 0.35rem;
  font-family: var(--font-mono);
  font-size: 0.54rem;
  font-weight: 760;
  letter-spacing: 0.1em;
}

.tool-orbit__core small {
  margin-top: 0.12rem;
  color: var(--ink-muted);
  font-family: var(--font-mono);
  font-size: 0.4rem;
  letter-spacing: 0.07em;
}

.skill-groups {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 0.8rem;
  perspective: 1200px;
}

.skill-group {
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 0.85rem;
  min-height: 180px;
  padding: 1rem;
  border: 1px solid var(--border-soft);
  background: var(--surface);
  transform-style: preserve-3d;
  backdrop-filter: blur(14px);
  transition: border-color 220ms ease, background 220ms ease;
}

.skill-group:hover {
  border-color: var(--border-strong);
  background: color-mix(in srgb, var(--indigo-core) 50%, transparent);
}

.skill-group__icon {
  display: grid;
  place-items: center;
  width: 42px;
  height: 42px;
  border: 1px solid var(--border-soft);
  background: color-mix(in srgb, var(--royal-violet) 16%, transparent);
  color: var(--ultraviolet-glow);
}

.skill-group h3 {
  margin: 0;
  color: var(--ink);
  font-size: 1rem;
  letter-spacing: -0.025em;
}

.skill-group p {
  margin: 0.35rem 0 0.75rem;
  color: var(--ink-muted);
  font-size: 0.72rem;
  line-height: 1.5;
}

.skill-group__tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.3rem;
}

.timeline-panel {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.timeline-track {
  position: relative;
  display: grid;
  grid-template-columns: repeat(5, minmax(0, 1fr));
  gap: 0.7rem;
  margin-top: clamp(2.5rem, 5vh, 5rem);
  padding-top: 2rem;
  perspective: 1400px;
}

.timeline-track__line {
  position: absolute;
  top: 0;
  right: 0;
  left: 0;
  height: 1px;
  background: linear-gradient(
    90deg,
    transparent,
    var(--royal-violet) 8%,
    var(--ultraviolet-glow) 52%,
    var(--royal-violet) 92%,
    transparent
  );
  box-shadow: 0 0 16px color-mix(in srgb, var(--electric-violet) 40%, transparent);
}

.timeline-entry {
  position: relative;
  min-height: 300px;
  padding: 1.2rem 1rem;
  border: 1px solid var(--border-soft);
  background: var(--surface);
  transform-style: preserve-3d;
  backdrop-filter: blur(15px);
  transition: border-color 220ms ease, transform 220ms ease;
}

.timeline-entry:hover {
  border-color: var(--border-strong);
  transform: translateY(-6px);
}

.timeline-entry__node {
  position: absolute;
  top: -3.05rem;
  left: 1rem;
  display: grid;
  place-items: center;
  width: 36px;
  height: 36px;
  border: 1px solid var(--ultraviolet-glow);
  border-radius: 50%;
  background: var(--void);
  color: var(--ultraviolet-glow);
  box-shadow: 0 0 20px color-mix(in srgb, var(--electric-violet) 38%, transparent);
}

.timeline-entry > span {
  color: var(--plasma-lilac);
  font-family: var(--font-mono);
  font-size: 0.52rem;
  letter-spacing: 0.08em;
}

.timeline-entry h3 {
  margin: 1.1rem 0 0;
  color: var(--ink);
  font-size: clamp(1rem, 1.3vw, 1.4rem);
  letter-spacing: -0.035em;
  line-height: 1.08;
}

.timeline-entry__subtitle {
  margin: 0.55rem 0 0 !important;
  color: var(--ultraviolet-glow) !important;
  font-family: var(--font-mono);
  font-size: 0.56rem !important;
  letter-spacing: 0.05em;
}

.timeline-entry > p:last-child {
  margin: 1rem 0 0;
  color: var(--ink-muted);
  font-size: 0.72rem;
  line-height: 1.52;
}

.chapter-panel--contact {
  border-right: 0;
  background:
    radial-gradient(circle at 80% 20%, color-mix(in srgb, var(--plasma-lilac) 18%, transparent), transparent 35%),
    linear-gradient(140deg, var(--indigo-core), var(--void) 66%);
}

.contact-panel {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  width: 100%;
  height: 100svh;
  padding: calc(var(--panel-pad-top) - 0.5rem) clamp(4.3rem, 6vw, 7rem) 1.5rem var(--page-gutter);
  overflow: hidden;
}

.contact-panel__mesh {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

.contact-panel__mesh span {
  position: absolute;
  border: 1px solid var(--border-soft);
  border-radius: 50%;
}

.contact-panel__mesh span:nth-child(1) {
  top: -34vw;
  right: -18vw;
  width: 72vw;
  height: 72vw;
}

.contact-panel__mesh span:nth-child(2) {
  top: -20vw;
  right: -4vw;
  width: 46vw;
  height: 46vw;
  border-style: dashed;
  animation: shell-rotate 34s linear infinite reverse;
}

.contact-panel__mesh span:nth-child(3) {
  right: 8vw;
  bottom: -15vw;
  width: 30vw;
  height: 30vw;
  background: color-mix(in srgb, var(--electric-violet) 12%, transparent);
  filter: blur(1px);
}

.contact-panel__topline,
.contact-panel__footer {
  position: relative;
  z-index: 2;
  display: flex;
  align-items: center;
  justify-content: space-between;
  color: var(--ink-muted);
  font-family: var(--font-mono);
  font-size: 0.55rem;
  letter-spacing: 0.1em;
}

.contact-panel__topline span:last-child {
  color: var(--ultraviolet-glow);
}

.contact-panel__main {
  position: relative;
  z-index: 2;
  max-width: 1180px;
  transform-style: preserve-3d;
}

.contact-panel__main > p:first-child {
  margin: 0 0 0.8rem;
  color: var(--plasma-lilac);
  font-family: var(--font-mono);
  font-size: 0.62rem;
  letter-spacing: 0.14em;
}

.contact-panel h2 {
  margin: 0;
  color: var(--ink);
  font-size: clamp(4rem, 8.3vw, 10rem);
  font-weight: 720;
  letter-spacing: -0.085em;
  line-height: 0.78;
}

.contact-panel h2 span {
  display: block;
  color: var(--ultraviolet-glow);
}

.contact-panel__intro {
  max-width: 650px;
  margin: 1.5rem 0 0;
  color: var(--ink-muted);
  font-size: clamp(0.95rem, 1.3vw, 1.16rem);
}

.contact-actions {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 0.7rem;
  margin-top: 2rem;
}

.contact-action {
  display: flex;
  align-items: center;
  justify-content: space-between;
  min-height: 86px;
  padding: 1rem 1.15rem;
  border: 1px solid var(--border-strong);
  background: color-mix(in srgb, var(--deep-violet) 65%, transparent);
  color: var(--lavender-mist);
  font-family: var(--font-mono);
  font-size: 0.58rem;
  letter-spacing: 0.08em;
  backdrop-filter: blur(14px);
  transition: background 220ms ease, border-color 220ms ease, box-shadow 220ms ease;
}

.contact-action span {
  display: inline-flex;
  align-items: center;
  gap: 0.65rem;
}

.contact-action:hover {
  border-color: var(--ultraviolet-glow);
  background: var(--royal-violet);
  box-shadow: 0 16px 42px color-mix(in srgb, var(--royal-violet) 28%, transparent);
}

.contact-action--primary {
  background: var(--electric-violet);
  color: var(--ink);
}

.contact-panel__footer {
  padding-top: 1rem;
  border-top: 1px solid var(--border-soft);
}

.contact-panel__footer span {
  display: inline-flex;
  align-items: center;
  gap: 0.45rem;
}

.not-found {
  display: grid;
  place-items: center;
  align-content: center;
  min-height: 100svh;
  padding: 2rem;
  background:
    radial-gradient(circle at 50% 42%, var(--indigo-core), transparent 42%),
    var(--void);
  text-align: center;
}

.not-found svg {
  color: var(--ultraviolet-glow);
}

.not-found p {
  color: var(--plasma-lilac);
  font-family: var(--font-mono);
  font-size: 0.65rem;
  letter-spacing: 0.12em;
}

.not-found h1 {
  margin: 0;
  color: var(--ink);
  font-size: clamp(7rem, 22vw, 18rem);
  line-height: 0.8;
  letter-spacing: -0.1em;
}

.not-found h2 {
  max-width: 630px;
  color: var(--ink-muted);
  font-size: clamp(1rem, 2vw, 1.5rem);
  font-weight: 460;
}

.not-found a {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  margin-top: 1.2rem;
  padding: 0.8rem 1rem;
  border: 1px solid var(--border-strong);
  background: var(--royal-violet);
  color: var(--ink);
  font-family: var(--font-mono);
  font-size: 0.6rem;
  letter-spacing: 0.08em;
}

@keyframes mesh-breathe {
  0% {
    border-radius: 42% 58% 66% 34% / 52% 34% 66% 48%;
    transform: scale(0.92) rotate(-6deg);
  }
  100% {
    border-radius: 62% 38% 40% 60% / 35% 62% 38% 65%;
    transform: scale(1.08) rotate(8deg);
  }
}

@keyframes field-scan {
  from {
    background-position: 0 -220px;
  }
  to {
    background-position: 0 calc(100vh + 220px);
  }
}

@keyframes boot-scan {
  from {
    transform: translateY(-24%);
  }
  to {
    transform: translateY(24%);
  }
}

@keyframes cursor-blink {
  0%,
  49% {
    opacity: 1;
  }
  50%,
  100% {
    opacity: 0;
  }
}

@keyframes status-pulse {
  0%,
  100% {
    transform: scale(0.8);
    opacity: 0.65;
  }
  50% {
    transform: scale(1.25);
    opacity: 1;
  }
}

@keyframes shell-rotate {
  to {
    transform: rotate(360deg);
  }
}

@keyframes loader-pulse {
  from {
    transform: translateY(0) scale(0.8);
    opacity: 0.5;
  }
  to {
    transform: translateY(-8px) scale(1.1);
    opacity: 1;
  }
}

@keyframes glitch-top {
  0% {
    opacity: 0;
    transform: translate(0);
  }
  12% {
    opacity: 0.9;
    transform: translate(7px, -2px);
  }
  24% {
    transform: translate(-5px, 1px);
  }
  42% {
    transform: translate(3px, 0);
  }
  62% {
    opacity: 0.55;
    transform: translate(-2px, -1px);
  }
  100% {
    opacity: 0;
    transform: translate(0);
  }
}

@keyframes glitch-bottom {
  0% {
    opacity: 0;
    transform: translate(0);
  }
  14% {
    opacity: 0.8;
    transform: translate(-6px, 2px);
  }
  30% {
    transform: translate(4px, -1px);
  }
  48% {
    transform: translate(-3px, 1px);
  }
  70% {
    opacity: 0.42;
    transform: translate(2px, 0);
  }
  100% {
    opacity: 0;
    transform: translate(0);
  }
}

@media (max-width: 1380px) {
  .site-nav__desktop button {
    padding: 0 0.6rem;
    font-size: 0.59rem;
  }

  .hero-title {
    font-size: clamp(4rem, 7.5vw, 8rem);
  }

  .panel-inner,
  .hero-panel__layout,
  .contact-panel {
    padding-right: 4.7rem;
  }

  .about-panel__grid {
    grid-template-columns: 1.05fr 0.85fr;
  }

  .education-file {
    grid-column: 1 / -1;
    min-height: 190px;
  }

  .education-file h3 {
    margin-top: 1.3rem;
  }

  .timeline-entry {
    min-height: 330px;
  }
}

@media (max-width: 1120px) {
  :root {
    --panel-pad-top: 8.3rem;
  }

  .site-nav {
    grid-template-columns: 1fr auto;
  }

  .site-nav__desktop,
  .site-nav__status,
  .chapter-rail {
    display: none;
  }

  .nav-toggle {
    display: grid;
  }

  .hero-panel__layout {
    grid-template-columns: 1fr;
    min-height: 100svh;
    height: auto;
    padding-right: var(--page-gutter);
  }

  .hero-copy {
    max-width: 880px;
  }

  .hero-scene-shell {
    position: absolute;
    top: 15%;
    right: -8%;
    width: min(62vw, 650px);
    opacity: 0.56;
  }

  .hero-copy__intro {
    max-width: 620px;
  }

  .panel-inner,
  .contact-panel {
    padding-right: var(--page-gutter);
  }

  .chapter-panel__index {
    opacity: 0.7;
  }

  .about-panel__grid {
    grid-template-columns: 1fr 0.82fr;
  }

  .projects-stage {
    grid-template-columns: 1fr;
  }

  .project-case,
  .project-case__trigger {
    min-height: 0;
  }

  .project-case__trigger {
    padding-bottom: 4.4rem;
  }

  .project-case__details {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .arsenal-panel__layout {
    grid-template-columns: 1fr;
  }

  .tool-orbit-shell {
    width: min(88vw, 560px);
  }

  .tool-node-position {
    transform: rotate(var(--node-angle)) translateX(clamp(140px, 35vw, 202px));
  }

  .skill-groups {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .timeline-track {
    grid-template-columns: repeat(2, minmax(0, 1fr));
    padding-top: 0;
  }

  .timeline-track__line {
    display: none;
  }

  .timeline-entry__node {
    top: 1rem;
    left: auto;
    right: 1rem;
  }

  .timeline-entry {
    min-height: 260px;
  }

  .contact-actions {
    grid-template-columns: 1fr;
  }

  .contact-panel h2 {
    font-size: clamp(4rem, 10.5vw, 8rem);
  }

  .contact-panel__footer {
    flex-wrap: wrap;
    gap: 0.75rem;
  }
}

@media (max-width: 760px) {
  :root {
    --page-gutter: 1rem;
    --nav-height: 60px;
    --panel-pad-top: 7rem;
    --panel-pad-bottom: 4rem;
  }

  body {
    font-size: 15px;
  }

  .site-nav-shell {
    top: 0.65rem;
  }

  .site-nav {
    min-height: 60px;
  }

  .site-mark {
    padding-left: 0.75rem;
  }

  .site-mark__glyph {
    width: 20px;
    height: 20px;
  }

  .mode-indicator {
    display: none;
  }

  .boot-sequence__terminal {
    min-height: 340px;
    box-shadow: 9px 9px 0 color-mix(in srgb, var(--indigo-core) 70%, transparent);
  }

  .boot-sequence__topline span:last-child {
    display: none;
  }

  .hero-panel__layout {
    min-height: 100svh;
    padding-top: 7.2rem;
    padding-bottom: 4.3rem;
  }

  .hero-scene-shell {
    top: 20%;
    right: -34%;
    width: 115vw;
    opacity: 0.32;
  }

  .hero-copy__meta {
    flex-direction: column;
    gap: 0.25rem;
  }

  .hero-title {
    max-width: 96vw;
    font-size: clamp(3.5rem, 17vw, 6rem);
    line-height: 0.8;
  }

  .hero-copy__intro {
    max-width: 92%;
  }

  .hero-actions {
    align-items: stretch;
    flex-direction: column;
    max-width: 360px;
  }

  .action-link {
    width: 100%;
  }

  .hero-copy__footnote {
    display: none;
  }

  .signal-strip__track span {
    padding: 0 1rem;
  }

  .panel-inner {
    min-height: auto;
    padding-bottom: 5rem;
  }

  .section-heading h2 {
    font-size: clamp(2.7rem, 13vw, 4.8rem);
  }

  .section-heading__meta {
    font-size: 0.55rem;
  }

  .about-panel__grid {
    grid-template-columns: 1fr;
  }

  .metric-matrix {
    grid-template-columns: repeat(2, 1fr);
  }

  .metric-cell {
    min-height: 128px;
  }

  .education-file {
    grid-column: auto;
  }

  .project-case__details {
    grid-template-columns: 1fr;
  }

  .projects-panel__note {
    align-items: flex-start;
  }

  .tool-orbit-shell {
    width: calc(100vw - 2rem);
  }

  .tool-node-position {
    transform: rotate(var(--node-angle)) translateX(clamp(125px, 39vw, 170px));
  }

  .tool-node {
    width: 58px;
    height: 50px;
    left: -29px;
    top: -25px;
  }

  .tool-node span {
    font-size: 0.37rem;
  }

  .tool-orbit__core {
    width: 108px;
    height: 108px;
  }

  .skill-groups {
    grid-template-columns: 1fr;
  }

  .skill-group {
    min-height: 0;
  }

  .timeline-track {
    grid-template-columns: 1fr;
  }

  .timeline-entry {
    min-height: 0;
  }

  .contact-panel {
    min-height: 100svh;
    height: auto;
    padding-top: 7.5rem;
    padding-bottom: 1.2rem;
  }

  .contact-panel__topline {
    align-items: flex-start;
    flex-direction: column;
    gap: 0.25rem;
  }

  .contact-panel h2 {
    font-size: clamp(3.8rem, 17vw, 6.3rem);
    line-height: 0.8;
  }

  .contact-action {
    min-height: 72px;
  }

  .contact-panel__footer {
    align-items: flex-start;
    flex-direction: column;
    margin-top: 4rem;
  }

  .contact-panel__footer span:last-child {
    display: none;
  }

  .chapter-panel__index {
    right: 4vw;
    bottom: 0;
    font-size: 42vw;
  }
}

@media (max-width: 430px) {
  .status-pill {
    max-width: 100%;
    font-size: 0.5rem;
  }

  .hero-title {
    font-size: 16vw;
  }

  .metric-matrix {
    grid-template-columns: 1fr;
  }

  .tool-node-position {
    transform: rotate(var(--node-angle)) translateX(34vw);
  }

  .tool-node {
    width: 52px;
    height: 46px;
    left: -26px;
    top: -23px;
  }

  .tool-orbit__core {
    width: 92px;
    height: 92px;
  }

  .tool-orbit__core svg {
    width: 27px;
    height: 27px;
  }

  .tool-orbit__core span {
    font-size: 0.46rem;
  }

  .tool-orbit__core small {
    display: none;
  }
}

@media (prefers-reduced-motion: reduce) {
  html {
    scroll-behavior: auto;
  }

  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    scroll-behavior: auto !important;
    transition-duration: 0.01ms !important;
  }

  .gradient-mesh,
  .background-field__scan,
  .signal-strip__track,
  .hero-scene-shell::after,
  .contact-panel__mesh span:nth-child(2) {
    animation: none !important;
  }
}
```

## `app/icon.svg`

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64">
  <rect width="64" height="64" rx="14" fill="#0b0615"/>
  <circle cx="32" cy="32" r="21" fill="none" stroke="#5b21b6" stroke-width="2"/>
  <ellipse cx="32" cy="32" rx="25" ry="10" fill="none" stroke="#9333ea" stroke-width="2" transform="rotate(42 32 32)"/>
  <ellipse cx="32" cy="32" rx="25" ry="10" fill="none" stroke="#c084fc" stroke-width="2" transform="rotate(-42 32 32)"/>
  <circle cx="32" cy="32" r="8" fill="#7c3aed" stroke="#e9d5ff" stroke-width="2"/>
</svg>
```

## `app/layout.tsx`

```tsx
import type { Metadata, Viewport } from "next";
import "@fontsource-variable/space-grotesk";
import "@fontsource-variable/jetbrains-mono";
import "./globals.css";

const siteUrl = process.env.NEXT_PUBLIC_SITE_URL ?? "http://localhost:3000";

export const metadata: Metadata = {
  metadataBase: new URL(siteUrl),
  title: "Muhammad Iqbal Latif | Offensive Security Portfolio",
  description:
    "Horizontal cyber-security portfolio covering offensive security, explainable npm supply-chain research, digital forensics, cryptography, and CTF engineering.",
  keywords: [
    "Muhammad Iqbal Latif",
    "Cyber Security",
    "Offensive Security",
    "Penetration Testing",
    "Supply Chain Security",
    "Digital Forensics",
    "CTF",
  ],
  authors: [{ name: "Muhammad Iqbal Latif" }],
  creator: "Muhammad Iqbal Latif",
  openGraph: {
    title: "Muhammad Iqbal Latif | Offensive Security Portfolio",
    description:
      "Offensive security, supply-chain research, digital forensics, cryptography, and CTF engineering.",
    type: "website",
    locale: "en_US",
  },
  twitter: {
    card: "summary_large_image",
    title: "Muhammad Iqbal Latif | Offensive Security Portfolio",
    description: "Break systems. Build resilience.",
  },
};

export const viewport: Viewport = {
  themeColor: "#0b0615",
  colorScheme: "dark",
};

export default function RootLayout({ children }: Readonly<{ children: React.ReactNode }>) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

## `app/manifest.ts`

```ts
import type { MetadataRoute } from "next";

export default function manifest(): MetadataRoute.Manifest {
  return {
    name: "Muhammad Iqbal Latif — Offensive Security Portfolio",
    short_name: "IQBAL//SEC",
    description:
      "Horizontal cyber-security portfolio for offensive security, supply-chain research, digital forensics, cryptography, and CTF engineering.",
    start_url: "/",
    display: "standalone",
    background_color: "#0b0615",
    theme_color: "#0b0615",
    icons: [{ src: "/icon.svg", sizes: "any", type: "image/svg+xml" }],
  };
}
```

## `app/not-found.tsx`

```tsx
import Link from "next/link";
import { ArrowLeft, ShieldAlert } from "lucide-react";

export default function NotFound() {
  return (
    <main className="not-found">
      <ShieldAlert size={62} />
      <p>ERROR // ROUTE_NOT_FOUND</p>
      <h1>404</h1>
      <h2>This node does not exist in the current attack surface.</h2>
      <Link href="/">
        <ArrowLeft size={18} /> RETURN TO ORIGIN
      </Link>
    </main>
  );
}
```

## `app/opengraph-image.tsx`

```tsx
import { ImageResponse } from "next/og";

export const alt = "Muhammad Iqbal Latif — Offensive Security Portfolio";
export const size = { width: 1200, height: 630 };
export const contentType = "image/png";

export default function OpenGraphImage() {
  return new ImageResponse(
    (
      <div
        style={{
          width: "100%",
          height: "100%",
          display: "flex",
          position: "relative",
          overflow: "hidden",
          background: "#0b0615",
          color: "#f5f3ff",
          padding: "58px",
          fontFamily: "sans-serif",
        }}
      >
        <div
          style={{
            position: "absolute",
            width: "660px",
            height: "660px",
            borderRadius: "50%",
            border: "2px solid #5b21b6",
            right: "-155px",
            top: "-18px",
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
            boxShadow: "0 0 110px #2e1065",
          }}
        >
          <div
            style={{
              width: "455px",
              height: "455px",
              borderRadius: "50%",
              border: "2px dashed #9333ea",
              display: "flex",
              alignItems: "center",
              justifyContent: "center",
            }}
          >
            <div
              style={{
                width: "220px",
                height: "220px",
                borderRadius: "38% 62% 52% 48%",
                background: "#7c3aed",
                border: "18px solid #2e1065",
                boxShadow: "0 0 80px #9333ea",
                transform: "rotate(28deg)",
              }}
            />
          </div>
        </div>

        <div
          style={{
            display: "flex",
            flexDirection: "column",
            justifyContent: "space-between",
            width: "100%",
          }}
        >
          <div
            style={{
              display: "flex",
              alignItems: "center",
              gap: "14px",
              fontSize: "21px",
              letterSpacing: "5px",
              color: "#e9d5ff",
            }}
          >
            <div
              style={{
                width: "15px",
                height: "15px",
                borderRadius: "50%",
                background: "#c084fc",
                boxShadow: "0 0 22px #9333ea",
              }}
            />
            IQBAL//SEC
          </div>

          <div style={{ display: "flex", flexDirection: "column", maxWidth: "850px" }}>
            <div
              style={{
                fontSize: "24px",
                color: "#a855f7",
                letterSpacing: "4px",
                marginBottom: "20px",
              }}
            >
              OFFENSIVE SECURITY • SUPPLY-CHAIN RESEARCH
            </div>
            <div
              style={{
                display: "flex",
                flexDirection: "column",
                fontSize: "91px",
                lineHeight: 0.82,
                fontWeight: 800,
                letterSpacing: "-7px",
              }}
            >
              BREAK SYSTEMS.
              <div style={{ color: "#c084fc" }}>BUILD RESILIENCE.</div>
            </div>
          </div>

          <div style={{ fontSize: "20px", color: "#b9a8d9", letterSpacing: "2px" }}>
            MUHAMMAD IQBAL LATIF // ISLAMABAD, PAKISTAN
          </div>
        </div>
      </div>
    ),
    size,
  );
}
```

## `app/page.tsx`

```tsx
import Portfolio from "@/components/Portfolio";

export default function Home() {
  return <Portfolio />;
}
```

## `components/BootSequence.tsx`

```tsx
"use client";

import { useEffect, useState } from "react";
import { AnimatePresence, motion, useReducedMotion } from "motion/react";

const BOOT_LINES = [
  "initializing secure session...",
  "mounting operator profile...",
  "verifying offensive toolchain...",
  "opening encrypted portfolio channel...",
] as const;

type BootSequenceProps = {
  onComplete: () => void;
};

export default function BootSequence({ onComplete }: BootSequenceProps) {
  const reduceMotion = useReducedMotion();
  const [visibleLines, setVisibleLines] = useState(0);
  const [isVisible, setIsVisible] = useState(true);

  useEffect(() => {
    if (reduceMotion) {
      const reducedTimer = window.setTimeout(() => setIsVisible(false), 0);
      return () => window.clearTimeout(reducedTimer);
    }

    const originalOverflow = document.body.style.overflow;
    document.body.style.overflow = "hidden";

    const timers = BOOT_LINES.map((_, index) =>
      window.setTimeout(() => setVisibleLines(index + 1), 180 + index * 280),
    );

    const finishTimer = window.setTimeout(() => {
      setIsVisible(false);
    }, 1650);

    const handleEscape = (event: KeyboardEvent) => {
      if (event.key === "Escape") {
        setIsVisible(false);
      }
    };

    window.addEventListener("keydown", handleEscape);

    return () => {
      timers.forEach(window.clearTimeout);
      window.clearTimeout(finishTimer);
      window.removeEventListener("keydown", handleEscape);
      document.body.style.overflow = originalOverflow;
    };
  }, [onComplete, reduceMotion]);

  const skip = () => {
    setIsVisible(false);
  };

  return (
    <AnimatePresence
      onExitComplete={() => {
        document.body.style.overflow = "";
        onComplete();
      }}
    >
      {isVisible && (
        <motion.div
          className="boot-sequence"
          role="dialog"
          aria-label="Portfolio boot sequence"
          initial={{ opacity: 1 }}
          exit={{ opacity: 0, clipPath: "inset(0 0 100% 0)" }}
          transition={{ duration: reduceMotion ? 0.01 : 0.58, ease: [0.76, 0, 0.24, 1] }}
        >
          <div className="boot-sequence__scan" aria-hidden="true" />
          <div className="boot-sequence__terminal">
            <div className="boot-sequence__topline">
              <span>IQBAL//SECURE_BOOT</span>
              <span>SESSION 026</span>
            </div>

            <div className="boot-sequence__output" aria-live="polite">
              {BOOT_LINES.slice(0, visibleLines).map((line, index) => (
                <motion.p
                  key={line}
                  initial={{ opacity: 0, x: -10 }}
                  animate={{ opacity: 1, x: 0 }}
                  transition={{ duration: 0.2 }}
                >
                  <span>{String(index + 1).padStart(2, "0")}</span>
                  {line}
                  {index === visibleLines - 1 && <i aria-hidden="true" />}
                </motion.p>
              ))}
            </div>

            <button className="boot-sequence__skip" type="button" onClick={skip}>
              SKIP BOOT <span>ESC</span>
            </button>
          </div>
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

## `components/CursorGlow.tsx`

```tsx
"use client";

import { useEffect, useState } from "react";
import { motion, useMotionValue, useSpring } from "motion/react";
import useMediaQuery from "@/lib/useMediaQuery";

type CursorMode = "default" | "action" | "card";

export default function CursorGlow() {
  const finePointer = useMediaQuery("(pointer: fine) and (min-width: 900px)");
  const [mode, setMode] = useState<CursorMode>("default");
  const [visible, setVisible] = useState(false);
  const pointerX = useMotionValue(-100);
  const pointerY = useMotionValue(-100);
  const x = useSpring(pointerX, { stiffness: 720, damping: 52, mass: 0.18 });
  const y = useSpring(pointerY, { stiffness: 720, damping: 52, mass: 0.18 });
  const glowX = useSpring(pointerX, { stiffness: 110, damping: 24, mass: 0.85 });
  const glowY = useSpring(pointerY, { stiffness: 110, damping: 24, mass: 0.85 });

  useEffect(() => {
    if (!finePointer) return;

    document.documentElement.classList.add("has-custom-cursor");

    const handlePointerMove = (event: PointerEvent) => {
      pointerX.set(event.clientX);
      pointerY.set(event.clientY);
      setVisible(true);
    };

    const handlePointerOver = (event: PointerEvent) => {
      const target = event.target instanceof Element ? event.target.closest<HTMLElement>("[data-cursor]") : null;
      const nextMode = target?.dataset.cursor;
      setMode(nextMode === "action" || nextMode === "card" ? nextMode : "default");
    };

    const handlePointerLeave = () => setVisible(false);

    window.addEventListener("pointermove", handlePointerMove, { passive: true });
    window.addEventListener("pointerover", handlePointerOver, { passive: true });
    document.documentElement.addEventListener("mouseleave", handlePointerLeave);

    return () => {
      document.documentElement.classList.remove("has-custom-cursor");
      window.removeEventListener("pointermove", handlePointerMove);
      window.removeEventListener("pointerover", handlePointerOver);
      document.documentElement.removeEventListener("mouseleave", handlePointerLeave);
    };
  }, [finePointer, pointerX, pointerY]);

  if (!finePointer) return null;

  return (
    <>
      <motion.div
        className="cursor-aura"
        aria-hidden="true"
        style={{ x: glowX, y: glowY, opacity: visible ? 1 : 0 }}
      />
      <motion.div
        className={`cursor-reticle cursor-reticle--${mode}`}
        aria-hidden="true"
        style={{ x, y, opacity: visible ? 1 : 0 }}
      >
        <span />
        <span />
        <span />
        <span />
        <i />
      </motion.div>
    </>
  );
}
```

## `components/CyberScene.tsx`

```tsx
"use client";

import { useMemo, useRef } from "react";
import { Canvas, useFrame } from "@react-three/fiber";
import { Float, Line } from "@react-three/drei";
import * as THREE from "three";
import type { MotionValue } from "motion/react";
import useMediaQuery from "@/lib/useMediaQuery";

const PALETTE = {
  void: "#0b0615",
  deepViolet: "#150a2e",
  indigoCore: "#2e1065",
  royalViolet: "#5b21b6",
  electricViolet: "#7c3aed",
  neonViolet: "#9333ea",
  plasmaLilac: "#a855f7",
  ultravioletGlow: "#c084fc",
  lavenderMist: "#e9d5ff",
} as const;

type CyberSceneProps = {
  exitProgress: MotionValue<number>;
  active: boolean;
  reducedMotion: boolean;
};

type SceneContentProps = CyberSceneProps & {
  compact: boolean;
};

function seeded(index: number, salt: number): number {
  const value = Math.sin(index * 12.9898 + salt * 78.233) * 43758.5453;
  return value - Math.floor(value);
}

function ParticleField({ compact, reducedMotion }: { compact: boolean; reducedMotion: boolean }) {
  const pointsRef = useRef<THREE.Points>(null);
  const count = compact ? 360 : 920;

  const positions = useMemo(() => {
    const values = new Float32Array(count * 3);

    for (let index = 0; index < count; index += 1) {
      const radius = 3.2 + seeded(index, 1) * 6.8;
      const theta = seeded(index, 2) * Math.PI * 2;
      const phi = Math.acos(2 * seeded(index, 3) - 1);

      values[index * 3] = radius * Math.sin(phi) * Math.cos(theta);
      values[index * 3 + 1] = radius * Math.sin(phi) * Math.sin(theta) * 0.66;
      values[index * 3 + 2] = radius * Math.cos(phi);
    }

    return values;
  }, [count]);

  useFrame((_, delta) => {
    if (!pointsRef.current || reducedMotion) return;
    pointsRef.current.rotation.y += delta * 0.018;
    pointsRef.current.rotation.x -= delta * 0.006;
  });

  return (
    <points ref={pointsRef} aria-hidden="true">
      <bufferGeometry>
        <bufferAttribute attach="attributes-position" args={[positions, 3]} />
      </bufferGeometry>
      <pointsMaterial
        color={PALETTE.ultravioletGlow}
        size={compact ? 0.026 : 0.022}
        sizeAttenuation
        transparent
        opacity={0.48}
        depthWrite={false}
        blending={THREE.AdditiveBlending}
      />
    </points>
  );
}

function PacketOrbit({
  radius,
  tilt,
  speed,
  offset,
  progress,
  reducedMotion,
}: {
  radius: number;
  tilt: [number, number, number];
  speed: number;
  offset: number;
  progress: MotionValue<number>;
  reducedMotion: boolean;
}) {
  const orbitRef = useRef<THREE.Group>(null);

  useFrame((state, delta) => {
    if (!orbitRef.current) return;
    const scrollDrive = progress.get();
    const timeDrive = reducedMotion ? 0 : state.clock.elapsedTime * speed;
    orbitRef.current.rotation.z = offset + timeDrive + scrollDrive * Math.PI * 1.6;
    orbitRef.current.rotation.y += reducedMotion ? 0 : delta * speed * 0.08;
  });

  return (
    <group ref={orbitRef} rotation={tilt}>
      <mesh rotation={[Math.PI / 2, 0, 0]}>
        <torusGeometry args={[radius, 0.006, 8, 160]} />
        <meshBasicMaterial color={PALETTE.royalViolet} transparent opacity={0.44} />
      </mesh>
      <mesh position={[radius, 0, 0]}>
        <icosahedronGeometry args={[0.09, 0]} />
        <meshStandardMaterial
          color={PALETTE.lavenderMist}
          emissive={PALETTE.neonViolet}
          emissiveIntensity={4.2}
          roughness={0.3}
          metalness={0.4}
        />
      </mesh>
    </group>
  );
}

function CyberBloom({ exitProgress, active, reducedMotion }: SceneContentProps) {
  const rootRef = useRef<THREE.Group>(null);
  const coreRef = useRef<THREE.Mesh>(null);
  const shellRef = useRef<THREE.Mesh>(null);
  const lineRef = useRef<THREE.Group>(null);

  const helixPoints = useMemo(() => {
    return Array.from({ length: 180 }, (_, index) => {
      const t = (index / 179) * Math.PI * 5.5;
      const radius = 1.75 + Math.sin(t * 0.5) * 0.14;
      return new THREE.Vector3(
        Math.cos(t) * radius,
        (index / 179 - 0.5) * 4.4,
        Math.sin(t) * radius,
      );
    });
  }, []);

  useFrame((state, delta) => {
    const exit = exitProgress.get();
    const pointerX = state.pointer.x;
    const pointerY = state.pointer.y;

    if (rootRef.current) {
      const targetScale = 1 - exit * 0.38;
      const targetZ = -exit * 3.2;
      rootRef.current.scale.lerp(new THREE.Vector3(targetScale, targetScale, targetScale), 0.08);
      rootRef.current.position.z = THREE.MathUtils.lerp(rootRef.current.position.z, targetZ, 0.08);
      rootRef.current.rotation.y = THREE.MathUtils.lerp(
        rootRef.current.rotation.y,
        pointerX * 0.16 + exit * 1.05,
        0.045,
      );
      rootRef.current.rotation.x = THREE.MathUtils.lerp(
        rootRef.current.rotation.x,
        pointerY * 0.1 - exit * 0.22,
        0.045,
      );
    }

    if (coreRef.current && !reducedMotion) {
      coreRef.current.rotation.x += delta * (active ? 0.22 : 0.08);
      coreRef.current.rotation.y += delta * (active ? 0.34 : 0.12);
    }

    if (shellRef.current && !reducedMotion) {
      shellRef.current.rotation.x -= delta * 0.08;
      shellRef.current.rotation.z += delta * 0.11;
    }

    if (lineRef.current && !reducedMotion) {
      lineRef.current.rotation.y -= delta * 0.055;
    }
  });

  return (
    <group ref={rootRef}>
      <Float speed={reducedMotion ? 0 : 1.4} rotationIntensity={0.18} floatIntensity={0.28}>
        <mesh ref={coreRef}>
          <torusKnotGeometry args={[1.16, 0.34, 220, 28, 2, 3]} />
          <meshPhysicalMaterial
            color={PALETTE.electricViolet}
            emissive={PALETTE.royalViolet}
            emissiveIntensity={2.6}
            roughness={0.24}
            metalness={0.72}
            clearcoat={1}
            clearcoatRoughness={0.18}
          />
        </mesh>

        <mesh ref={shellRef} scale={1.18}>
          <torusKnotGeometry args={[1.16, 0.34, 180, 18, 2, 3]} />
          <meshBasicMaterial
            color={PALETTE.ultravioletGlow}
            wireframe
            transparent
            opacity={0.24}
            blending={THREE.AdditiveBlending}
          />
        </mesh>
      </Float>

      <group ref={lineRef} rotation={[0.2, 0, 0.65]}>
        <Line
          points={helixPoints}
          color={PALETTE.plasmaLilac}
          lineWidth={0.7}
          transparent
          opacity={0.52}
        />
      </group>

      <PacketOrbit
        radius={2.15}
        tilt={[0.2, 0.8, 0.4]}
        speed={0.42}
        offset={0}
        progress={exitProgress}
        reducedMotion={reducedMotion}
      />
      <PacketOrbit
        radius={2.55}
        tilt={[1.25, 0.1, -0.35]}
        speed={-0.31}
        offset={1.8}
        progress={exitProgress}
        reducedMotion={reducedMotion}
      />
      <PacketOrbit
        radius={2.92}
        tilt={[0.75, -0.65, 0.2]}
        speed={0.22}
        offset={3.6}
        progress={exitProgress}
        reducedMotion={reducedMotion}
      />

      <mesh scale={3.25}>
        <sphereGeometry args={[1, 32, 32]} />
        <meshBasicMaterial
          color={PALETTE.neonViolet}
          transparent
          opacity={0.035}
          side={THREE.BackSide}
        />
      </mesh>
    </group>
  );
}

export default function CyberScene({ exitProgress, active, reducedMotion }: CyberSceneProps) {
  const compact = useMediaQuery("(max-width: 900px), (pointer: coarse)");

  return (
    <div className="cyber-scene" aria-label="Interactive violet cyber bloom visualization">
      <Canvas
        camera={{ position: [0, 0, compact ? 7.8 : 6.6], fov: compact ? 52 : 46 }}
        dpr={compact ? [1, 1.25] : [1, 1.7]}
        gl={{ alpha: true, antialias: !compact, powerPreference: "high-performance" }}
      >
        <ambientLight intensity={0.42} color={PALETTE.lavenderMist} />
        <pointLight position={[4, 4, 5]} intensity={48} color={PALETTE.ultravioletGlow} distance={16} />
        <pointLight position={[-4, -2, 3]} intensity={38} color={PALETTE.electricViolet} distance={14} />
        <pointLight position={[0, -5, -2]} intensity={26} color={PALETTE.neonViolet} distance={12} />
        <ParticleField compact={compact} reducedMotion={reducedMotion} />
        <CyberBloom
          exitProgress={exitProgress}
          active={active}
          reducedMotion={reducedMotion}
          compact={compact}
        />
      </Canvas>
      <div className="cyber-scene__hud cyber-scene__hud--top" aria-hidden="true">
        <span>LIVE CORE</span>
        <i />
        <span>R3F // 026</span>
      </div>
      <div className="cyber-scene__hud cyber-scene__hud--bottom" aria-hidden="true">
        <span>ROTATION VECTOR</span>
        <span>VIOLET CHANNEL</span>
      </div>
    </div>
  );
}
```

## `components/MagneticLink.tsx`

```tsx
"use client";

import { useRef } from "react";
import { motion, useMotionValue, useSpring, type HTMLMotionProps } from "motion/react";

type MagneticLinkProps = HTMLMotionProps<"a">;

export default function MagneticLink({ children, className, ...props }: MagneticLinkProps) {
  const ref = useRef<HTMLAnchorElement>(null);
  const rawX = useMotionValue(0);
  const rawY = useMotionValue(0);
  const x = useSpring(rawX, { stiffness: 360, damping: 24, mass: 0.35 });
  const y = useSpring(rawY, { stiffness: 360, damping: 24, mass: 0.35 });

  const handlePointerMove = (event: React.PointerEvent<HTMLAnchorElement>) => {
    const element = ref.current;
    if (!element || event.pointerType === "touch") return;

    const rect = element.getBoundingClientRect();
    const offsetX = event.clientX - (rect.left + rect.width / 2);
    const offsetY = event.clientY - (rect.top + rect.height / 2);

    rawX.set(offsetX * 0.16);
    rawY.set(offsetY * 0.16);
  };

  const reset = () => {
    rawX.set(0);
    rawY.set(0);
  };

  return (
    <motion.a
      ref={ref}
      className={className}
      style={{ x, y }}
      data-cursor="action"
      onPointerMove={handlePointerMove}
      onPointerLeave={reset}
      onPointerCancel={reset}
      {...props}
    >
      {children}
    </motion.a>
  );
}
```

## `components/Navbar.tsx`

```tsx
"use client";

import { useState } from "react";
import { Menu, X } from "lucide-react";
import { AnimatePresence, motion, type MotionValue } from "motion/react";

export type ChapterNavItem = {
  id: string;
  label: string;
  code: string;
};

type NavbarProps = {
  chapters: readonly ChapterNavItem[];
  activeIndex: number;
  progress: MotionValue<number>;
  onNavigate: (index: number) => void;
};

export default function Navbar({ chapters, activeIndex, progress, onNavigate }: NavbarProps) {
  const [open, setOpen] = useState(false);


  return (
    <>
      <motion.div className="chapter-progress" aria-hidden="true" style={{ scaleX: progress }} />

      <header className="site-nav-shell">
        <nav className="site-nav" aria-label="Portfolio chapters">
          <button
            className="site-mark"
            type="button"
            onClick={() => onNavigate(0)}
            aria-label="Return to opening chapter"
            data-cursor="action"
          >
            <span className="site-mark__glyph" aria-hidden="true">
              <i />
              <i />
              <i />
            </span>
            <span>
              IQBAL<em>{"//"}</em>SEC
            </span>
          </button>

          <div className="site-nav__desktop">
            {chapters.map((chapter, index) => (
              <button
                key={chapter.id}
                type="button"
                className={index === activeIndex ? "is-active" : undefined}
                onClick={() => onNavigate(index)}
                data-cursor="action"
                aria-current={index === activeIndex ? "page" : undefined}
              >
                <small>{chapter.code}</small>
                {chapter.label}
                <i />
              </button>
            ))}
          </div>

          <div className="site-nav__status" aria-live="polite">
            <span>{chapters[activeIndex]?.code ?? "00"}</span>
            <p>{chapters[activeIndex]?.label ?? "Origin"}</p>
          </div>

          <button
            className="nav-toggle"
            type="button"
            onClick={() => setOpen((current) => !current)}
            aria-expanded={open}
            aria-controls="mobile-navigation"
            aria-label={open ? "Close navigation" : "Open navigation"}
            data-cursor="action"
          >
            {open ? <X size={20} /> : <Menu size={20} />}
          </button>
        </nav>
      </header>

      <AnimatePresence>
        {open && (
          <motion.div
            id="mobile-navigation"
            className="mobile-nav"
            initial={{ clipPath: "inset(0 0 100% 0)" }}
            animate={{ clipPath: "inset(0 0 0% 0)" }}
            exit={{ clipPath: "inset(0 0 100% 0)" }}
            transition={{ duration: 0.45, ease: [0.76, 0, 0.24, 1] }}
          >
            <div className="mobile-nav__label">CHAPTER DIRECTORY</div>
            {chapters.map((chapter, index) => (
              <button
                key={chapter.id}
                type="button"
                onClick={() => {
                  setOpen(false);
                  onNavigate(index);
                }}
                className={index === activeIndex ? "is-active" : undefined}
              >
                <span>{chapter.code}</span>
                {chapter.label}
              </button>
            ))}
          </motion.div>
        )}
      </AnimatePresence>

      <aside className="chapter-rail" aria-label="Chapter progress">
        {chapters.map((chapter, index) => (
          <button
            key={chapter.id}
            type="button"
            className={index === activeIndex ? "is-active" : undefined}
            onClick={() => onNavigate(index)}
            aria-label={`Open chapter ${chapter.code}: ${chapter.label}`}
            data-cursor="action"
          >
            <span>{chapter.code}</span>
            <i />
          </button>
        ))}
      </aside>
    </>
  );
}
```

## `components/Portfolio.tsx`

```tsx
"use client";

import dynamic from "next/dynamic";
import {
  useCallback,
  useEffect,
  useRef,
  useState,
  type CSSProperties,
  type ReactNode,
} from "react";
import {
  ArrowDownRight,
  ArrowRight,
  ArrowUpRight,
  ChevronDown,
  Download,
  Mail,
  MapPin,
  Phone,
  ShieldCheck,
  Sparkles,
  Terminal,
} from "lucide-react";
import {
  AnimatePresence,
  MotionConfig,
  motion,
  useMotionValueEvent,
  useReducedMotion,
  useScroll,
  useSpring,
  useTransform,
  type MotionValue,
} from "motion/react";
import BootSequence from "@/components/BootSequence";
import CursorGlow from "@/components/CursorGlow";
import MagneticLink from "@/components/MagneticLink";
import Navbar, { type ChapterNavItem } from "@/components/Navbar";
import SectionHeading from "@/components/SectionHeading";
import {
  HERO_SIGNALS,
  METRICS,
  NAV_ITEMS,
  PROJECTS,
  SKILL_GROUPS,
  TIMELINE,
  TOOL_NODES,
} from "@/lib/data";
import useMediaQuery from "@/lib/useMediaQuery";

const CyberScene = dynamic(() => import("@/components/CyberScene"), {
  ssr: false,
  loading: () => (
    <div className="scene-loader" role="status" aria-label="Loading 3D cyber visualization">
      <span />
      <span />
      <span />
    </div>
  ),
});

const CHAPTERS: readonly ChapterNavItem[] = [
  { id: "top", label: "Origin", code: "00" },
  ...NAV_ITEMS.map((item, index) => ({
    id: item.href.slice(1),
    label: item.label,
    code: String(index + 1).padStart(2, "0"),
  })),
];

const PANEL_COUNT = CHAPTERS.length;

type PanelFrameProps = {
  id: string;
  index: number;
  progress: MotionValue<number>;
  horizontal: boolean;
  active: boolean;
  children: ReactNode;
  className?: string;
};

type OrbitNodeStyle = CSSProperties & { "--node-angle": string };

type Project = (typeof PROJECTS)[number];

type ProjectCaseProps = {
  project: Project;
  cardIndex: number;
  chapterProgress: MotionValue<number>;
  horizontal: boolean;
  expanded: boolean;
  onToggle: () => void;
};

function PanelFrame({
  id,
  index,
  progress,
  horizontal,
  active,
  children,
  className = "",
}: PanelFrameProps) {
  const segment = 1 / (PANEL_COUNT - 1);
  const center = index * segment;
  const opacity = useTransform(
    progress,
    [center - segment * 0.78, center, center + segment * 0.78],
    [0.24, 1, 0.24],
  );
  const rotateY = useTransform(
    progress,
    [center - segment, center, center + segment],
    [11, 0, -11],
  );
  const rotateX = useTransform(
    progress,
    [center - segment, center, center + segment],
    [-2.5, 0, 2.5],
  );
  const z = useTransform(
    progress,
    [center - segment, center, center + segment],
    [-150, 0, -150],
  );
  const scale = useTransform(
    progress,
    [center - segment, center, center + segment],
    [0.94, 1, 0.94],
  );

  return (
    <motion.section
      id={id}
      className={`chapter-panel chapter-panel--${index} ${active ? "is-active" : ""} ${className}`}
      data-panel-index={index}
      aria-label={`${CHAPTERS[index].label} chapter`}
      style={horizontal ? { opacity, rotateY, rotateX, z, scale } : undefined}
    >
      <div className="chapter-panel__index" aria-hidden="true">
        {CHAPTERS[index].code}
      </div>
      {children}
    </motion.section>
  );
}

function SignalStrip() {
  return (
    <div className="signal-strip" aria-hidden="true">
      <motion.div
        className="signal-strip__track"
        animate={{ x: ["0%", "-50%"] }}
        transition={{ duration: 26, ease: "linear", repeat: Number.POSITIVE_INFINITY }}
      >
        {[...HERO_SIGNALS, ...HERO_SIGNALS].map((signal, index) => (
          <span key={`${signal}-${index}`}>
            <i />
            {signal}
          </span>
        ))}
      </motion.div>
    </div>
  );
}

function HeroPanel({
  active,
  exitProgress,
  reducedMotion,
  onNavigate,
}: {
  active: boolean;
  exitProgress: MotionValue<number>;
  reducedMotion: boolean;
  onNavigate: (index: number) => void;
}) {
  return (
    <div className="hero-panel__layout">
      <div className="hero-copy">
        <motion.div
          className="status-pill"
          initial={{ opacity: 0, y: 12 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.18, duration: 0.55 }}
        >
          <span />
          SECURE CHANNEL AVAILABLE
        </motion.div>

        <div className="hero-copy__meta">
          <span>MUHAMMAD IQBAL LATIF</span>
          <span>ISLAMABAD // PAKISTAN</span>
        </div>

        <h1 className={active ? "hero-title glitch-reveal is-active" : "hero-title glitch-reveal"} data-text="BREAK SYSTEMS. BUILD RESILIENCE.">
          <span>BREAK</span>
          <span>SYSTEMS.</span>
          <span className="hero-title__accent">BUILD RESILIENCE.</span>
        </h1>

        <motion.p
          className="hero-copy__intro"
          initial={{ opacity: 0, y: 22 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.58, duration: 0.72, ease: [0.16, 1, 0.3, 1] }}
        >
          Cyber-security student and offensive-security practitioner focused on penetration testing,
          explainable npm supply-chain defense, digital forensics, cryptography, and CTF engineering.
        </motion.p>

        <motion.div
          className="hero-actions"
          initial={{ opacity: 0, y: 18 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.78, duration: 0.6 }}
        >
          <MagneticLink
            href="#projects"
            className="action-link action-link--primary"
            onClick={(event) => {
              event.preventDefault();
              onNavigate(2);
            }}
          >
            ENTER CASE FILES <ArrowDownRight size={18} />
          </MagneticLink>
          <MagneticLink
            href="/Muhammad-Iqbal-Latif-CV.pdf"
            className="action-link action-link--ghost"
            download
          >
            DOWNLOAD CV <Download size={17} />
          </MagneticLink>
        </motion.div>

        <div className="hero-copy__footnote">
          <span>SCROLL VERTICAL</span>
          <i />
          <span>MOVE HORIZONTAL</span>
        </div>
      </div>

      <motion.div
        className="hero-scene-shell"
        initial={{ opacity: 0, scale: 0.82, rotateY: -12 }}
        animate={{ opacity: 1, scale: 1, rotateY: 0 }}
        transition={{ delay: 0.35, duration: 1.1, ease: [0.16, 1, 0.3, 1] }}
      >
        <CyberScene exitProgress={exitProgress} active={active} reducedMotion={reducedMotion} />
        <div className="hero-scene-shell__caption">
          <span>CYBER BLOOM</span>
          <p>ATTACK SURFACE // LIVE MODEL</p>
        </div>
      </motion.div>

      <SignalStrip />
    </div>
  );
}

function AboutPanel({ active }: { active: boolean }) {
  return (
    <div className="panel-inner about-panel">
      <SectionHeading
        index="01"
        eyebrow="OPERATOR PROFILE // ABOUT"
        title="SECURITY THROUGH ADVERSARIAL THINKING."
        active={active}
      />

      <div className="about-panel__grid">
        <motion.div
          className="about-manifesto"
          initial={false}
          animate={active ? { rotateY: 0, x: 0, opacity: 1 } : { rotateY: 9, x: 80, opacity: 0.28 }}
          transition={{ duration: 0.7, ease: [0.16, 1, 0.3, 1] }}
        >
          <p className="about-manifesto__lead">
            I study systems by tracing where assumptions fail: exposed interfaces, insecure trust
            boundaries, weak package updates, malformed traffic, and logic that behaves differently
            under pressure.
          </p>
          <p>
            My work connects offensive testing with explainable defensive analysis. That includes
            full exploitation chains in authorized labs, forensic investigation, cryptographic
            challenge solving, and an academic focus on malicious npm supply-chain behavior.
          </p>
          <div className="about-manifesto__tags">
            <span>OFFENSIVE SECURITY</span>
            <span>SECURITY RESEARCH</span>
            <span>CTF ENGINEERING</span>
          </div>
        </motion.div>

        <div className="metric-matrix" aria-label="Portfolio metrics">
          {METRICS.map((metric, index) => (
            <motion.article
              key={metric.label}
              className="metric-cell"
              initial={false}
              animate={
                active
                  ? { opacity: 1, rotateX: 0, z: 0, y: 0 }
                  : { opacity: 0.24, rotateX: index % 2 === 0 ? 16 : -16, z: -70, y: 30 }
              }
              transition={{ delay: index * 0.07, duration: 0.62, ease: [0.16, 1, 0.3, 1] }}
            >
              <span>{metric.value}</span>
              <p>{metric.label}</p>
              <i />
            </motion.article>
          ))}
        </div>

        <motion.div
          className="education-file"
          initial={false}
          animate={active ? { opacity: 1, y: 0, rotateX: 0 } : { opacity: 0.3, y: 55, rotateX: -8 }}
          transition={{ delay: 0.18, duration: 0.72, ease: [0.16, 1, 0.3, 1] }}
        >
          <div className="education-file__topline">
            <span>EDU_RECORD</span>
            <span>ACTIVE</span>
          </div>
          <h3>B.S. Cyber Security</h3>
          <p>COMSATS University Islamabad</p>
          <div className="education-file__details">
            <span>CGPA 3.45 / 4.00</span>
            <span>EXPECTED JULY 2027</span>
          </div>
          <small>
            Network Security · Cryptography · Ethical Hacking · Digital Forensics · Web Technologies
          </small>
        </motion.div>
      </div>
    </div>
  );
}

function ProjectCase({
  project,
  cardIndex,
  chapterProgress,
  horizontal,
  expanded,
  onToggle,
}: ProjectCaseProps) {
  const depth = cardIndex - 1;
  const x = useTransform(chapterProgress, [-1, 0, 1], [110 + depth * 60, depth * 28, -85 - depth * 45]);
  const y = useTransform(chapterProgress, [-1, 0, 1], [depth * 45, -depth * 16, depth * -30]);
  const rotateY = useTransform(chapterProgress, [-1, 0, 1], [13 + depth * 4, depth * -2, -12 + depth * 3]);
  const z = useTransform(chapterProgress, [-1, 0, 1], [-110 - Math.abs(depth) * 40, depth * 32, -90]);
  const Icon = project.icon;

  return (
    <motion.article
      className={`project-case ${expanded ? "is-expanded" : ""}`}
      style={horizontal ? { x, y, rotateY, z } : undefined}
      data-cursor="card"
      layout
    >
      <button
        type="button"
        className="project-case__trigger"
        onClick={onToggle}
        aria-expanded={expanded}
        data-cursor="card"
      >
        <div className="project-case__topline">
          <span>CASE_{project.index}</span>
          <span>{project.year}</span>
        </div>
        <div className="project-case__icon">
          <Icon size={24} strokeWidth={1.6} />
        </div>
        <p className="project-case__type">{project.type}</p>
        <h3>{project.title}</h3>
        <p className="project-case__summary">{project.summary}</p>
        <div className="project-case__toggle">
          <span>{expanded ? "COLLAPSE FILE" : "EXPAND FILE"}</span>
          <ChevronDown size={17} />
        </div>
      </button>

      <AnimatePresence initial={false}>
        {expanded && (
          <motion.div
            className="project-case__details"
            initial={{ height: 0, opacity: 0 }}
            animate={{ height: "auto", opacity: 1 }}
            exit={{ height: 0, opacity: 0 }}
            transition={{ duration: 0.42, ease: [0.16, 1, 0.3, 1] }}
          >
            <div>
              <p>OUTCOMES</p>
              <ul>
                {project.outcomes.map((outcome) => (
                  <li key={outcome}>{outcome}</li>
                ))}
              </ul>
            </div>
            <div>
              <p>STACK / DOMAIN</p>
              <div className="project-case__stack">
                {project.stack.map((item) => (
                  <span key={item}>{item}</span>
                ))}
              </div>
            </div>
          </motion.div>
        )}
      </AnimatePresence>
    </motion.article>
  );
}

function ProjectsPanel({
  active,
  progress,
  horizontal,
}: {
  active: boolean;
  progress: MotionValue<number>;
  horizontal: boolean;
}) {
  const [expanded, setExpanded] = useState(0);
  const center = 2 / (PANEL_COUNT - 1);
  const span = 1 / (PANEL_COUNT - 1);
  const chapterProgress = useTransform(progress, [center - span, center + span], [-1, 1]);

  return (
    <div className="panel-inner projects-panel">
      <SectionHeading
        index="02"
        eyebrow="CASE ARCHIVE // PROJECTS"
        title="RESEARCH BUILT AS WORKING SYSTEMS."
        active={active}
      />

      <div className="projects-stage">
        {PROJECTS.map((project, index) => (
          <ProjectCase
            key={project.index}
            project={project}
            cardIndex={index}
            chapterProgress={chapterProgress}
            horizontal={horizontal}
            expanded={expanded === index}
            onToggle={() => setExpanded((current) => (current === index ? -1 : index))}
          />
        ))}
      </div>
      <div className="projects-panel__note">
        <Terminal size={16} /> SELECT A CASE FILE TO INSPECT OUTCOMES AND DOMAIN STACK
      </div>
    </div>
  );
}

function ArsenalPanel({
  active,
  progress,
  horizontal,
}: {
  active: boolean;
  progress: MotionValue<number>;
  horizontal: boolean;
}) {
  const center = 3 / (PANEL_COUNT - 1);
  const span = 1 / (PANEL_COUNT - 1);
  const chapterProgress = useTransform(progress, [center - span, center + span], [-1, 1]);
  const ringRotation = useTransform(chapterProgress, [-1, 1], [-95, 125]);
  const counterRotation = useTransform(ringRotation, (value) => -value);

  return (
    <div className="panel-inner arsenal-panel">
      <SectionHeading
        index="03"
        eyebrow="CAPABILITY MAP // ARSENAL"
        title="TOOLS ARE USEFUL. METHODOLOGY IS DECISIVE."
        active={active}
      />

      <div className="arsenal-panel__layout">
        <div className="tool-orbit-shell" aria-label="Scroll-driven security tool orbit">
          <div className="tool-orbit-shell__grid" aria-hidden="true" />
          <motion.div
            className="tool-orbit"
            style={horizontal ? { rotate: ringRotation } : undefined}
          >
            <div className="tool-orbit__ring" />
            {TOOL_NODES.map((tool) => {
              const Icon = tool.icon;
              const style: OrbitNodeStyle = { "--node-angle": `${tool.orbit * 36}deg` };
              return (
                <div className="tool-node-position" style={style} key={tool.label}>
                  <motion.div
                    className="tool-node"
                    style={horizontal ? { rotate: counterRotation } : undefined}
                    data-cursor="card"
                  >
                    <Icon size={18} strokeWidth={1.6} />
                    <span>{tool.label}</span>
                  </motion.div>
                </div>
              );
            })}
            <div className="tool-orbit__core">
              <ShieldCheck size={35} strokeWidth={1.4} />
              <span>OPERATOR</span>
              <small>SCROLL DRIVEN</small>
            </div>
          </motion.div>
        </div>

        <div className="skill-groups">
          {SKILL_GROUPS.map((group, index) => {
            const Icon = group.icon;
            return (
              <motion.article
                key={group.title}
                className="skill-group"
                initial={false}
                animate={
                  active
                    ? { opacity: 1, x: 0, rotateY: 0 }
                    : { opacity: 0.22, x: 65 + index * 12, rotateY: -8 - index * 2 }
                }
                transition={{ delay: index * 0.065, duration: 0.58, ease: [0.16, 1, 0.3, 1] }}
                data-cursor="card"
              >
                <div className="skill-group__icon">
                  <Icon size={21} strokeWidth={1.6} />
                </div>
                <div>
                  <h3>{group.title}</h3>
                  <p>{group.description}</p>
                  <div className="skill-group__tags">
                    {group.skills.map((skill) => (
                      <span key={skill}>{skill}</span>
                    ))}
                  </div>
                </div>
              </motion.article>
            );
          })}
        </div>
      </div>
    </div>
  );
}

function TimelinePanel({ active }: { active: boolean }) {
  return (
    <div className="panel-inner timeline-panel">
      <SectionHeading
        index="04"
        eyebrow="TRACE LOG // EXPERIENCE"
        title="A CONTINUOUS RECORD OF PRACTICE."
        active={active}
      />

      <div className="timeline-track">
        <div className="timeline-track__line" aria-hidden="true" />
        {TIMELINE.map((item, index) => {
          const Icon = item.icon;
          return (
            <motion.article
              className="timeline-entry"
              key={`${item.period}-${item.title}`}
              initial={false}
              animate={
                active
                  ? { opacity: 1, y: 0, rotateX: 0, z: 0 }
                  : {
                      opacity: 0.22,
                      y: index % 2 === 0 ? 75 : -55,
                      rotateX: index % 2 === 0 ? 11 : -11,
                      z: -80,
                    }
              }
              transition={{ delay: index * 0.07, duration: 0.66, ease: [0.16, 1, 0.3, 1] }}
              data-cursor="card"
            >
              <div className="timeline-entry__node">
                <Icon size={18} />
              </div>
              <span>{item.period}</span>
              <h3>{item.title}</h3>
              <p className="timeline-entry__subtitle">{item.subtitle}</p>
              <p>{item.body}</p>
            </motion.article>
          );
        })}
      </div>
    </div>
  );
}

function ContactPanel({ active }: { active: boolean }) {
  return (
    <div className="contact-panel">
      <div className="contact-panel__mesh" aria-hidden="true">
        <span />
        <span />
        <span />
      </div>

      <div className="contact-panel__topline">
        <span>FINAL CHAPTER // OPEN CHANNEL</span>
        <span>STATUS: AVAILABLE</span>
      </div>

      <motion.div
        className="contact-panel__main"
        initial={false}
        animate={active ? { opacity: 1, scale: 1, rotateY: 0 } : { opacity: 0.24, scale: 0.9, rotateY: 12 }}
        transition={{ duration: 0.78, ease: [0.16, 1, 0.3, 1] }}
      >
        <p>SECURE CONNECTION REQUEST</p>
        <h2 className={active ? "glitch-reveal is-active" : "glitch-reveal"} data-text="LET'S INVESTIGATE WHAT OTHERS OVERLOOK.">
          LET&apos;S INVESTIGATE
          <span>WHAT OTHERS OVERLOOK.</span>
        </h2>
        <p className="contact-panel__intro">
          Available for security research, penetration-testing collaboration, internships, CTF work,
          and technically demanding projects where clear analysis matters.
        </p>

        <div className="contact-actions">
          <MagneticLink
            className="contact-action contact-action--primary"
            href="mailto:fa23-bct-026@isbstudent.comsats.edu.pk"
          >
            <span>
              <Mail size={20} /> EMAIL CHANNEL
            </span>
            <ArrowUpRight size={24} />
          </MagneticLink>
          <MagneticLink className="contact-action" href="tel:+923328899801">
            <span>
              <Phone size={20} /> PHONE CHANNEL
            </span>
            <ArrowUpRight size={24} />
          </MagneticLink>
          <MagneticLink className="contact-action" href="/Muhammad-Iqbal-Latif-CV.pdf" download>
            <span>
              <Download size={20} /> DOWNLOAD DOSSIER
            </span>
            <ArrowDownRight size={24} />
          </MagneticLink>
        </div>
      </motion.div>

      <footer className="contact-panel__footer">
        <span>
          <MapPin size={15} /> ISLAMABAD, PAKISTAN
        </span>
        <span>MUHAMMAD IQBAL LATIF // 2026</span>
        <span>BUILT WITH NEXT.JS · MOTION · R3F</span>
      </footer>
    </div>
  );
}

function BackgroundField({ progress }: { progress: MotionValue<number> }) {
  const meshOneX = useTransform(progress, [0, 1], ["-8vw", "48vw"]);
  const meshTwoX = useTransform(progress, [0, 1], ["62vw", "5vw"]);
  const meshOneRotate = useTransform(progress, [0, 1], [-12, 22]);
  const meshTwoRotate = useTransform(progress, [0, 1], [18, -26]);

  return (
    <div className="background-field" aria-hidden="true">
      <motion.div
        className="gradient-mesh gradient-mesh--one"
        style={{ x: meshOneX, rotate: meshOneRotate }}
      />
      <motion.div
        className="gradient-mesh gradient-mesh--two"
        style={{ x: meshTwoX, rotate: meshTwoRotate }}
      />
      <div className="background-field__grid" />
      <div className="background-field__scan" />
    </div>
  );
}

export default function Portfolio() {
  const scrollRootRef = useRef<HTMLDivElement>(null);
  const desktopFine = useMediaQuery("(min-width: 1100px) and (pointer: fine)");
  const prefersReducedMotion = Boolean(useReducedMotion());
  const horizontal = desktopFine && !prefersReducedMotion;
  const [activeIndex, setActiveIndex] = useState(0);
  const [bootComplete, setBootComplete] = useState(false);
  const completeBoot = useCallback(() => setBootComplete(true), []);

  const { scrollYProgress } = useScroll({
    target: scrollRootRef,
    offset: ["start start", "end end"],
  });

  const progress = useSpring(scrollYProgress, {
    stiffness: 115,
    damping: 28,
    mass: 0.35,
  });

  const trackX = useTransform(progress, [0, 1], ["0vw", `-${(PANEL_COUNT - 1) * 100}vw`]);
  const heroExit = useTransform(progress, [0, 1 / (PANEL_COUNT - 1)], [0, 1]);

  useMotionValueEvent(progress, "change", (latest) => {
    if (!horizontal) return;
    const nextIndex = Math.max(0, Math.min(PANEL_COUNT - 1, Math.round(latest * (PANEL_COUNT - 1))));
    setActiveIndex((current) => (current === nextIndex ? current : nextIndex));
  });

  useEffect(() => {
    if (horizontal) return;

    const sections = Array.from(document.querySelectorAll<HTMLElement>("[data-panel-index]"));
    const observer = new IntersectionObserver(
      (entries) => {
        const visible = entries
          .filter((entry) => entry.isIntersecting)
          .sort((a, b) => b.intersectionRatio - a.intersectionRatio)[0];
        if (!visible) return;
        const index = Number((visible.target as HTMLElement).dataset.panelIndex ?? 0);
        setActiveIndex(index);
      },
      { rootMargin: "-28% 0px -48% 0px", threshold: [0.1, 0.3, 0.55, 0.8] },
    );

    sections.forEach((section) => observer.observe(section));
    return () => observer.disconnect();
  }, [horizontal]);

  const navigateTo = useCallback(
    (index: number) => {
      const resolvedIndex = Math.max(0, Math.min(PANEL_COUNT - 1, index));
      const behavior: ScrollBehavior = prefersReducedMotion ? "auto" : "smooth";

      if (horizontal && scrollRootRef.current) {
        const root = scrollRootRef.current;
        const rootTop = root.getBoundingClientRect().top + window.scrollY;
        const scrollableDistance = root.offsetHeight - window.innerHeight;
        const target = rootTop + scrollableDistance * (resolvedIndex / (PANEL_COUNT - 1));
        window.scrollTo({ top: target, behavior });
        return;
      }

      document.getElementById(CHAPTERS[resolvedIndex].id)?.scrollIntoView({ behavior, block: "start" });
    },
    [horizontal, prefersReducedMotion],
  );

  return (
    <MotionConfig reducedMotion="user" transition={{ ease: [0.16, 1, 0.3, 1] }}>
      {!bootComplete && <BootSequence onComplete={completeBoot} />}
      <div className={`portfolio-shell ${horizontal ? "is-horizontal" : "is-vertical"}`}>
        <BackgroundField progress={progress} />
        <div className="noise-layer" aria-hidden="true" />
        <CursorGlow />
        <Navbar
          chapters={CHAPTERS}
          activeIndex={activeIndex}
          progress={progress}
          onNavigate={navigateTo}
        />

        <div
          ref={scrollRootRef}
          className="horizontal-scroll-root"
          style={horizontal ? { height: `${PANEL_COUNT * 100}svh` } : undefined}
        >
          <div className="horizontal-sticky-viewport">
            <motion.main
              className="horizontal-track"
              style={horizontal ? { x: trackX } : undefined}
              aria-label="Muhammad Iqbal Latif cybersecurity portfolio"
            >
              <PanelFrame
                id="top"
                index={0}
                progress={progress}
                horizontal={horizontal}
                active={activeIndex === 0}
                className="hero-panel"
              >
                <HeroPanel
                  active={activeIndex === 0}
                  exitProgress={heroExit}
                  reducedMotion={prefersReducedMotion}
                  onNavigate={navigateTo}
                />
              </PanelFrame>

              <PanelFrame
                id="about"
                index={1}
                progress={progress}
                horizontal={horizontal}
                active={activeIndex === 1}
              >
                <AboutPanel active={activeIndex === 1} />
              </PanelFrame>

              <PanelFrame
                id="projects"
                index={2}
                progress={progress}
                horizontal={horizontal}
                active={activeIndex === 2}
              >
                <ProjectsPanel
                  active={activeIndex === 2}
                  progress={progress}
                  horizontal={horizontal}
                />
              </PanelFrame>

              <PanelFrame
                id="arsenal"
                index={3}
                progress={progress}
                horizontal={horizontal}
                active={activeIndex === 3}
              >
                <ArsenalPanel
                  active={activeIndex === 3}
                  progress={progress}
                  horizontal={horizontal}
                />
              </PanelFrame>

              <PanelFrame
                id="timeline"
                index={4}
                progress={progress}
                horizontal={horizontal}
                active={activeIndex === 4}
              >
                <TimelinePanel active={activeIndex === 4} />
              </PanelFrame>

              <PanelFrame
                id="contact"
                index={5}
                progress={progress}
                horizontal={horizontal}
                active={activeIndex === 5}
                className="chapter-panel--contact"
              >
                <ContactPanel active={activeIndex === 5} />
              </PanelFrame>
            </motion.main>
          </div>
        </div>

        <div className="mode-indicator" aria-hidden="true">
          <Sparkles size={14} />
          <span>{horizontal ? "HORIZONTAL TRACE MODE" : "VERTICAL FIELD MODE"}</span>
          <ArrowRight size={14} />
        </div>
      </div>
    </MotionConfig>
  );
}
```

## `components/SectionHeading.tsx`

```tsx
"use client";

import { motion } from "motion/react";

type SectionHeadingProps = {
  index: string;
  eyebrow: string;
  title: string;
  active: boolean;
  align?: "left" | "right";
};

export default function SectionHeading({
  index,
  eyebrow,
  title,
  active,
  align = "left",
}: SectionHeadingProps) {
  return (
    <motion.header
      className={`section-heading section-heading--${align}`}
      initial={false}
      animate={active ? "active" : "idle"}
      variants={{
        idle: { opacity: 0.45 },
        active: { opacity: 1 },
      }}
      transition={{ duration: 0.35 }}
    >
      <div className="section-heading__meta">
        <span>{index}</span>
        <i />
        <p>{eyebrow}</p>
      </div>
      <h2 className={active ? "glitch-reveal is-active" : "glitch-reveal"} data-text={title}>
        {title}
      </h2>
    </motion.header>
  );
}
```

## `lib/data.ts`

```ts
import {
  Binary,
  Blocks,
  Braces,
  Bug,
  Cable,
  Cpu,
  FileKey,
  Fingerprint,
  Network,
  Radar,
  ScanSearch,
  ServerCog,
  ShieldCheck,
  TerminalSquare,
  Workflow,
} from "lucide-react";

export const NAV_ITEMS = [
  { label: "About", href: "#about" },
  { label: "Projects", href: "#projects" },
  { label: "Arsenal", href: "#arsenal" },
  { label: "Timeline", href: "#timeline" },
  { label: "Contact", href: "#contact" },
] as const;

export const HERO_SIGNALS = [
  "OFFENSIVE SECURITY",
  "SUPPLY-CHAIN RESEARCH",
  "CTF ENGINEERING",
  "DIGITAL FORENSICS",
] as const;

export const PROJECTS = [
  {
    index: "01",
    title: "AI-Powered npm Supply Chain Security System",
    year: "2026",
    type: "Final Year Project",
    summary:
      "An explainable security analysis pipeline for detecting malicious npm package updates through differential code analysis, attack-chain scoring, and machine-learning risk evaluation.",
    outcomes: [
      "Differential package-version analysis",
      "ML-assisted malicious update detection",
      "Attack-chain scoring and compliance mapping",
    ],
    stack: ["Code Intelligence", "Machine Learning", "Supply Chain", "Explainability"],
    icon: Workflow,
  },
  {
    index: "02",
    title: "Chess Encryption Document System",
    year: "2024",
    type: "Applied Cryptography",
    summary:
      "A secure document-management concept using a custom cipher whose key transformations are derived from chess-piece movement patterns.",
    outcomes: [
      "Custom encryption-key model",
      "Secure document storage workflow",
      "Rule-driven cryptographic transformations",
    ],
    stack: ["Cryptography", "Secure Storage", "Cipher Design"],
    icon: FileKey,
  },
  {
    index: "03",
    title: "Local Gaming Platform",
    year: "2025",
    type: "Full-Stack Engineering",
    summary:
      "A locally hosted browser-based gaming platform built end to end with a Node.js backend and a responsive web interface.",
    outcomes: [
      "Local multiplayer web architecture",
      "Integrated frontend and backend",
      "Node.js service implementation",
    ],
    stack: ["Node.js", "Web Engineering", "Local Hosting"],
    icon: Blocks,
  },
] as const;

export const SKILL_GROUPS = [
  {
    title: "Offensive Operations",
    description: "Attack-surface discovery, exploitation validation, and adversarial testing.",
    icon: ScanSearch,
    skills: ["Burp Suite", "Metasploit", "Nmap", "OpenVAS", "Penetration Testing", "Red Teaming"],
  },
  {
    title: "Reverse & Binary",
    description: "Static and dynamic analysis of executables, packages, and low-level behavior.",
    icon: Binary,
    skills: ["Ghidra", "IDA Pro", "Binary Exploitation", "Package Analysis", "Linux Internals"],
  },
  {
    title: "Detection & Response",
    description: "Traffic inspection, event correlation, incident analysis, and operational monitoring.",
    icon: Radar,
    skills: ["Wireshark", "Splunk", "SIEM", "SOC Operations", "Incident Response", "Digital Forensics"],
  },
  {
    title: "Infrastructure",
    description: "Security work across operating systems, networks, web stacks, and identity environments.",
    icon: Network,
    skills: ["TCP/IP", "Active Directory", "Kali", "Ubuntu", "Arch", "Fedora", "Node.js"],
  },
] as const;

export const TOOL_NODES = [
  { label: "BURP", icon: Bug, orbit: 0 },
  { label: "NMAP", icon: Radar, orbit: 1 },
  { label: "GHIDRA", icon: Binary, orbit: 2 },
  { label: "WIRESHARK", icon: Cable, orbit: 3 },
  { label: "SPLUNK", icon: ServerCog, orbit: 4 },
  { label: "LINUX", icon: TerminalSquare, orbit: 5 },
  { label: "CRYPTO", icon: FileKey, orbit: 6 },
  { label: "OSINT", icon: Fingerprint, orbit: 7 },
  { label: "NODE", icon: Braces, orbit: 8 },
  { label: "ML", icon: Cpu, orbit: 9 },
] as const;

export const TIMELINE = [
  {
    period: "2026",
    title: "CRTA Modules Completed",
    subtitle: "CyberWarFare Labs",
    body: "Completed the Certified Red Team Analyst learning modules; certification exam remains pending.",
    icon: ShieldCheck,
  },
  {
    period: "2026",
    title: "npm Supply Chain Security FYP",
    subtitle: "COMSATS University Islamabad",
    body: "Developing an explainable analysis system for malicious npm package updates and real-time supply-chain risk assessment.",
    icon: Workflow,
  },
  {
    period: "2024 — Present",
    title: "CTF Technical & Organizing Team",
    subtitle: "Cyber-Hacktivators Club",
    body: "Co-organized a multi-university CTF for 50 teams, designed OSINT and forensic challenges, and administered the CTFd platform.",
    icon: TerminalSquare,
  },
  {
    period: "2024",
    title: "Certified Ethical Hacker",
    subtitle: "MetaPi Technologies",
    body: "Completed CEH training under reference MetaPi/24/NAVTTC/071.",
    icon: ShieldCheck,
  },
  {
    period: "2023 — Present",
    title: "Hands-on Security Practice",
    subtitle: "TryHackMe & Hack The Box",
    body: "Continuous practice across web exploitation, privilege escalation, forensics, penetration testing, binary exploitation, and OSINT.",
    icon: Bug,
  },
] as const;

export const METRICS = [
  { value: "3.45", label: "CGPA / 4.00" },
  { value: "50", label: "CTF teams hosted" },
  { value: "TOP 1–5%", label: "TryHackMe ranking" },
  { value: "2027", label: "Expected graduation" },
] as const;
```

## `lib/useMediaQuery.ts`

```ts
"use client";

import { useEffect, useState } from "react";

export default function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(false);

  useEffect(() => {
    const media = window.matchMedia(query);
    const update = () => setMatches(media.matches);

    update();
    media.addEventListener("change", update);

    return () => media.removeEventListener("change", update);
  }, [query]);

  return matches;
}
```

## `eslint.config.mjs`

```javascript
import { defineConfig, globalIgnores } from "eslint/config";
import nextVitals from "eslint-config-next/core-web-vitals";
import nextTs from "eslint-config-next/typescript";

export default defineConfig([
  ...nextVitals,
  ...nextTs,
  globalIgnores([".next/**", "out/**", "build/**", "next-env.d.ts"]),
]);
```

## `next-env.d.ts`

```ts
/// <reference types="next" />
/// <reference types="next/image-types/global" />
import "./.next/types/routes.d.ts";

// NOTE: This file should not be edited
// see https://nextjs.org/docs/app/api-reference/config/typescript for more information.
```

## `next.config.ts`

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  reactStrictMode: true,
  poweredByHeader: false,
};

export default nextConfig;
```

## `package.json`

```json
{
  "name": "iqbal-horizontal-cyber-portfolio",
  "version": "2.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint .",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "@fontsource-variable/jetbrains-mono": "5.3.0",
    "@fontsource-variable/space-grotesk": "5.3.0",
    "@react-three/drei": "10.7.7",
    "@react-three/fiber": "9.6.1",
    "lucide-react": "1.25.0",
    "motion": "12.42.2",
    "next": "16.2.10",
    "react": "19.2.7",
    "react-dom": "19.2.7",
    "three": "0.185.1"
  },
  "devDependencies": {
    "@types/node": "26.1.1",
    "@types/react": "19.2.17",
    "@types/react-dom": "19.2.3",
    "@types/three": "0.185.1",
    "eslint": "9.39.5",
    "eslint-config-next": "16.2.10",
    "typescript": "6.0.3"
  },
  "engines": {
    "node": ">=20.9.0"
  },
  "overrides": {
    "postcss": "8.5.10"
  }
}
```

## `package-lock.json`

```json
{
  "name": "iqbal-horizontal-cyber-portfolio",
  "version": "2.0.0",
  "lockfileVersion": 3,
  "requires": true,
  "packages": {
    "": {
      "name": "iqbal-horizontal-cyber-portfolio",
      "version": "2.0.0",
      "dependencies": {
        "@fontsource-variable/jetbrains-mono": "5.3.0",
        "@fontsource-variable/space-grotesk": "5.3.0",
        "@react-three/drei": "10.7.7",
        "@react-three/fiber": "9.6.1",
        "lucide-react": "1.25.0",
        "motion": "12.42.2",
        "next": "16.2.10",
        "react": "19.2.7",
        "react-dom": "19.2.7",
        "three": "0.185.1"
      },
      "devDependencies": {
        "@types/node": "26.1.1",
        "@types/react": "19.2.17",
        "@types/react-dom": "19.2.3",
        "@types/three": "0.185.1",
        "eslint": "9.39.5",
        "eslint-config-next": "16.2.10",
        "typescript": "6.0.3"
      },
      "engines": {
        "node": ">=20.9.0"
      }
    },
    "node_modules/@babel/code-frame": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/code-frame/-/code-frame-7.29.7.tgz",
      "integrity": "sha512-Aup7aUOfpbAUg2ROOJN6Iw5f9DMBlzu0mIkm/malLQFN/YQgO48wCj0Kxa3sEHJvPVFg7siR+qRInwXd2qhQKw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@babel/helper-validator-identifier": "^7.29.7",
        "js-tokens": "^4.0.0",
        "picocolors": "^1.1.1"
      },
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/@babel/compat-data": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/compat-data/-/compat-data-7.29.7.tgz",
      "integrity": "sha512-locTkQyKvwIEgBzVrn8693ebc97F2U8ZHjbXwDXJ5Fn2TCpNwTlKcaKLkdHop5c/icOFE7qt7Q9JC5hnKNa6Gg==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/@babel/core": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/core/-/core-7.29.7.tgz",
      "integrity": "sha512-RgHBCvtjbOK2gXSNBNIkNoEc9qoVEtau3hj8gEqKQuL3HZAibKarWFEI3Lfm6EYKkLalOh8eSrj9b+ch9H/VBA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@babel/code-frame": "^7.29.7",
        "@babel/generator": "^7.29.7",
        "@babel/helper-compilation-targets": "^7.29.7",
        "@babel/helper-module-transforms": "^7.29.7",
        "@babel/helpers": "^7.29.7",
        "@babel/parser": "^7.29.7",
        "@babel/template": "^7.29.7",
        "@babel/traverse": "^7.29.7",
        "@babel/types": "^7.29.7",
        "@jridgewell/remapping": "^2.3.5",
        "convert-source-map": "^2.0.0",
        "debug": "^4.1.0",
        "gensync": "^1.0.0-beta.2",
        "json5": "^2.2.3",
        "semver": "^6.3.1"
      },
      "engines": {
        "node": ">=6.9.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/babel"
      }
    },
    "node_modules/@babel/generator": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/generator/-/generator-7.29.7.tgz",
      "integrity": "sha512-DkXD5OJQaAQIdZ1bt3UZdEnHAn9Imd3IVBdX03UFe+ony9Ojw5pzr9YVKGDY1jt+Gcn/FnGkNf8r+Vj5NOJWtQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@babel/parser": "^7.29.7",
        "@babel/types": "^7.29.7",
        "@jridgewell/gen-mapping": "^0.3.12",
        "@jridgewell/trace-mapping": "^0.3.28",
        "jsesc": "^3.0.2"
      },
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/@babel/helper-compilation-targets": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/helper-compilation-targets/-/helper-compilation-targets-7.29.7.tgz",
      "integrity": "sha512-wem6WaBj4NaVYVdNhLPPVacES6ZJ+KBBfSkTMD3YZxbP3rm3Di85tJU5ljaUNhaOynt+Aj0xruhYuzQBt8n71g==",
      "dev": true,
      "dependencies": {
        "@babel/compat-data": "^7.29.7",
        "@babel/helper-validator-option": "^7.29.7",
        "browserslist": "^4.24.0",
        "lru-cache": "^5.1.1",
        "semver": "^6.3.1"
      },
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/@babel/helper-globals": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/helper-globals/-/helper-globals-7.29.7.tgz",
      "integrity": "sha512-3nQVUAtvkKH9zahfWgw96Jc/uFOmjACE1kQz82E2lqWmHBgjzbNlsC22nuQTfahmWeQtTq5nQ/4Nnd2A1wj4zA==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/@babel/helper-module-imports": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/helper-module-imports/-/helper-module-imports-7.29.7.tgz",
      "integrity": "sha512-ejHwrQQYcm9xnTivShn2IDOlIzInN34AXskvq9QicvCtEzq1Vzclu/tKF8Jq1Cg8JG2GL6/EmjgsCT7lXepE3g==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@babel/traverse": "^7.29.7",
        "@babel/types": "^7.29.7"
      },
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/@babel/helper-module-transforms": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/helper-module-transforms/-/helper-module-transforms-7.29.7.tgz",
      "integrity": "sha512-UPUVSyXbOh627KiCIGQSgwWzGeBKLkaJ9PJEdrngIwMSzxLR4jS4+f1f1jb7VzBbg8nFLaYotvVPFCTqdrmTAg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@babel/helper-module-imports": "^7.29.7",
        "@babel/helper-validator-identifier": "^7.29.7",
        "@babel/traverse": "^7.29.7"
      },
      "engines": {
        "node": ">=6.9.0"
      },
      "peerDependencies": {
        "@babel/core": "^7.0.0"
      }
    },
    "node_modules/@babel/helper-string-parser": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/helper-string-parser/-/helper-string-parser-7.29.7.tgz",
      "integrity": "sha512-Pb5ijPrZ89GDH8223L4UP8i6QApWxs04RbPQJTeWDV0/keR2E36MeKnyr6LYmUUvqRRI+Iv87SuF1W6ErINzYw==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/@babel/helper-validator-identifier": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/helper-validator-identifier/-/helper-validator-identifier-7.29.7.tgz",
      "integrity": "sha512-qehxGkRj55h/ff8EMaJ+cYhyaKlHIxqYDn682wQD7RNp9UujOQsHog2uS0r2vzr4pW+sXf90NeeayjcNaX3fFg==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/@babel/helper-validator-option": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/helper-validator-option/-/helper-validator-option-7.29.7.tgz",
      "integrity": "sha512-N9ZErrD+yW5geCDtBqnOoxmR8+tNKiGuxKlDpuJxfsqpa2dFcexaziGAE/qoHLiDDreVNMupxGmSoNlyvsA3gw==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/@babel/helpers": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/helpers/-/helpers-7.29.7.tgz",
      "integrity": "sha512-1k2lAGRMfHTcwuNYcCNUmaUffmQv8KWMfh2iJUUeRlwlwH4FdNG7mfPI10NPfLHJFThE4Tyr4mv7kTNZOiPuBg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@babel/template": "^7.29.7",
        "@babel/types": "^7.29.7"
      },
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/@babel/parser": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/parser/-/parser-7.29.7.tgz",
      "integrity": "sha512-hnORnjP/1P/zFEndoeX+n+t1RwWRJiJpM/jO7FW32Kn9r5+sJB2JWOdYo4L6k78j15eCwY3Gm/7364B1EMwtNg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@babel/types": "^7.29.7"
      },
      "bin": {
        "parser": "bin/babel-parser.js"
      },
      "engines": {
        "node": ">=6.0.0"
      }
    },
    "node_modules/@babel/runtime": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/runtime/-/runtime-7.29.7.tgz",
      "integrity": "sha512-Nq8OhGWiZIZGV6hLHoyAKLLcJihP/xFeBMGJoUrxTX2psI8dCifzLhZISFb+VWS3wFMRDmCGw5R+dOySCqPLhw==",
      "license": "MIT",
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/@babel/template": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/template/-/template-7.29.7.tgz",
      "integrity": "sha512-puq+Gf35oI24FeN11LkoUQFqv9uwNeWpxXZi/Ji3rRIoKAzKnxRaZ+Gkj0vKS9ZCiTESfng1N9LyOyXvo+m+Gg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@babel/code-frame": "^7.29.7",
        "@babel/parser": "^7.29.7",
        "@babel/types": "^7.29.7"
      },
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/@babel/traverse": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/traverse/-/traverse-7.29.7.tgz",
      "integrity": "sha512-EhlfNQtZ+NK22w5BM61ciuiq1m58ed33Wr1Xan//ZRTy6hgjnwyCffRYwzsGXdASJSUJ1guZILsErh1eQcl+zw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@babel/code-frame": "^7.29.7",
        "@babel/generator": "^7.29.7",
        "@babel/helper-globals": "^7.29.7",
        "@babel/parser": "^7.29.7",
        "@babel/template": "^7.29.7",
        "@babel/types": "^7.29.7",
        "debug": "^4.3.1"
      },
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/@babel/types": {
      "version": "7.29.7",
      "resolved": "https://registry.npmjs.org/@babel/types/-/types-7.29.7.tgz",
      "integrity": "sha512-4zBIxpPzowiZpusoFkyGVwakdRJUyuH5PxQ/PrqghfdFWWasvnCdPfQXHrenDai+gyLARulZjZowCOj6fjT4pA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@babel/helper-string-parser": "^7.29.7",
        "@babel/helper-validator-identifier": "^7.29.7"
      },
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/@dimforge/rapier3d-compat": {
      "version": "0.12.0",
      "resolved": "https://registry.npmjs.org/@dimforge/rapier3d-compat/-/rapier3d-compat-0.12.0.tgz",
      "integrity": "sha512-uekIGetywIgopfD97oDL5PfeezkFpNhwlzlaEYNOA0N6ghdsOvh/HYjSMek5Q2O1PYvRSDFcqFVJl4r4ZBwOow==",
      "license": "Apache-2.0"
    },
    "node_modules/@emnapi/core": {
      "version": "1.10.0",
      "resolved": "https://registry.npmjs.org/@emnapi/core/-/core-1.10.0.tgz",
      "integrity": "sha512-yq6OkJ4p82CAfPl0u9mQebQHKPJkY7WrIuk205cTYnYe+k2Z8YBh11FrbRG/H6ihirqcacOgl2BIO8oyMQLeXw==",
      "dev": true,
      "license": "MIT",
      "optional": true,
      "dependencies": {
        "@emnapi/wasi-threads": "1.2.1",
        "tslib": "^2.4.0"
      }
    },
    "node_modules/@emnapi/runtime": {
      "version": "1.11.2",
      "resolved": "https://registry.npmjs.org/@emnapi/runtime/-/runtime-1.11.2.tgz",
      "integrity": "sha512-kyOl3X0DuTiT1h2ft8r2fYO8JYtU9a9Xis/zBSiGArNaagCOWx90N1k2wxp18czFDH+OgcWGb5ZP/XMt3dcyPA==",
      "license": "MIT",
      "optional": true,
      "dependencies": {
        "tslib": "^2.4.0"
      }
    },
    "node_modules/@emnapi/wasi-threads": {
      "version": "1.2.1",
      "resolved": "https://registry.npmjs.org/@emnapi/wasi-threads/-/wasi-threads-1.2.1.tgz",
      "integrity": "sha512-uTII7OYF+/Mes/MrcIOYp5yOtSMLBWSIoLPpcgwipoiKbli6k322tcoFsxoIIxPDqW01SQGAgko4EzZi2BNv2w==",
      "dev": true,
      "license": "MIT",
      "optional": true,
      "dependencies": {
        "tslib": "^2.4.0"
      }
    },
    "node_modules/@eslint-community/eslint-utils": {
      "version": "4.9.1",
      "resolved": "https://registry.npmjs.org/@eslint-community/eslint-utils/-/eslint-utils-4.9.1.tgz",
      "integrity": "sha512-phrYmNiYppR7znFEdqgfWHXR6NCkZEK7hwWDHZUjit/2/U0r6XvkDl0SYnoM51Hq7FhCGdLDT6zxCCOY1hexsQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "eslint-visitor-keys": "^3.4.3"
      },
      "engines": {
        "node": "^12.22.0 || ^14.17.0 || >=16.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/eslint"
      },
      "peerDependencies": {
        "eslint": "^6.0.0 || ^7.0.0 || >=8.0.0"
      }
    },
    "node_modules/@eslint-community/eslint-utils/node_modules/eslint-visitor-keys": {
      "version": "3.4.3",
      "resolved": "https://registry.npmjs.org/eslint-visitor-keys/-/eslint-visitor-keys-3.4.3.tgz",
      "integrity": "sha512-wpc+LXeiyiisxPlEkUzU6svyS1frIO3Mgxj1fdy7Pm8Ygzguax2N3Fa/D/ag1WqbOprdI+uY6wMUl8/a2G+iag==",
      "dev": true,
      "license": "Apache-2.0",
      "engines": {
        "node": "^12.22.0 || ^14.17.0 || >=16.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/eslint"
      }
    },
    "node_modules/@eslint-community/regexpp": {
      "version": "4.12.2",
      "resolved": "https://registry.npmjs.org/@eslint-community/regexpp/-/regexpp-4.12.2.tgz",
      "integrity": "sha512-EriSTlt5OC9/7SXkRSCAhfSxxoSUgBm33OH+IkwbdpgoqsSsUg7y3uh+IICI/Qg4BBWr3U2i39RpmycbxMq4ew==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": "^12.0.0 || ^14.0.0 || >=16.0.0"
      }
    },
    "node_modules/@eslint/config-array": {
      "version": "0.21.2",
      "resolved": "https://registry.npmjs.org/@eslint/config-array/-/config-array-0.21.2.tgz",
      "integrity": "sha512-nJl2KGTlrf9GjLimgIru+V/mzgSK0ABCDQRvxw5BjURL7WfH5uoWmizbH7QB6MmnMBd8cIC9uceWnezL1VZWWw==",
      "dev": true,
      "license": "Apache-2.0",
      "dependencies": {
        "@eslint/object-schema": "^2.1.7",
        "debug": "^4.3.1",
        "minimatch": "^3.1.5"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      }
    },
    "node_modules/@eslint/config-helpers": {
      "version": "0.4.2",
      "resolved": "https://registry.npmjs.org/@eslint/config-helpers/-/config-helpers-0.4.2.tgz",
      "integrity": "sha512-gBrxN88gOIf3R7ja5K9slwNayVcZgK6SOUORm2uBzTeIEfeVaIhOpCtTox3P6R7o2jLFwLFTLnC7kU/RGcYEgw==",
      "dev": true,
      "dependencies": {
        "@eslint/core": "^0.17.0"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      }
    },
    "node_modules/@eslint/core": {
      "version": "0.17.0",
      "resolved": "https://registry.npmjs.org/@eslint/core/-/core-0.17.0.tgz",
      "integrity": "sha512-yL/sLrpmtDaFEiUj1osRP4TI2MDz1AddJL+jZ7KSqvBuliN4xqYY54IfdN8qD8Toa6g1iloph1fxQNkjOxrrpQ==",
      "dev": true,
      "license": "Apache-2.0",
      "dependencies": {
        "@types/json-schema": "^7.0.15"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      }
    },
    "node_modules/@eslint/eslintrc": {
      "version": "3.3.6",
      "resolved": "https://registry.npmjs.org/@eslint/eslintrc/-/eslintrc-3.3.6.tgz",
      "integrity": "sha512-l2Ul9PrHsPCKcEY/ac7VgFj9D80C7S68sOKc618SyHDPK36s1XcFebXY0iTzUVn4Yq+YbwvSnDmCz9yxjX+QrA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "ajv": "^6.14.0",
        "debug": "^4.3.2",
        "espree": "^10.0.1",
        "globals": "^14.0.0",
        "ignore": "^5.2.0",
        "import-fresh": "^3.2.1",
        "js-yaml": "^4.3.0",
        "minimatch": "^3.1.5",
        "strip-json-comments": "^3.1.1"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "url": "https://opencollective.com/eslint"
      }
    },
    "node_modules/@eslint/eslintrc/node_modules/globals": {
      "version": "14.0.0",
      "resolved": "https://registry.npmjs.org/globals/-/globals-14.0.0.tgz",
      "integrity": "sha512-oahGvuMGQlPw/ivIYBjVSrWAfWLBeku5tpPE2fOPLi+WHffIWbuh2tCjhyQhTBPMf5E9jDEH4FOmTYgYwbKwtQ==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=18"
      },
      "funding": {
        "url": "https://github.com/sponsors/sindresorhus"
      }
    },
    "node_modules/@eslint/js": {
      "version": "9.39.5",
      "resolved": "https://registry.npmjs.org/@eslint/js/-/js-9.39.5.tgz",
      "integrity": "sha512-QywQuszQh77pIXCsq998c8hbhSTI/azTty1Z6N53dmAudKHhy573j3yvRLsX2BSp8YpLtoCEG8E9DJe+8zUh4A==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "url": "https://eslint.org/donate"
      }
    },
    "node_modules/@eslint/object-schema": {
      "version": "2.1.7",
      "resolved": "https://registry.npmjs.org/@eslint/object-schema/-/object-schema-2.1.7.tgz",
      "integrity": "sha512-VtAOaymWVfZcmZbp6E2mympDIHvyjXs/12LqWYjVw6qjrfF+VK+fyG33kChz3nnK+SU5/NeHOqrTEHS8sXO3OA==",
      "dev": true,
      "license": "Apache-2.0",
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      }
    },
    "node_modules/@eslint/plugin-kit": {
      "version": "0.4.1",
      "resolved": "https://registry.npmjs.org/@eslint/plugin-kit/-/plugin-kit-0.4.1.tgz",
      "integrity": "sha512-43/qtrDUokr7LJqoF2c3+RInu/t4zfrpYdoSDfYyhg52rwLV6TnOvdG4fXm7IkSB3wErkcmJS9iEhjVtOSEjjA==",
      "dev": true,
      "license": "Apache-2.0",
      "dependencies": {
        "@eslint/core": "^0.17.0",
        "levn": "^0.4.1"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      }
    },
    "node_modules/@fontsource-variable/jetbrains-mono": {
      "version": "5.3.0",
      "resolved": "https://registry.npmjs.org/@fontsource-variable/jetbrains-mono/-/jetbrains-mono-5.3.0.tgz",
      "integrity": "sha512-F32xpS2NsGYoQi2ADSkKTgpJj7ozajsGgDJ8woTnqjmIB+dxDIqImjl4pXZVEExu8UFZ2ndhmX18EBS/hdz3Lw==",
      "license": "OFL-1.1",
      "funding": {
        "url": "https://github.com/sponsors/ayuhito"
      }
    },
    "node_modules/@fontsource-variable/space-grotesk": {
      "version": "5.3.0",
      "resolved": "https://registry.npmjs.org/@fontsource-variable/space-grotesk/-/space-grotesk-5.3.0.tgz",
      "integrity": "sha512-2IxmvfB08i9vnGB3Ym/AXvhRE+8XOjWMXIyDum03c+tPwH0FUoMNQfGpU8NXPxjbws0Vvss3AH0Zqt4oJBBAdw==",
      "license": "OFL-1.1",
      "funding": {
        "url": "https://github.com/sponsors/ayuhito"
      }
    },
    "node_modules/@humanfs/core": {
      "version": "0.19.2",
      "resolved": "https://registry.npmjs.org/@humanfs/core/-/core-0.19.2.tgz",
      "integrity": "sha512-UhXNm+CFMWcbChXywFwkmhqjs3PRCmcSa/hfBgLIb7oQ5HNb1wS0icWsGtSAUNgefHeI+eBrA8I1fxmbHsGdvA==",
      "dev": true,
      "license": "Apache-2.0",
      "dependencies": {
        "@humanfs/types": "^0.15.0"
      },
      "engines": {
        "node": ">=18.18.0"
      }
    },
    "node_modules/@humanfs/node": {
      "version": "0.16.8",
      "resolved": "https://registry.npmjs.org/@humanfs/node/-/node-0.16.8.tgz",
      "integrity": "sha512-gE1eQNZ3R++kTzFUpdGlpmy8kDZD/MLyHqDwqjkVQI0JMdI1D51sy1H958PNXYkM2rAac7e5/CnIKZrHtPh3BQ==",
      "dev": true,
      "license": "Apache-2.0",
      "dependencies": {
        "@humanfs/core": "^0.19.2",
        "@humanfs/types": "^0.15.0",
        "@humanwhocodes/retry": "^0.4.0"
      },
      "engines": {
        "node": ">=18.18.0"
      }
    },
    "node_modules/@humanfs/types": {
      "version": "0.15.0",
      "resolved": "https://registry.npmjs.org/@humanfs/types/-/types-0.15.0.tgz",
      "integrity": "sha512-ZZ1w0aoQkwuUuC7Yf+7sdeaNfqQiiLcSRbfI08oAxqLtpXQr9AIVX7Ay7HLDuiLYAaFPu8oBYNq/QIi9URHJ3Q==",
      "dev": true,
      "license": "Apache-2.0",
      "engines": {
        "node": ">=18.18.0"
      }
    },
    "node_modules/@humanwhocodes/module-importer": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/@humanwhocodes/module-importer/-/module-importer-1.0.1.tgz",
      "integrity": "sha512-bxveV4V8v5Yb4ncFTT3rPSgZBOpCkjfK0y4oVVVJwIuDVBRMDXrPyXRL988i5ap9m9bnyEEjWfm5WkBmtffLfA==",
      "dev": true,
      "license": "Apache-2.0",
      "engines": {
        "node": ">=12.22"
      },
      "funding": {
        "type": "github",
        "url": "https://github.com/sponsors/nzakas"
      }
    },
    "node_modules/@humanwhocodes/retry": {
      "version": "0.4.3",
      "resolved": "https://registry.npmjs.org/@humanwhocodes/retry/-/retry-0.4.3.tgz",
      "integrity": "sha512-bV0Tgo9K4hfPCek+aMAn81RppFKv2ySDQeMoSZuvTASywNTnVJCArCZE2FWqpvIatKu7VMRLWlR1EazvVhDyhQ==",
      "dev": true,
      "license": "Apache-2.0",
      "engines": {
        "node": ">=18.18"
      },
      "funding": {
        "type": "github",
        "url": "https://github.com/sponsors/nzakas"
      }
    },
    "node_modules/@img/colour": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/@img/colour/-/colour-1.1.0.tgz",
      "integrity": "sha512-Td76q7j57o/tLVdgS746cYARfSyxk8iEfRxewL9h4OMzYhbW4TAcppl0mT4eyqXddh6L/jwoM75mo7ixa/pCeQ==",
      "optional": true,
      "engines": {
        "node": ">=18"
      }
    },
    "node_modules/@img/sharp-darwin-arm64": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/@img/sharp-darwin-arm64/-/sharp-darwin-arm64-0.34.5.tgz",
      "integrity": "sha512-imtQ3WMJXbMY4fxb/Ndp6HBTNVtWCUI0WdobyheGf5+ad6xX8VIDO8u2xE4qc/fr08CKG/7dDseFtn6M6g/r3w==",
      "cpu": [
        "arm64"
      ],
      "license": "Apache-2.0",
      "optional": true,
      "os": [
        "darwin"
      ],
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      },
      "optionalDependencies": {
        "@img/sharp-libvips-darwin-arm64": "1.2.4"
      }
    },
    "node_modules/@img/sharp-darwin-x64": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/@img/sharp-darwin-x64/-/sharp-darwin-x64-0.34.5.tgz",
      "integrity": "sha512-YNEFAF/4KQ/PeW0N+r+aVVsoIY0/qxxikF2SWdp+NRkmMB7y9LBZAVqQ4yhGCm/H3H270OSykqmQMKLBhBJDEw==",
      "cpu": [
        "x64"
      ],
      "license": "Apache-2.0",
      "optional": true,
      "os": [
        "darwin"
      ],
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      },
      "optionalDependencies": {
        "@img/sharp-libvips-darwin-x64": "1.2.4"
      }
    },
    "node_modules/@img/sharp-libvips-darwin-arm64": {
      "version": "1.2.4",
      "resolved": "https://registry.npmjs.org/@img/sharp-libvips-darwin-arm64/-/sharp-libvips-darwin-arm64-1.2.4.tgz",
      "integrity": "sha512-zqjjo7RatFfFoP0MkQ51jfuFZBnVE2pRiaydKJ1G/rHZvnsrHAOcQALIi9sA5co5xenQdTugCvtb1cuf78Vf4g==",
      "cpu": [
        "arm64"
      ],
      "license": "LGPL-3.0-or-later",
      "optional": true,
      "os": [
        "darwin"
      ],
      "funding": {
        "url": "https://opencollective.com/libvips"
      }
    },
    "node_modules/@img/sharp-libvips-darwin-x64": {
      "version": "1.2.4",
      "resolved": "https://registry.npmjs.org/@img/sharp-libvips-darwin-x64/-/sharp-libvips-darwin-x64-1.2.4.tgz",
      "integrity": "sha512-1IOd5xfVhlGwX+zXv2N93k0yMONvUlANylbJw1eTah8K/Jtpi15KC+WSiaX/nBmbm2HxRM1gZ0nSdjSsrZbGKg==",
      "cpu": [
        "x64"
      ],
      "license": "LGPL-3.0-or-later",
      "optional": true,
      "os": [
        "darwin"
      ],
      "funding": {
        "url": "https://opencollective.com/libvips"
      }
    },
    "node_modules/@img/sharp-libvips-linux-arm": {
      "version": "1.2.4",
      "resolved": "https://registry.npmjs.org/@img/sharp-libvips-linux-arm/-/sharp-libvips-linux-arm-1.2.4.tgz",
      "integrity": "sha512-bFI7xcKFELdiNCVov8e44Ia4u2byA+l3XtsAj+Q8tfCwO6BQ8iDojYdvoPMqsKDkuoOo+X6HZA0s0q11ANMQ8A==",
      "cpu": [
        "arm"
      ],
      "license": "LGPL-3.0-or-later",
      "optional": true,
      "os": [
        "linux"
      ],
      "funding": {
        "url": "https://opencollective.com/libvips"
      }
    },
    "node_modules/@img/sharp-libvips-linux-arm64": {
      "version": "1.2.4",
      "resolved": "https://registry.npmjs.org/@img/sharp-libvips-linux-arm64/-/sharp-libvips-linux-arm64-1.2.4.tgz",
      "integrity": "sha512-excjX8DfsIcJ10x1Kzr4RcWe1edC9PquDRRPx3YVCvQv+U5p7Yin2s32ftzikXojb1PIFc/9Mt28/y+iRklkrw==",
      "cpu": [
        "arm64"
      ],
      "license": "LGPL-3.0-or-later",
      "optional": true,
      "os": [
        "linux"
      ],
      "funding": {
        "url": "https://opencollective.com/libvips"
      }
    },
    "node_modules/@img/sharp-libvips-linux-ppc64": {
      "version": "1.2.4",
      "resolved": "https://registry.npmjs.org/@img/sharp-libvips-linux-ppc64/-/sharp-libvips-linux-ppc64-1.2.4.tgz",
      "integrity": "sha512-FMuvGijLDYG6lW+b/UvyilUWu5Ayu+3r2d1S8notiGCIyYU/76eig1UfMmkZ7vwgOrzKzlQbFSuQfgm7GYUPpA==",
      "cpu": [
        "ppc64"
      ],
      "license": "LGPL-3.0-or-later",
      "optional": true,
      "os": [
        "linux"
      ],
      "funding": {
        "url": "https://opencollective.com/libvips"
      }
    },
    "node_modules/@img/sharp-libvips-linux-riscv64": {
      "version": "1.2.4",
      "resolved": "https://registry.npmjs.org/@img/sharp-libvips-linux-riscv64/-/sharp-libvips-linux-riscv64-1.2.4.tgz",
      "integrity": "sha512-oVDbcR4zUC0ce82teubSm+x6ETixtKZBh/qbREIOcI3cULzDyb18Sr/Wcyx7NRQeQzOiHTNbZFF1UwPS2scyGA==",
      "cpu": [
        "riscv64"
      ],
      "license": "LGPL-3.0-or-later",
      "optional": true,
      "os": [
        "linux"
      ],
      "funding": {
        "url": "https://opencollective.com/libvips"
      }
    },
    "node_modules/@img/sharp-libvips-linux-s390x": {
      "version": "1.2.4",
      "resolved": "https://registry.npmjs.org/@img/sharp-libvips-linux-s390x/-/sharp-libvips-linux-s390x-1.2.4.tgz",
      "integrity": "sha512-qmp9VrzgPgMoGZyPvrQHqk02uyjA0/QrTO26Tqk6l4ZV0MPWIW6LTkqOIov+J1yEu7MbFQaDpwdwJKhbJvuRxQ==",
      "cpu": [
        "s390x"
      ],
      "license": "LGPL-3.0-or-later",
      "optional": true,
      "os": [
        "linux"
      ],
      "funding": {
        "url": "https://opencollective.com/libvips"
      }
    },
    "node_modules/@img/sharp-libvips-linux-x64": {
      "version": "1.2.4",
      "resolved": "https://registry.npmjs.org/@img/sharp-libvips-linux-x64/-/sharp-libvips-linux-x64-1.2.4.tgz",
      "integrity": "sha512-tJxiiLsmHc9Ax1bz3oaOYBURTXGIRDODBqhveVHonrHJ9/+k89qbLl0bcJns+e4t4rvaNBxaEZsFtSfAdquPrw==",
      "cpu": [
        "x64"
      ],
      "optional": true,
      "os": [
        "linux"
      ],
      "funding": {
        "url": "https://opencollective.com/libvips"
      }
    },
    "node_modules/@img/sharp-libvips-linuxmusl-arm64": {
      "version": "1.2.4",
      "resolved": "https://registry.npmjs.org/@img/sharp-libvips-linuxmusl-arm64/-/sharp-libvips-linuxmusl-arm64-1.2.4.tgz",
      "integrity": "sha512-FVQHuwx1IIuNow9QAbYUzJ+En8KcVm9Lk5+uGUQJHaZmMECZmOlix9HnH7n1TRkXMS0pGxIJokIVB9SuqZGGXw==",
      "cpu": [
        "arm64"
      ],
      "license": "LGPL-3.0-or-later",
      "optional": true,
      "os": [
        "linux"
      ],
      "funding": {
        "url": "https://opencollective.com/libvips"
      }
    },
    "node_modules/@img/sharp-libvips-linuxmusl-x64": {
      "version": "1.2.4",
      "resolved": "https://registry.npmjs.org/@img/sharp-libvips-linuxmusl-x64/-/sharp-libvips-linuxmusl-x64-1.2.4.tgz",
      "integrity": "sha512-+LpyBk7L44ZIXwz/VYfglaX/okxezESc6UxDSoyo2Ks6Jxc4Y7sGjpgU9s4PMgqgjj1gZCylTieNamqA1MF7Dg==",
      "cpu": [
        "x64"
      ],
      "license": "LGPL-3.0-or-later",
      "optional": true,
      "os": [
        "linux"
      ],
      "funding": {
        "url": "https://opencollective.com/libvips"
      }
    },
    "node_modules/@img/sharp-linux-arm": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/@img/sharp-linux-arm/-/sharp-linux-arm-0.34.5.tgz",
      "integrity": "sha512-9dLqsvwtg1uuXBGZKsxem9595+ujv0sJ6Vi8wcTANSFpwV/GONat5eCkzQo/1O6zRIkh0m/8+5BjrRr7jDUSZw==",
      "cpu": [
        "arm"
      ],
      "license": "Apache-2.0",
      "optional": true,
      "os": [
        "linux"
      ],
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      },
      "optionalDependencies": {
        "@img/sharp-libvips-linux-arm": "1.2.4"
      }
    },
    "node_modules/@img/sharp-linux-arm64": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/@img/sharp-linux-arm64/-/sharp-linux-arm64-0.34.5.tgz",
      "integrity": "sha512-bKQzaJRY/bkPOXyKx5EVup7qkaojECG6NLYswgktOZjaXecSAeCWiZwwiFf3/Y+O1HrauiE3FVsGxFg8c24rZg==",
      "cpu": [
        "arm64"
      ],
      "license": "Apache-2.0",
      "optional": true,
      "os": [
        "linux"
      ],
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      },
      "optionalDependencies": {
        "@img/sharp-libvips-linux-arm64": "1.2.4"
      }
    },
    "node_modules/@img/sharp-linux-ppc64": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/@img/sharp-linux-ppc64/-/sharp-linux-ppc64-0.34.5.tgz",
      "integrity": "sha512-7zznwNaqW6YtsfrGGDA6BRkISKAAE1Jo0QdpNYXNMHu2+0dTrPflTLNkpc8l7MUP5M16ZJcUvysVWWrMefZquA==",
      "cpu": [
        "ppc64"
      ],
      "license": "Apache-2.0",
      "optional": true,
      "os": [
        "linux"
      ],
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      },
      "optionalDependencies": {
        "@img/sharp-libvips-linux-ppc64": "1.2.4"
      }
    },
    "node_modules/@img/sharp-linux-riscv64": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/@img/sharp-linux-riscv64/-/sharp-linux-riscv64-0.34.5.tgz",
      "integrity": "sha512-51gJuLPTKa7piYPaVs8GmByo7/U7/7TZOq+cnXJIHZKavIRHAP77e3N2HEl3dgiqdD/w0yUfiJnII77PuDDFdw==",
      "cpu": [
        "riscv64"
      ],
      "license": "Apache-2.0",
      "optional": true,
      "os": [
        "linux"
      ],
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      },
      "optionalDependencies": {
        "@img/sharp-libvips-linux-riscv64": "1.2.4"
      }
    },
    "node_modules/@img/sharp-linux-s390x": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/@img/sharp-linux-s390x/-/sharp-linux-s390x-0.34.5.tgz",
      "integrity": "sha512-nQtCk0PdKfho3eC5MrbQoigJ2gd1CgddUMkabUj+rBevs8tZ2cULOx46E7oyX+04WGfABgIwmMC0VqieTiR4jg==",
      "cpu": [
        "s390x"
      ],
      "license": "Apache-2.0",
      "optional": true,
      "os": [
        "linux"
      ],
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      },
      "optionalDependencies": {
        "@img/sharp-libvips-linux-s390x": "1.2.4"
      }
    },
    "node_modules/@img/sharp-linux-x64": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/@img/sharp-linux-x64/-/sharp-linux-x64-0.34.5.tgz",
      "integrity": "sha512-MEzd8HPKxVxVenwAa+JRPwEC7QFjoPWuS5NZnBt6B3pu7EG2Ge0id1oLHZpPJdn3OQK+BQDiw9zStiHBTJQQQQ==",
      "cpu": [
        "x64"
      ],
      "license": "Apache-2.0",
      "optional": true,
      "os": [
        "linux"
      ],
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      },
      "optionalDependencies": {
        "@img/sharp-libvips-linux-x64": "1.2.4"
      }
    },
    "node_modules/@img/sharp-linuxmusl-arm64": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/@img/sharp-linuxmusl-arm64/-/sharp-linuxmusl-arm64-0.34.5.tgz",
      "integrity": "sha512-fprJR6GtRsMt6Kyfq44IsChVZeGN97gTD331weR1ex1c1rypDEABN6Tm2xa1wE6lYb5DdEnk03NZPqA7Id21yg==",
      "cpu": [
        "arm64"
      ],
      "license": "Apache-2.0",
      "optional": true,
      "os": [
        "linux"
      ],
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      },
      "optionalDependencies": {
        "@img/sharp-libvips-linuxmusl-arm64": "1.2.4"
      }
    },
    "node_modules/@img/sharp-linuxmusl-x64": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/@img/sharp-linuxmusl-x64/-/sharp-linuxmusl-x64-0.34.5.tgz",
      "integrity": "sha512-Jg8wNT1MUzIvhBFxViqrEhWDGzqymo3sV7z7ZsaWbZNDLXRJZoRGrjulp60YYtV4wfY8VIKcWidjojlLcWrd8Q==",
      "cpu": [
        "x64"
      ],
      "license": "Apache-2.0",
      "optional": true,
      "os": [
        "linux"
      ],
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      },
      "optionalDependencies": {
        "@img/sharp-libvips-linuxmusl-x64": "1.2.4"
      }
    },
    "node_modules/@img/sharp-wasm32": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/@img/sharp-wasm32/-/sharp-wasm32-0.34.5.tgz",
      "integrity": "sha512-OdWTEiVkY2PHwqkbBI8frFxQQFekHaSSkUIJkwzclWZe64O1X4UlUjqqqLaPbUpMOQk6FBu/HtlGXNblIs0huw==",
      "cpu": [
        "wasm32"
      ],
      "license": "Apache-2.0 AND LGPL-3.0-or-later AND MIT",
      "optional": true,
      "dependencies": {
        "@emnapi/runtime": "^1.7.0"
      },
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      }
    },
    "node_modules/@img/sharp-win32-arm64": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/@img/sharp-win32-arm64/-/sharp-win32-arm64-0.34.5.tgz",
      "integrity": "sha512-WQ3AgWCWYSb2yt+IG8mnC6Jdk9Whs7O0gxphblsLvdhSpSTtmu69ZG1Gkb6NuvxsNACwiPV6cNSZNzt0KPsw7g==",
      "cpu": [
        "arm64"
      ],
      "license": "Apache-2.0 AND LGPL-3.0-or-later",
      "optional": true,
      "os": [
        "win32"
      ],
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      }
    },
    "node_modules/@img/sharp-win32-ia32": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/@img/sharp-win32-ia32/-/sharp-win32-ia32-0.34.5.tgz",
      "integrity": "sha512-FV9m/7NmeCmSHDD5j4+4pNI8Cp3aW+JvLoXcTUo0IqyjSfAZJ8dIUmijx1qaJsIiU+Hosw6xM5KijAWRJCSgNg==",
      "cpu": [
        "ia32"
      ],
      "license": "Apache-2.0 AND LGPL-3.0-or-later",
      "optional": true,
      "os": [
        "win32"
      ],
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      }
    },
    "node_modules/@img/sharp-win32-x64": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/@img/sharp-win32-x64/-/sharp-win32-x64-0.34.5.tgz",
      "integrity": "sha512-+29YMsqY2/9eFEiW93eqWnuLcWcufowXewwSNIT6UwZdUUCrM3oFjMWH/Z6/TMmb4hlFenmfAVbpWeup2jryCw==",
      "cpu": [
        "x64"
      ],
      "license": "Apache-2.0 AND LGPL-3.0-or-later",
      "optional": true,
      "os": [
        "win32"
      ],
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      }
    },
    "node_modules/@jridgewell/gen-mapping": {
      "version": "0.3.13",
      "resolved": "https://registry.npmjs.org/@jridgewell/gen-mapping/-/gen-mapping-0.3.13.tgz",
      "integrity": "sha512-2kkt/7niJ6MgEPxF0bYdQ6etZaA+fQvDcLKckhy1yIQOzaoKjBBjSj63/aLVjYE3qhRt5dvM+uUyfCg6UKCBbA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@jridgewell/sourcemap-codec": "^1.5.0",
        "@jridgewell/trace-mapping": "^0.3.24"
      }
    },
    "node_modules/@jridgewell/remapping": {
      "version": "2.3.5",
      "resolved": "https://registry.npmjs.org/@jridgewell/remapping/-/remapping-2.3.5.tgz",
      "integrity": "sha512-LI9u/+laYG4Ds1TDKSJW2YPrIlcVYOwi2fUC6xB43lueCjgxV4lffOCZCtYFiH6TNOX+tQKXx97T4IKHbhyHEQ==",
      "dev": true,
      "dependencies": {
        "@jridgewell/gen-mapping": "^0.3.5",
        "@jridgewell/trace-mapping": "^0.3.24"
      }
    },
    "node_modules/@jridgewell/resolve-uri": {
      "version": "3.1.2",
      "resolved": "https://registry.npmjs.org/@jridgewell/resolve-uri/-/resolve-uri-3.1.2.tgz",
      "integrity": "sha512-bRISgCIjP20/tbWSPWMEi54QVPRZExkuD9lJL+UIxUKtwVJA8wW1Trb1jMs1RFXo1CBTNZ/5hpC9QvmKWdopKw==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=6.0.0"
      }
    },
    "node_modules/@jridgewell/sourcemap-codec": {
      "version": "1.5.5",
      "resolved": "https://registry.npmjs.org/@jridgewell/sourcemap-codec/-/sourcemap-codec-1.5.5.tgz",
      "integrity": "sha512-cYQ9310grqxueWbl+WuIUIaiUaDcj7WOq5fVhEljNVgRfOUhY9fy2zTvfoqWsnebh8Sl70VScFbICvJnLKB0Og==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/@jridgewell/trace-mapping": {
      "version": "0.3.31",
      "resolved": "https://registry.npmjs.org/@jridgewell/trace-mapping/-/trace-mapping-0.3.31.tgz",
      "integrity": "sha512-zzNR+SdQSDJzc8joaeP8QQoCQr8NuYx2dIIytl1QeBEZHJ9uW6hebsrYgbz8hJwUQao3TWCMtmfV8Nu1twOLAw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@jridgewell/resolve-uri": "^3.1.0",
        "@jridgewell/sourcemap-codec": "^1.4.14"
      }
    },
    "node_modules/@mediapipe/tasks-vision": {
      "version": "0.10.17",
      "resolved": "https://registry.npmjs.org/@mediapipe/tasks-vision/-/tasks-vision-0.10.17.tgz",
      "integrity": "sha512-CZWV/q6TTe8ta61cZXjfnnHsfWIdFhms03M9T7Cnd5y2mdpylJM0rF1qRq+wsQVRMLz1OYPVEBU9ph2Bx8cxrg=="
    },
    "node_modules/@monogrid/gainmap-js": {
      "version": "3.4.0",
      "resolved": "https://registry.npmjs.org/@monogrid/gainmap-js/-/gainmap-js-3.4.0.tgz",
      "integrity": "sha512-2Z0FATFHaoYJ8b+Y4y4Hgfn3FRFwuU5zRrk+9dFWp4uGAdHGqVEdP7HP+gLA3X469KXHmfupJaUbKo1b/aDKIg==",
      "license": "MIT",
      "dependencies": {
        "promise-worker-transferable": "^1.0.4"
      },
      "peerDependencies": {
        "three": ">= 0.159.0"
      }
    },
    "node_modules/@napi-rs/wasm-runtime": {
      "version": "1.1.6",
      "resolved": "https://registry.npmjs.org/@napi-rs/wasm-runtime/-/wasm-runtime-1.1.6.tgz",
      "integrity": "sha512-ZLv/JdUfkvOy9eCnnBaGfiO+XimbjebAeO+MRQqD/B+FR1tnRN0tpKSJHRbE8sFfS6aqsXZ67TQjfwfsxULVbg==",
      "dev": true,
      "license": "MIT",
      "optional": true,
      "dependencies": {
        "@tybys/wasm-util": "^0.10.3"
      },
      "funding": {
        "type": "github",
        "url": "https://github.com/sponsors/Brooooooklyn"
      },
      "peerDependencies": {
        "@emnapi/core": "^1.7.1",
        "@emnapi/runtime": "^1.7.1"
      }
    },
    "node_modules/@next/env": {
      "version": "16.2.10",
      "resolved": "https://registry.npmjs.org/@next/env/-/env-16.2.10.tgz",
      "integrity": "sha512-zLPxg9M0MEHmygpj5OuxjQ+vHMiy/K7cSp74G8ecYolmgUWw0RwN02tF56npup/+qaI8JB97hQgS/r2Hb6QwVA==",
      "license": "MIT"
    },
    "node_modules/@next/eslint-plugin-next": {
      "version": "16.2.10",
      "resolved": "https://registry.npmjs.org/@next/eslint-plugin-next/-/eslint-plugin-next-16.2.10.tgz",
      "integrity": "sha512-Gs8D2m21VnJeFo9qvYIIqJH94frWerWYu41BprU1pLtRVF7PCQNLiFZZ3fG+iPuj3K83Cwv/rt+msLOy8Qgu3Q==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "fast-glob": "3.3.1"
      }
    },
    "node_modules/@next/swc-darwin-arm64": {
      "version": "16.2.10",
      "resolved": "https://registry.npmjs.org/@next/swc-darwin-arm64/-/swc-darwin-arm64-16.2.10.tgz",
      "integrity": "sha512-v9IdJCa0H0mbo+8z5zwUpOk1Vj7RjkcI5uNYf5Ws1y6szf/p3Mzl9hLaST8SCt6L9h8NGnruZcd2+o0NTNwDhA==",
      "cpu": [
        "arm64"
      ],
      "license": "MIT",
      "optional": true,
      "os": [
        "darwin"
      ],
      "engines": {
        "node": ">= 10"
      }
    },
    "node_modules/@next/swc-darwin-x64": {
      "version": "16.2.10",
      "resolved": "https://registry.npmjs.org/@next/swc-darwin-x64/-/swc-darwin-x64-16.2.10.tgz",
      "integrity": "sha512-17IS0jJRViROGmA9uGdNR8VPJpfbnaVG7E9qhso5jDLkmyd0lSDORWxbcKINzcFqzZqGwGtMSnrFRxBpuUYjLQ==",
      "cpu": [
        "x64"
      ],
      "license": "MIT",
      "optional": true,
      "os": [
        "darwin"
      ],
      "engines": {
        "node": ">= 10"
      }
    },
    "node_modules/@next/swc-linux-arm64-gnu": {
      "version": "16.2.10",
      "resolved": "https://registry.npmjs.org/@next/swc-linux-arm64-gnu/-/swc-linux-arm64-gnu-16.2.10.tgz",
      "integrity": "sha512-GRQRsRtuciNJvB54AvvuQTiq0oZtFwa1owQqtZD8wwnGpM2L39MV22kpI72YSXLKIyY40LC66EiLFv4PiicXxg==",
      "cpu": [
        "arm64"
      ],
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ],
      "engines": {
        "node": ">= 10"
      }
    },
    "node_modules/@next/swc-linux-arm64-musl": {
      "version": "16.2.10",
      "resolved": "https://registry.npmjs.org/@next/swc-linux-arm64-musl/-/swc-linux-arm64-musl-16.2.10.tgz",
      "integrity": "sha512-zkN9MQYS7UQBro+FnISUq1itaQjXI9xqISzuQ+2bc921NcJ1x4yPCqrn77tVN6/dOOXaaWVX3k6/bR07pPwK+A==",
      "cpu": [
        "arm64"
      ],
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ],
      "engines": {
        "node": ">= 10"
      }
    },
    "node_modules/@next/swc-linux-x64-gnu": {
      "version": "16.2.10",
      "resolved": "https://registry.npmjs.org/@next/swc-linux-x64-gnu/-/swc-linux-x64-gnu-16.2.10.tgz",
      "integrity": "sha512-iCVJnwvrPYECvA6WM/7+oo+OiTvedIKLxtCLAZP4xZR3nXa1zmzZyLPbYCmWvpd4CvMYF1EMTafd0ii3DygLvA==",
      "cpu": [
        "x64"
      ],
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ],
      "engines": {
        "node": ">= 10"
      }
    },
    "node_modules/@next/swc-linux-x64-musl": {
      "version": "16.2.10",
      "resolved": "https://registry.npmjs.org/@next/swc-linux-x64-musl/-/swc-linux-x64-musl-16.2.10.tgz",
      "integrity": "sha512-ov2g4H0dHY9bPoOU83m91hWT7Iq5qy13bUnyyshLU3HGR1Ownn0X9QpmDPc5iIUaahTp7f7LeGAhV4DSFtackw==",
      "cpu": [
        "x64"
      ],
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ],
      "engines": {
        "node": ">= 10"
      }
    },
    "node_modules/@next/swc-win32-arm64-msvc": {
      "version": "16.2.10",
      "resolved": "https://registry.npmjs.org/@next/swc-win32-arm64-msvc/-/swc-win32-arm64-msvc-16.2.10.tgz",
      "integrity": "sha512-DwAnhLX76HQiFFQNgWlcK+JzlnD1rZ+UK/WY0ZMI/deXpvgnesjNYrqcfo1JzBuz4Kf7o3brIBL0glI1junatA==",
      "cpu": [
        "arm64"
      ],
      "license": "MIT",
      "optional": true,
      "os": [
        "win32"
      ],
      "engines": {
        "node": ">= 10"
      }
    },
    "node_modules/@next/swc-win32-x64-msvc": {
      "version": "16.2.10",
      "resolved": "https://registry.npmjs.org/@next/swc-win32-x64-msvc/-/swc-win32-x64-msvc-16.2.10.tgz",
      "integrity": "sha512-0JXq3b85Jk9Jg4ntLUbXSPvoDw3gpZou7twuKdoFG2jOw635v7+IiXfTaa0TxVMyx78pUjnrVYwLgjKfX4e6/A==",
      "cpu": [
        "x64"
      ],
      "license": "MIT",
      "optional": true,
      "os": [
        "win32"
      ],
      "engines": {
        "node": ">= 10"
      }
    },
    "node_modules/@nodelib/fs.scandir": {
      "version": "2.1.5",
      "resolved": "https://registry.npmjs.org/@nodelib/fs.scandir/-/fs.scandir-2.1.5.tgz",
      "integrity": "sha512-vq24Bq3ym5HEQm2NKCr3yXDwjc7vTsEThRDnkp2DK9p1uqLR+DHurm/NOTo0KG7HYHU7eppKZj3MyqYuMBf62g==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@nodelib/fs.stat": "2.0.5",
        "run-parallel": "^1.1.9"
      },
      "engines": {
        "node": ">= 8"
      }
    },
    "node_modules/@nodelib/fs.stat": {
      "version": "2.0.5",
      "resolved": "https://registry.npmjs.org/@nodelib/fs.stat/-/fs.stat-2.0.5.tgz",
      "integrity": "sha512-RkhPPp2zrqDAQA/2jNhnztcPAlv64XdhIp7a7454A5ovI7Bukxgt7MX7udwAu3zg1DcpPU0rz3VV1SeaqvY4+A==",
      "dev": true,
      "engines": {
        "node": ">= 8"
      }
    },
    "node_modules/@nodelib/fs.walk": {
      "version": "1.2.8",
      "resolved": "https://registry.npmjs.org/@nodelib/fs.walk/-/fs.walk-1.2.8.tgz",
      "integrity": "sha512-oGB+UxlgWcgQkgwo8GcEGwemoTFt3FIO9ababBmaGwXIoBKZ+GTy0pP185beGg7Llih/NSHSV2XAs1lnznocSg==",
      "dev": true,
      "dependencies": {
        "@nodelib/fs.scandir": "2.1.5",
        "fastq": "^1.6.0"
      },
      "engines": {
        "node": ">= 8"
      }
    },
    "node_modules/@nolyfill/is-core-module": {
      "version": "1.0.39",
      "resolved": "https://registry.npmjs.org/@nolyfill/is-core-module/-/is-core-module-1.0.39.tgz",
      "integrity": "sha512-nn5ozdjYQpUCZlWGuxcJY/KpxkWQs4DcbMCmKojjyrYDEAGy4Ce19NN4v5MduafTwJlbKc99UA8YhSVqq9yPZA==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=12.4.0"
      }
    },
    "node_modules/@react-three/drei": {
      "version": "10.7.7",
      "resolved": "https://registry.npmjs.org/@react-three/drei/-/drei-10.7.7.tgz",
      "integrity": "sha512-ff+J5iloR0k4tC++QtD/j9u3w5fzfgFAWDtAGQah9pF2B1YgOq/5JxqY0/aVoQG5r3xSZz0cv5tk2YuBob4xEQ==",
      "license": "MIT",
      "dependencies": {
        "@babel/runtime": "^7.26.0",
        "@mediapipe/tasks-vision": "0.10.17",
        "@monogrid/gainmap-js": "^3.0.6",
        "@use-gesture/react": "^10.3.1",
        "camera-controls": "^3.1.0",
        "cross-env": "^7.0.3",
        "detect-gpu": "^5.0.56",
        "glsl-noise": "^0.0.0",
        "hls.js": "^1.5.17",
        "maath": "^0.10.8",
        "meshline": "^3.3.1",
        "stats-gl": "^2.2.8",
        "stats.js": "^0.17.0",
        "suspend-react": "^0.1.3",
        "three-mesh-bvh": "^0.8.3",
        "three-stdlib": "^2.35.6",
        "troika-three-text": "^0.52.4",
        "tunnel-rat": "^0.1.2",
        "use-sync-external-store": "^1.4.0",
        "utility-types": "^3.11.0",
        "zustand": "^5.0.1"
      },
      "peerDependencies": {
        "@react-three/fiber": "^9.0.0",
        "react": "^19",
        "react-dom": "^19",
        "three": ">=0.159"
      },
      "peerDependenciesMeta": {
        "react-dom": {
          "optional": true
        }
      }
    },
    "node_modules/@react-three/fiber": {
      "version": "9.6.1",
      "resolved": "https://registry.npmjs.org/@react-three/fiber/-/fiber-9.6.1.tgz",
      "integrity": "sha512-zF0rsKcVYpcJwbFEnv2HkHX9cvOEgsfQo/X8lwmR2dn13S4qEQJXir9fxf5js2LQFoXqxOY7MDkOkYx2uZ4gSg==",
      "license": "MIT",
      "dependencies": {
        "@babel/runtime": "^7.17.8",
        "@types/webxr": "*",
        "base64-js": "^1.5.1",
        "buffer": "^6.0.3",
        "its-fine": "^2.0.0",
        "react-use-measure": "^2.1.7",
        "scheduler": "^0.27.0",
        "suspend-react": "^0.1.3",
        "use-sync-external-store": "^1.4.0",
        "zustand": "^5.0.3"
      },
      "peerDependencies": {
        "expo": ">=43.0",
        "expo-asset": ">=8.4",
        "expo-file-system": ">=11.0",
        "expo-gl": ">=11.0",
        "react": ">=19 <19.3",
        "react-dom": ">=19 <19.3",
        "react-native": ">=0.78",
        "three": ">=0.156"
      },
      "peerDependenciesMeta": {
        "expo": {
          "optional": true
        },
        "expo-asset": {
          "optional": true
        },
        "expo-file-system": {
          "optional": true
        },
        "expo-gl": {
          "optional": true
        },
        "react-dom": {
          "optional": true
        },
        "react-native": {
          "optional": true
        }
      }
    },
    "node_modules/@rtsao/scc": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/@rtsao/scc/-/scc-1.1.0.tgz",
      "integrity": "sha512-zt6OdqaDoOnJ1ZYsCYGt9YmWzDXl4vQdKTyJev62gFhRGKdx7mcT54V9KIjg+d2wi9EXsPvAPKe7i7WjfVWB8g==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/@swc/helpers": {
      "version": "0.5.15",
      "resolved": "https://registry.npmjs.org/@swc/helpers/-/helpers-0.5.15.tgz",
      "integrity": "sha512-JQ5TuMi45Owi4/BIMAJBoSQoOJu12oOk/gADqlcUL9JEdHB8vyjUSsxqeNXnmXHjYKMi2WcYtezGEEhqUI/E2g==",
      "license": "Apache-2.0",
      "dependencies": {
        "tslib": "^2.8.0"
      }
    },
    "node_modules/@tweenjs/tween.js": {
      "version": "23.1.3",
      "resolved": "https://registry.npmjs.org/@tweenjs/tween.js/-/tween.js-23.1.3.tgz",
      "integrity": "sha512-vJmvvwFxYuGnF2axRtPYocag6Clbb5YS7kLL+SO/TeVFzHqDIWrNKYtcsPMibjDx9O+bu+psAy9NKfWklassUA==",
      "license": "MIT"
    },
    "node_modules/@tybys/wasm-util": {
      "version": "0.10.3",
      "resolved": "https://registry.npmjs.org/@tybys/wasm-util/-/wasm-util-0.10.3.tgz",
      "integrity": "sha512-F3fo1MYrRJYL3zER0OUOmkutjr1Vp23m7OsSgp7nq4SP6OqX6C/56XFIPAl5bt3zaBRjmW7SGz3u/6LwFpYcOg==",
      "dev": true,
      "license": "MIT",
      "optional": true,
      "dependencies": {
        "tslib": "^2.4.0"
      }
    },
    "node_modules/@types/draco3d": {
      "version": "1.4.10",
      "resolved": "https://registry.npmjs.org/@types/draco3d/-/draco3d-1.4.10.tgz",
      "integrity": "sha512-AX22jp8Y7wwaBgAixaSvkoG4M/+PlAcm3Qs4OW8yT9DM4xUpWKeFhLueTAyZF39pviAdcDdeJoACapiAceqNcw==",
      "license": "MIT"
    },
    "node_modules/@types/estree": {
      "version": "1.0.9",
      "resolved": "https://registry.npmjs.org/@types/estree/-/estree-1.0.9.tgz",
      "integrity": "sha512-GhdPgy1el4/ImP05X05Uw4cw2/M93BCUmnEvWZNStlCzEKME4Fkk+YpoA5OiHNQmoS7Cafb8Xa3Pya8m1Qrzeg==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/@types/json-schema": {
      "version": "7.0.15",
      "resolved": "https://registry.npmjs.org/@types/json-schema/-/json-schema-7.0.15.tgz",
      "integrity": "sha512-5+fP8P8MFNC+AyZCDxrB2pkZFPGzqQWUzpSeuuVLvm8VMcorNYavBqoFcxK8bQz4Qsbn4oUEEem4wDLfcysGHA==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/@types/json5": {
      "version": "0.0.29",
      "resolved": "https://registry.npmjs.org/@types/json5/-/json5-0.0.29.tgz",
      "integrity": "sha512-dRLjCWHYg4oaA77cxO64oO+7JwCwnIzkZPdrrC71jQmQtlhM556pwKo5bUzqvZndkVbeFLIIi+9TC40JNF5hNQ==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/@types/node": {
      "version": "26.1.1",
      "resolved": "https://registry.npmjs.org/@types/node/-/node-26.1.1.tgz",
      "integrity": "sha512-nxAkRSVkN1Y0JC1W8ky/fTfkGsMmcrRsbx+3XoZE+rMOX71kLYTV7fLXpqud1GpbpP5TuffXFqfX7fH2GgZREw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "undici-types": "~8.3.0"
      }
    },
    "node_modules/@types/offscreencanvas": {
      "version": "2019.7.3",
      "resolved": "https://registry.npmjs.org/@types/offscreencanvas/-/offscreencanvas-2019.7.3.tgz",
      "integrity": "sha512-ieXiYmgSRXUDeOntE1InxjWyvEelZGP63M+cGuquuRLuIKKT1osnkXjxev9B7d1nXSug5vpunx+gNlbVxMlC9A==",
      "license": "MIT"
    },
    "node_modules/@types/react": {
      "version": "19.2.17",
      "resolved": "https://registry.npmjs.org/@types/react/-/react-19.2.17.tgz",
      "integrity": "sha512-MXfmqaVPEVgkBT/aY0aGCkRWWtByiYQXo3xdQ8r5RzuFrPiRn8Gar2tQdXSUQ2GKV3bkXckek89V8wQBY2Q/Aw==",
      "license": "MIT",
      "dependencies": {
        "csstype": "^3.2.2"
      }
    },
    "node_modules/@types/react-dom": {
      "version": "19.2.3",
      "resolved": "https://registry.npmjs.org/@types/react-dom/-/react-dom-19.2.3.tgz",
      "integrity": "sha512-jp2L/eY6fn+KgVVQAOqYItbF0VY/YApe5Mz2F0aykSO8gx31bYCZyvSeYxCHKvzHG5eZjc+zyaS5BrBWya2+kQ==",
      "dev": true,
      "license": "MIT",
      "peerDependencies": {
        "@types/react": "^19.2.0"
      }
    },
    "node_modules/@types/react-reconciler": {
      "version": "0.28.9",
      "resolved": "https://registry.npmjs.org/@types/react-reconciler/-/react-reconciler-0.28.9.tgz",
      "integrity": "sha512-HHM3nxyUZ3zAylX8ZEyrDNd2XZOnQ0D5XfunJF5FLQnZbHHYq4UWvW1QfelQNXv1ICNkwYhfxjwfnqivYB6bFg==",
      "license": "MIT",
      "peerDependencies": {
        "@types/react": "*"
      }
    },
    "node_modules/@types/stats.js": {
      "version": "0.17.4",
      "resolved": "https://registry.npmjs.org/@types/stats.js/-/stats.js-0.17.4.tgz",
      "integrity": "sha512-jIBvWWShCvlBqBNIZt0KAshWpvSjhkwkEu4ZUcASoAvhmrgAUI2t1dXrjSL4xXVLB4FznPrIsX3nKXFl/Dt4vA==",
      "license": "MIT"
    },
    "node_modules/@types/three": {
      "version": "0.185.1",
      "resolved": "https://registry.npmjs.org/@types/three/-/three-0.185.1.tgz",
      "integrity": "sha512-db1xTb+EgYF2didW+eudSvVPtn75zo+fGsY8ShQrJY/B5ZBmC2Fiaykv3aImHAlCNEGuMPkPGXBJGLwzu5mC7A==",
      "dependencies": {
        "@dimforge/rapier3d-compat": "~0.12.0",
        "@tweenjs/tween.js": "~23.1.3",
        "@types/stats.js": "*",
        "@types/webxr": ">=0.5.17",
        "fflate": "~0.8.2",
        "meshoptimizer": "~1.1.1"
      }
    },
    "node_modules/@types/webxr": {
      "version": "0.5.24",
      "resolved": "https://registry.npmjs.org/@types/webxr/-/webxr-0.5.24.tgz",
      "integrity": "sha512-h8fgEd/DpoS9CBrjEQXR+dIDraopAEfu4wYVNY2tEPwk60stPWhvZMf4Foo5FakuQ7HFZoa8WceaWFervK2Ovg==",
      "license": "MIT"
    },
    "node_modules/@typescript-eslint/eslint-plugin": {
      "version": "8.65.0",
      "resolved": "https://registry.npmjs.org/@typescript-eslint/eslint-plugin/-/eslint-plugin-8.65.0.tgz",
      "integrity": "sha512-IEgob78X12rHpUmtcwFsXhZdVGJtwTVP8FiCLZkR6GlYVrl2PcuB+KhCE5BlVC/eQpQnu8WXRtkHZuPar+gCRA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@eslint-community/regexpp": "^4.12.2",
        "@typescript-eslint/scope-manager": "8.65.0",
        "@typescript-eslint/type-utils": "8.65.0",
        "@typescript-eslint/utils": "8.65.0",
        "@typescript-eslint/visitor-keys": "8.65.0",
        "ignore": "^7.0.5",
        "natural-compare": "^1.4.0",
        "ts-api-utils": "^2.5.0"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/typescript-eslint"
      },
      "peerDependencies": {
        "@typescript-eslint/parser": "^8.65.0",
        "eslint": "^8.57.0 || ^9.0.0 || ^10.0.0",
        "typescript": ">=4.8.4 <6.1.0"
      }
    },
    "node_modules/@typescript-eslint/eslint-plugin/node_modules/ignore": {
      "version": "7.0.6",
      "resolved": "https://registry.npmjs.org/ignore/-/ignore-7.0.6.tgz",
      "integrity": "sha512-BAg6QkE8W+TuQLrrw0Ugr7HegXduRuuj8/ti2kSOc+jz1dmx8/WNcjr6XGnq5YpDWxFwwaavqD0+jIUOKelTsw==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 4"
      }
    },
    "node_modules/@typescript-eslint/parser": {
      "version": "8.65.0",
      "resolved": "https://registry.npmjs.org/@typescript-eslint/parser/-/parser-8.65.0.tgz",
      "integrity": "sha512-CZ4nMxWwgu1HEEFNkeaCptra9QCtkmKdgf3sWh1rl1trIhmxLilgTV4cwcbQ4wemnT4sWQN8CaKOmdYx+g2gMA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@typescript-eslint/scope-manager": "8.65.0",
        "@typescript-eslint/types": "8.65.0",
        "@typescript-eslint/typescript-estree": "8.65.0",
        "@typescript-eslint/visitor-keys": "8.65.0",
        "debug": "^4.4.3"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/typescript-eslint"
      },
      "peerDependencies": {
        "eslint": "^8.57.0 || ^9.0.0 || ^10.0.0",
        "typescript": ">=4.8.4 <6.1.0"
      }
    },
    "node_modules/@typescript-eslint/project-service": {
      "version": "8.65.0",
      "resolved": "https://registry.npmjs.org/@typescript-eslint/project-service/-/project-service-8.65.0.tgz",
      "integrity": "sha512-SxnPhbTsGahizDgbu7oqFH/xVtzIqMd/s+WtnSxNxJZJpLbdT5IPdzg8EZxO3+PoKahXmwJLeNQOpKJb3/bi7Q==",
      "dev": true,
      "dependencies": {
        "@typescript-eslint/tsconfig-utils": "^8.65.0",
        "@typescript-eslint/types": "^8.65.0",
        "debug": "^4.4.3"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/typescript-eslint"
      },
      "peerDependencies": {
        "typescript": ">=4.8.4 <6.1.0"
      }
    },
    "node_modules/@typescript-eslint/scope-manager": {
      "version": "8.65.0",
      "resolved": "https://registry.npmjs.org/@typescript-eslint/scope-manager/-/scope-manager-8.65.0.tgz",
      "integrity": "sha512-Esbl8OSYiVxBokYgWPf7VVWg/BE798wXhimnn9ML9Pt5qoDf8bfQlgjlKXR/k98+AcNzlLKYrpCcrcuZ9DZLgg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@typescript-eslint/types": "8.65.0",
        "@typescript-eslint/visitor-keys": "8.65.0"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/typescript-eslint"
      }
    },
    "node_modules/@typescript-eslint/tsconfig-utils": {
      "version": "8.65.0",
      "resolved": "https://registry.npmjs.org/@typescript-eslint/tsconfig-utils/-/tsconfig-utils-8.65.0.tgz",
      "integrity": "sha512-j6GzGqCiRdA7Qhur2VVmKZAkBLfnHFQfx4TaJGL9RMveZqCo48jSHHO0DTgizEnGhtWnqmbtCUSrqSkdiY/0Hg==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/typescript-eslint"
      },
      "peerDependencies": {
        "typescript": ">=4.8.4 <6.1.0"
      }
    },
    "node_modules/@typescript-eslint/type-utils": {
      "version": "8.65.0",
      "resolved": "https://registry.npmjs.org/@typescript-eslint/type-utils/-/type-utils-8.65.0.tgz",
      "integrity": "sha512-YjaZ7PRI5qY7ax2L3PbvX0rRyGtipAReCWs0mhhDBHjH/vl0g0BonaGXrKdKpMbIIsMIwDgbk/xzkBTyAltS5g==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@typescript-eslint/types": "8.65.0",
        "@typescript-eslint/typescript-estree": "8.65.0",
        "@typescript-eslint/utils": "8.65.0",
        "debug": "^4.4.3",
        "ts-api-utils": "^2.5.0"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/typescript-eslint"
      },
      "peerDependencies": {
        "eslint": "^8.57.0 || ^9.0.0 || ^10.0.0",
        "typescript": ">=4.8.4 <6.1.0"
      }
    },
    "node_modules/@typescript-eslint/types": {
      "version": "8.65.0",
      "resolved": "https://registry.npmjs.org/@typescript-eslint/types/-/types-8.65.0.tgz",
      "integrity": "sha512-JSSwWNy+H0E/01jJEM+hrX6N0OFDzFzeIhHFSAS01tlVaevpG8cFyYRPhS5yjGOvBUx3sqQHVMjCL1CAZZMxBg==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/typescript-eslint"
      }
    },
    "node_modules/@typescript-eslint/typescript-estree": {
      "version": "8.65.0",
      "resolved": "https://registry.npmjs.org/@typescript-eslint/typescript-estree/-/typescript-estree-8.65.0.tgz",
      "integrity": "sha512-JboAE2swaYt4tb1fHhHTABE2K+OLy09XfcTbhnk4Pw96f9dd2e9iYsJ28gBggHlo5z5x1rkyWvcPoTuNTd4oGg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@typescript-eslint/project-service": "8.65.0",
        "@typescript-eslint/tsconfig-utils": "8.65.0",
        "@typescript-eslint/types": "8.65.0",
        "@typescript-eslint/visitor-keys": "8.65.0",
        "debug": "^4.4.3",
        "minimatch": "^10.2.2",
        "semver": "^7.7.3",
        "tinyglobby": "^0.2.15",
        "ts-api-utils": "^2.5.0"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/typescript-eslint"
      },
      "peerDependencies": {
        "typescript": ">=4.8.4 <6.1.0"
      }
    },
    "node_modules/@typescript-eslint/typescript-estree/node_modules/balanced-match": {
      "version": "4.0.4",
      "resolved": "https://registry.npmjs.org/balanced-match/-/balanced-match-4.0.4.tgz",
      "integrity": "sha512-BLrgEcRTwX2o6gGxGOCNyMvGSp35YofuYzw9h1IMTRmKqttAZZVU67bdb9Pr2vUHA8+j3i2tJfjO6C6+4myGTA==",
      "dev": true,
      "engines": {
        "node": "18 || 20 || >=22"
      }
    },
    "node_modules/@typescript-eslint/typescript-estree/node_modules/brace-expansion": {
      "version": "5.0.7",
      "resolved": "https://registry.npmjs.org/brace-expansion/-/brace-expansion-5.0.7.tgz",
      "integrity": "sha512-7oFy703dxfY3/NLxC1fh2SUCQ0H9rmAY+5EpDVfXjUTTs+HEwR2nYaqLv+GWcTsumwxPfiz6CzCNkwXwBUwqCA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "balanced-match": "^4.0.2"
      },
      "engines": {
        "node": "18 || 20 || >=22"
      }
    },
    "node_modules/@typescript-eslint/typescript-estree/node_modules/minimatch": {
      "version": "10.2.5",
      "resolved": "https://registry.npmjs.org/minimatch/-/minimatch-10.2.5.tgz",
      "integrity": "sha512-MULkVLfKGYDFYejP07QOurDLLQpcjk7Fw+7jXS2R2czRQzR56yHRveU5NDJEOviH+hETZKSkIk5c+T23GjFUMg==",
      "dev": true,
      "license": "BlueOak-1.0.0",
      "dependencies": {
        "brace-expansion": "^5.0.5"
      },
      "engines": {
        "node": "18 || 20 || >=22"
      },
      "funding": {
        "url": "https://github.com/sponsors/isaacs"
      }
    },
    "node_modules/@typescript-eslint/typescript-estree/node_modules/semver": {
      "version": "7.8.5",
      "resolved": "https://registry.npmjs.org/semver/-/semver-7.8.5.tgz",
      "integrity": "sha512-Y7/KDsb8LjooZpwaqGyulO6DQlksgCncchHGk+sZIY4SBvUocMBEFH5Ur1fI4dV+Jvl0w6cjvucaIi40puRioA==",
      "dev": true,
      "license": "ISC",
      "bin": {
        "semver": "bin/semver.js"
      },
      "engines": {
        "node": ">=10"
      }
    },
    "node_modules/@typescript-eslint/utils": {
      "version": "8.65.0",
      "resolved": "https://registry.npmjs.org/@typescript-eslint/utils/-/utils-8.65.0.tgz",
      "integrity": "sha512-gXiwIHsYreboxeJucHKPvgwl7dXt50mF8s1/c00cP/WoVTyWKFdtfhRWwZiXYFU5H2O8vVoSLNrexFZjYS/SGA==",
      "dev": true,
      "dependencies": {
        "@eslint-community/eslint-utils": "^4.9.1",
        "@typescript-eslint/scope-manager": "8.65.0",
        "@typescript-eslint/types": "8.65.0",
        "@typescript-eslint/typescript-estree": "8.65.0"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/typescript-eslint"
      },
      "peerDependencies": {
        "eslint": "^8.57.0 || ^9.0.0 || ^10.0.0",
        "typescript": ">=4.8.4 <6.1.0"
      }
    },
    "node_modules/@typescript-eslint/visitor-keys": {
      "version": "8.65.0",
      "resolved": "https://registry.npmjs.org/@typescript-eslint/visitor-keys/-/visitor-keys-8.65.0.tgz",
      "integrity": "sha512-8C71BQkGjiMmXtop7pHVJu1l2NNShFdkCyD6a2ezzs5vU/L3LRtb69EtcteFwz0mYMPzIgOw0n6OV4VBUWZd7A==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@typescript-eslint/types": "8.65.0",
        "eslint-visitor-keys": "^5.0.0"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/typescript-eslint"
      }
    },
    "node_modules/@typescript-eslint/visitor-keys/node_modules/eslint-visitor-keys": {
      "version": "5.0.1",
      "resolved": "https://registry.npmjs.org/eslint-visitor-keys/-/eslint-visitor-keys-5.0.1.tgz",
      "integrity": "sha512-tD40eHxA35h0PEIZNeIjkHoDR4YjjJp34biM0mDvplBe//mB+IHCqHDGV7pxF+7MklTvighcCPPZC7ynWyjdTA==",
      "dev": true,
      "license": "Apache-2.0",
      "engines": {
        "node": "^20.19.0 || ^22.13.0 || >=24"
      },
      "funding": {
        "url": "https://opencollective.com/eslint"
      }
    },
    "node_modules/@unrs/resolver-binding-android-arm-eabi": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-android-arm-eabi/-/resolver-binding-android-arm-eabi-1.12.2.tgz",
      "integrity": "sha512-g5T90pqg1bo/7mytQx6F4iBNC0Wsh9cu+z9veDbFjc7HjpesJFWD7QMS0NGStXM075+7dJPPVvBbpZlnrdpi/w==",
      "cpu": [
        "arm"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "android"
      ]
    },
    "node_modules/@unrs/resolver-binding-android-arm64": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-android-arm64/-/resolver-binding-android-arm64-1.12.2.tgz",
      "integrity": "sha512-YGCRZv/9GLhwmz6mYDeTsm/92BAyR28l6c2ReweVW5pWgfsitWLY8upvfRlGdoyD8HjeTHSYJWyZGD4KJA/nFQ==",
      "cpu": [
        "arm64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "android"
      ]
    },
    "node_modules/@unrs/resolver-binding-darwin-arm64": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-darwin-arm64/-/resolver-binding-darwin-arm64-1.12.2.tgz",
      "integrity": "sha512-u9DiNT1auQMO20A9SyTuG3wUgQWB9Z7KjAg0uFuCDR1FsAY8A0CG2S6JpHS1xwm/w1G08bjXZDcyOCjv1WAm2w==",
      "cpu": [
        "arm64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "darwin"
      ]
    },
    "node_modules/@unrs/resolver-binding-darwin-x64": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-darwin-x64/-/resolver-binding-darwin-x64-1.12.2.tgz",
      "integrity": "sha512-f7rPLi/T1HVKZu/u6t87lroib16n8vrSzcyxI7lg4BGO9UF26KhQL44sd9eOUgrTYhvRXtWOIZT5PejdPyJfUA==",
      "cpu": [
        "x64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "darwin"
      ]
    },
    "node_modules/@unrs/resolver-binding-freebsd-x64": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-freebsd-x64/-/resolver-binding-freebsd-x64-1.12.2.tgz",
      "integrity": "sha512-BpcOjWCJub6nRZUS2zA20pmLvjtqAtGejETaIyRLiZiQf++cbrjltLA5NN/xaXfqeOBOSlMFbemIl5/S5tljmg==",
      "cpu": [
        "x64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "freebsd"
      ]
    },
    "node_modules/@unrs/resolver-binding-linux-arm-gnueabihf": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-linux-arm-gnueabihf/-/resolver-binding-linux-arm-gnueabihf-1.12.2.tgz",
      "integrity": "sha512-vZTDvdSISZjJx66OzJqtsOhzifbqRjbmI1Mnu49fQDwog5GtDI4QidRiEAYbZCRj9C8YZEW+3ZjqsyS9GR4k2A==",
      "cpu": [
        "arm"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ]
    },
    "node_modules/@unrs/resolver-binding-linux-arm-musleabihf": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-linux-arm-musleabihf/-/resolver-binding-linux-arm-musleabihf-1.12.2.tgz",
      "integrity": "sha512-BiPI+IrIlwcW4nLLMM21+B1dFPzd55yAVgVGrdgDjNef+ch03GdxrcyaIz8X9SsQirh/kCQ7mviyWlMxdh2D7g==",
      "cpu": [
        "arm"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ]
    },
    "node_modules/@unrs/resolver-binding-linux-arm64-gnu": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-linux-arm64-gnu/-/resolver-binding-linux-arm64-gnu-1.12.2.tgz",
      "integrity": "sha512-zJc0H99FEPoFfSrNpa91HYfxzfAJCr502oxNK1cfdC9hlaFI43RT+JFCann9JUgZmLzzntChHyn13Sgn9ljHNg==",
      "cpu": [
        "arm64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ]
    },
    "node_modules/@unrs/resolver-binding-linux-arm64-musl": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-linux-arm64-musl/-/resolver-binding-linux-arm64-musl-1.12.2.tgz",
      "integrity": "sha512-KQ3Lki6l+Pz1k/eBipN41ES+YUK30beLGb9YqcB1O542cyLCNE6GaxrfcY3T6EezmGGk84wb5XyO9loTM9tkcA==",
      "cpu": [
        "arm64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ]
    },
    "node_modules/@unrs/resolver-binding-linux-loong64-gnu": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-linux-loong64-gnu/-/resolver-binding-linux-loong64-gnu-1.12.2.tgz",
      "integrity": "sha512-3SJGEh1DborhG6pyxvhPzCT4bbSIVihsvgJc13P1bHG7KLdNDaF9T3gsTwFc7Jw/5Y5/iWOjkEx7Zy0NvCGX3Q==",
      "cpu": [
        "loong64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ]
    },
    "node_modules/@unrs/resolver-binding-linux-loong64-musl": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-linux-loong64-musl/-/resolver-binding-linux-loong64-musl-1.12.2.tgz",
      "integrity": "sha512-jiuG/Obbel7uw1PwHNFfrkiKhLAF6mnyZ6aWlOAVN9WqKm8v0OFGnciJIHu8+CMvXLQ8AD51LPzAoUfT21D5Ew==",
      "cpu": [
        "loong64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ]
    },
    "node_modules/@unrs/resolver-binding-linux-ppc64-gnu": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-linux-ppc64-gnu/-/resolver-binding-linux-ppc64-gnu-1.12.2.tgz",
      "integrity": "sha512-q7xRvVpmcfeL+LlZg8Pbbo6QaTZwDU5BaGZbwfhkEsXJn3Was8xYfE0RBH266xZt0rM6B7i8xAYIvjthuUIWHg==",
      "cpu": [
        "ppc64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ]
    },
    "node_modules/@unrs/resolver-binding-linux-riscv64-gnu": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-linux-riscv64-gnu/-/resolver-binding-linux-riscv64-gnu-1.12.2.tgz",
      "integrity": "sha512-0CVdx6lcnT3Q9inOH8tsMIOJ6ImndllMjqJHg8RLVdB7Vq4SfkEXl9mCSsVNuNA4MCYycRicCUxPCabVHJRr6A==",
      "cpu": [
        "riscv64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ]
    },
    "node_modules/@unrs/resolver-binding-linux-riscv64-musl": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-linux-riscv64-musl/-/resolver-binding-linux-riscv64-musl-1.12.2.tgz",
      "integrity": "sha512-iOwlRo9vnp6R6ohHQS11n0NnfdXx/omhkocmIfaPRpQhKZ+3BDMkkdRVh53qjkFkpPddf+FETA28NwGN7l5l+w==",
      "cpu": [
        "riscv64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ]
    },
    "node_modules/@unrs/resolver-binding-linux-s390x-gnu": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-linux-s390x-gnu/-/resolver-binding-linux-s390x-gnu-1.12.2.tgz",
      "integrity": "sha512-HYJtLfXq94q8iZNFT1lknx258wlkkWhZeUXJRqzKBBUJ00CvZ+N33zgbCqimLjsyw5Va6uUxhVa12mI+kaveEw==",
      "cpu": [
        "s390x"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ]
    },
    "node_modules/@unrs/resolver-binding-linux-x64-gnu": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-linux-x64-gnu/-/resolver-binding-linux-x64-gnu-1.12.2.tgz",
      "integrity": "sha512-mPsUhunKKDih5O96Y6enDQyHc1SqBPlY1E/SfMWDM3EdJ95Z9CArPeCVwCCqbP45ljvivdEk8Fxn+SIb1rDAJQ==",
      "cpu": [
        "x64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ]
    },
    "node_modules/@unrs/resolver-binding-linux-x64-musl": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-linux-x64-musl/-/resolver-binding-linux-x64-musl-1.12.2.tgz",
      "integrity": "sha512-azrt6+5ydLd8Vt210AAFis/lZevSfPw93EJRIJG+xPu4WCJ8K0kppCTpMyLPcKT7H15M4Jnt2tMp5bOvCkRC6A==",
      "cpu": [
        "x64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "linux"
      ]
    },
    "node_modules/@unrs/resolver-binding-openharmony-arm64": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-openharmony-arm64/-/resolver-binding-openharmony-arm64-1.12.2.tgz",
      "integrity": "sha512-YZ9hP4O0X9PQb8eO980qmLNGH4zT3I9+SZTdt0Pr0YyuGQhYKoOZkV02VzrzyOZJ5xIJ3UFIenKkUkGg8GjgWQ==",
      "cpu": [
        "arm64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "openharmony"
      ]
    },
    "node_modules/@unrs/resolver-binding-wasm32-wasi": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-wasm32-wasi/-/resolver-binding-wasm32-wasi-1.12.2.tgz",
      "integrity": "sha512-tYFDIkMxSflfEc/h92ZWNsZlHSwgimbNHSO3PL2JWQHfCuC2q316jMyYU9TIWZsFK2bQwyK5VAdYgn8ygPj69A==",
      "cpu": [
        "wasm32"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "dependencies": {
        "@emnapi/core": "1.10.0",
        "@emnapi/runtime": "1.10.0",
        "@napi-rs/wasm-runtime": "^1.1.4"
      },
      "engines": {
        "node": ">=14.0.0"
      }
    },
    "node_modules/@unrs/resolver-binding-wasm32-wasi/node_modules/@emnapi/runtime": {
      "version": "1.10.0",
      "resolved": "https://registry.npmjs.org/@emnapi/runtime/-/runtime-1.10.0.tgz",
      "integrity": "sha512-ewvYlk86xUoGI0zQRNq/mC+16R1QeDlKQy21Ki3oSYXNgLb45GV1P6A0M+/s6nyCuNDqe5VpaY84BzXGwVbwFA==",
      "dev": true,
      "license": "MIT",
      "optional": true,
      "dependencies": {
        "tslib": "^2.4.0"
      }
    },
    "node_modules/@unrs/resolver-binding-win32-arm64-msvc": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-win32-arm64-msvc/-/resolver-binding-win32-arm64-msvc-1.12.2.tgz",
      "integrity": "sha512-qzNyg3xL0VPQmCaUh+N5jSitce6k+uCBfMDesWRnlULOZaqUkaJ0ybdT+UqlAWJoQjuqfIU/0Ptx9bteN4D82g==",
      "cpu": [
        "arm64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "win32"
      ]
    },
    "node_modules/@unrs/resolver-binding-win32-ia32-msvc": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-win32-ia32-msvc/-/resolver-binding-win32-ia32-msvc-1.12.2.tgz",
      "integrity": "sha512-WD9sY00OfpHVGfsnHZoA8jVT+esS/Bg8z8jzxp5BnDCjjwsuKsPQrzswwpFy4J1AUJbXPRfkpcX0mXrzeXW79g==",
      "cpu": [
        "ia32"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "win32"
      ]
    },
    "node_modules/@unrs/resolver-binding-win32-x64-msvc": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/@unrs/resolver-binding-win32-x64-msvc/-/resolver-binding-win32-x64-msvc-1.12.2.tgz",
      "integrity": "sha512-nAB74NfSNKknqQ1RrYj6uz8FcXEomu/MATJZxh/x+BArzN2U3JbOYC0APYzUIGhVY3m5hRxA8VPNdPBoG8txlA==",
      "cpu": [
        "x64"
      ],
      "dev": true,
      "license": "MIT",
      "optional": true,
      "os": [
        "win32"
      ]
    },
    "node_modules/@use-gesture/core": {
      "version": "10.3.1",
      "resolved": "https://registry.npmjs.org/@use-gesture/core/-/core-10.3.1.tgz",
      "integrity": "sha512-WcINiDt8WjqBdUXye25anHiNxPc0VOrlT8F6LLkU6cycrOGUDyY/yyFmsg3k8i5OLvv25llc0QC45GhR/C8llw==",
      "license": "MIT"
    },
    "node_modules/@use-gesture/react": {
      "version": "10.3.1",
      "resolved": "https://registry.npmjs.org/@use-gesture/react/-/react-10.3.1.tgz",
      "integrity": "sha512-Yy19y6O2GJq8f7CHf7L0nxL8bf4PZCPaVOCgJrusOeFHY1LvHgYXnmnXg6N5iwAnbgbZCDjo60SiM6IPJi9C5g==",
      "license": "MIT",
      "dependencies": {
        "@use-gesture/core": "10.3.1"
      },
      "peerDependencies": {
        "react": ">= 16.8.0"
      }
    },
    "node_modules/acorn": {
      "version": "8.17.0",
      "resolved": "https://registry.npmjs.org/acorn/-/acorn-8.17.0.tgz",
      "integrity": "sha512-xRQbDb9BnwDafYNn6Vwl839DYVjqXYb1XVGtWAZ1kcDc6iwAL4hg3B1dZlRiuENFeO2H53gFG3in621AdERVAg==",
      "dev": true,
      "license": "MIT",
      "bin": {
        "acorn": "bin/acorn"
      },
      "engines": {
        "node": ">=0.4.0"
      }
    },
    "node_modules/acorn-jsx": {
      "version": "5.3.2",
      "resolved": "https://registry.npmjs.org/acorn-jsx/-/acorn-jsx-5.3.2.tgz",
      "integrity": "sha512-rq9s+JNhf0IChjtDXxllJ7g41oZk5SlXtp0LHwyA5cejwn7vKmKp4pPri6YEePv2PU65sAsegbXtIinmDFDXgQ==",
      "dev": true,
      "peerDependencies": {
        "acorn": "^6.0.0 || ^7.0.0 || ^8.0.0"
      }
    },
    "node_modules/ajv": {
      "version": "6.15.0",
      "resolved": "https://registry.npmjs.org/ajv/-/ajv-6.15.0.tgz",
      "integrity": "sha512-fgFx7Hfoq60ytK2c7DhnF8jIvzYgOMxfugjLOSMHjLIPgenqa7S7oaagATUq99mV6IYvN2tRmC0wnTYX6iPbMw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "fast-deep-equal": "^3.1.1",
        "fast-json-stable-stringify": "^2.0.0",
        "json-schema-traverse": "^0.4.1",
        "uri-js": "^4.2.2"
      },
      "funding": {
        "type": "github",
        "url": "https://github.com/sponsors/epoberezkin"
      }
    },
    "node_modules/ansi-styles": {
      "version": "4.3.0",
      "resolved": "https://registry.npmjs.org/ansi-styles/-/ansi-styles-4.3.0.tgz",
      "integrity": "sha512-zbB9rCJAT1rbjiVDb2hqKFHNYLxgtk8NURxZ3IZwD3F6NtxbXZQCnnSi1Lkx+IDohdPlFp222wVALIheZJQSEg==",
      "dev": true,
      "dependencies": {
        "color-convert": "^2.0.1"
      },
      "engines": {
        "node": ">=8"
      },
      "funding": {
        "url": "https://github.com/chalk/ansi-styles?sponsor=1"
      }
    },
    "node_modules/argparse": {
      "version": "2.0.1",
      "resolved": "https://registry.npmjs.org/argparse/-/argparse-2.0.1.tgz",
      "integrity": "sha512-8+9WqebbFzpX9OR+Wa6O29asIogeRMzcGtAINdpMHHyAg10f05aSFVBbcEqGf/PXw1EjAZ+q2/bEBg3DvurK3Q==",
      "dev": true,
      "license": "Python-2.0"
    },
    "node_modules/aria-query": {
      "version": "5.3.2",
      "resolved": "https://registry.npmjs.org/aria-query/-/aria-query-5.3.2.tgz",
      "integrity": "sha512-COROpnaoap1E2F000S62r6A60uHZnmlvomhfyT2DlTcrY1OrBKn2UhH7qn5wTC9zMvD0AY7csdPSNwKP+7WiQw==",
      "dev": true,
      "license": "Apache-2.0",
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/array-buffer-byte-length": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/array-buffer-byte-length/-/array-buffer-byte-length-1.0.2.tgz",
      "integrity": "sha512-LHE+8BuR7RYGDKvnrmcuSq3tDcKv9OFEXQt/HpbZhY7V6h0zlUXutnAD82GiFx9rdieCMjkvtcsPqBwgUl1Iiw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.3",
        "is-array-buffer": "^3.0.5"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/array-includes": {
      "version": "3.1.9",
      "resolved": "https://registry.npmjs.org/array-includes/-/array-includes-3.1.9.tgz",
      "integrity": "sha512-FmeCCAenzH0KH381SPT5FZmiA/TmpndpcaShhfgEN9eCVjnFBqq3l1xrI42y8+PPLI6hypzou4GXw00WHmPBLQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.8",
        "call-bound": "^1.0.4",
        "define-properties": "^1.2.1",
        "es-abstract": "^1.24.0",
        "es-object-atoms": "^1.1.1",
        "get-intrinsic": "^1.3.0",
        "is-string": "^1.1.1",
        "math-intrinsics": "^1.1.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/array.prototype.findlast": {
      "version": "1.2.5",
      "resolved": "https://registry.npmjs.org/array.prototype.findlast/-/array.prototype.findlast-1.2.5.tgz",
      "integrity": "sha512-CVvd6FHg1Z3POpBLxO6E6zr+rSKEQ9L6rZHAaY7lLfhKsWYUBBOuMs0e9o24oopj6H+geRCX0YJ+TJLBK2eHyQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.7",
        "define-properties": "^1.2.1",
        "es-abstract": "^1.23.2",
        "es-errors": "^1.3.0",
        "es-object-atoms": "^1.0.0",
        "es-shim-unscopables": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/array.prototype.findlastindex": {
      "version": "1.2.6",
      "resolved": "https://registry.npmjs.org/array.prototype.findlastindex/-/array.prototype.findlastindex-1.2.6.tgz",
      "integrity": "sha512-F/TKATkzseUExPlfvmwQKGITM3DGTK+vkAsCZoDc5daVygbJBnjEUCbgkAvVFsgfXfX4YIqZ/27G3k3tdXrTxQ==",
      "dev": true,
      "dependencies": {
        "call-bind": "^1.0.8",
        "call-bound": "^1.0.4",
        "define-properties": "^1.2.1",
        "es-abstract": "^1.23.9",
        "es-errors": "^1.3.0",
        "es-object-atoms": "^1.1.1",
        "es-shim-unscopables": "^1.1.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/array.prototype.flat": {
      "version": "1.3.3",
      "resolved": "https://registry.npmjs.org/array.prototype.flat/-/array.prototype.flat-1.3.3.tgz",
      "integrity": "sha512-rwG/ja1neyLqCuGZ5YYrznA62D4mZXg0i1cIskIUKSiqF3Cje9/wXAls9B9s1Wa2fomMsIv8czB8jZcPmxCXFg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.8",
        "define-properties": "^1.2.1",
        "es-abstract": "^1.23.5",
        "es-shim-unscopables": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/array.prototype.flatmap": {
      "version": "1.3.3",
      "resolved": "https://registry.npmjs.org/array.prototype.flatmap/-/array.prototype.flatmap-1.3.3.tgz",
      "integrity": "sha512-Y7Wt51eKJSyi80hFrJCePGGNo5ktJCslFuboqJsbf57CCPcm5zztluPlc4/aD8sWsKvlwatezpV4U1efk8kpjg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.8",
        "define-properties": "^1.2.1",
        "es-abstract": "^1.23.5",
        "es-shim-unscopables": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/array.prototype.tosorted": {
      "version": "1.1.4",
      "resolved": "https://registry.npmjs.org/array.prototype.tosorted/-/array.prototype.tosorted-1.1.4.tgz",
      "integrity": "sha512-p6Fx8B7b7ZhL/gmUsAy0D15WhvDccw3mnGNbZpi3pmeJdxtWsj2jEaI4Y6oo3XiHfzuSgPwKc04MYt6KgvC/wA==",
      "dev": true,
      "dependencies": {
        "call-bind": "^1.0.7",
        "define-properties": "^1.2.1",
        "es-abstract": "^1.23.3",
        "es-errors": "^1.3.0",
        "es-shim-unscopables": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/arraybuffer.prototype.slice": {
      "version": "1.0.4",
      "resolved": "https://registry.npmjs.org/arraybuffer.prototype.slice/-/arraybuffer.prototype.slice-1.0.4.tgz",
      "integrity": "sha512-BNoCY6SXXPQ7gF2opIP4GBE+Xw7U+pHMYKuzjgCN3GwiaIR09UUeKfheyIry77QtrCBlC0KK0q5/TER/tYh3PQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "array-buffer-byte-length": "^1.0.1",
        "call-bind": "^1.0.8",
        "define-properties": "^1.2.1",
        "es-abstract": "^1.23.5",
        "es-errors": "^1.3.0",
        "get-intrinsic": "^1.2.6",
        "is-array-buffer": "^3.0.4"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/ast-types-flow": {
      "version": "0.0.8",
      "resolved": "https://registry.npmjs.org/ast-types-flow/-/ast-types-flow-0.0.8.tgz",
      "integrity": "sha512-OH/2E5Fg20h2aPrbe+QL8JZQFko0YZaF+j4mnQ7BGhfavO7OpSLa8a0y9sBwomHdSbkhTS8TQNayBfnW5DwbvQ==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/async-function": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/async-function/-/async-function-1.0.0.tgz",
      "integrity": "sha512-hsU18Ae8CDTR6Kgu9DYf0EbCr/a5iGL0rytQDobUcdpYOKokk8LEjVphnXkDkgpi0wYVsqrXuP0bZxJaTqdgoA==",
      "dev": true,
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/available-typed-arrays": {
      "version": "1.0.7",
      "resolved": "https://registry.npmjs.org/available-typed-arrays/-/available-typed-arrays-1.0.7.tgz",
      "integrity": "sha512-wvUjBtSGN7+7SjNpq/9M2Tg350UZD3q62IFZLbRAR1bSMlCo1ZaeW+BJ+D090e4hIIZLBcTDWe4Mh4jvUDajzQ==",
      "dev": true,
      "dependencies": {
        "possible-typed-array-names": "^1.0.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/axe-core": {
      "version": "4.12.1",
      "resolved": "https://registry.npmjs.org/axe-core/-/axe-core-4.12.1.tgz",
      "integrity": "sha512-s7iGf5GaVMxEG0ENN9x+xTr7GFZCb1ZP/1uATUpCEK2X78nDB3RwbtFCo9pGAf9ru+VwoQ464DkaLEeRM08wJA==",
      "dev": true,
      "license": "MPL-2.0",
      "engines": {
        "node": ">=4"
      }
    },
    "node_modules/axobject-query": {
      "version": "4.1.0",
      "resolved": "https://registry.npmjs.org/axobject-query/-/axobject-query-4.1.0.tgz",
      "integrity": "sha512-qIj0G9wZbMGNLjLmg1PT6v2mE9AH2zlnADJD/2tC6E00hgmhUOfEB6greHPAfLRSufHqROIUTkw6E+M3lH0PTQ==",
      "dev": true,
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/balanced-match": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/balanced-match/-/balanced-match-1.0.2.tgz",
      "integrity": "sha512-3oSeUO0TMV67hN1AmbXsK4yaqU7tjiHlbxRDZOpH0KW9+CeX4bRAaX0Anxt0tx2MrpRpWwQaPwIlISEJhYU5Pw==",
      "dev": true
    },
    "node_modules/base64-js": {
      "version": "1.5.1",
      "resolved": "https://registry.npmjs.org/base64-js/-/base64-js-1.5.1.tgz",
      "integrity": "sha512-AKpaYlHn8t4SVbOHCy+b5+KKgvR4vrsD8vbvrbiQJps7fKDTkjkDry6ji0rUJjC0kzbNePLwzxq8iypo41qeWA==",
      "funding": [
        {
          "type": "github",
          "url": "https://github.com/sponsors/feross"
        },
        {
          "type": "patreon",
          "url": "https://www.patreon.com/feross"
        },
        {
          "type": "consulting",
          "url": "https://feross.org/support"
        }
      ],
      "license": "MIT"
    },
    "node_modules/baseline-browser-mapping": {
      "version": "2.10.44",
      "resolved": "https://registry.npmjs.org/baseline-browser-mapping/-/baseline-browser-mapping-2.10.44.tgz",
      "integrity": "sha512-T3ghW+sl/ZJ8w1v/yQx3qvJ9040DWoLBz8JT/CILbAKcFyG9b2MRe75v6W5uXjv6uH1lumK2Kv46y2zSkcej0Q==",
      "license": "Apache-2.0",
      "bin": {
        "baseline-browser-mapping": "dist/cli.cjs"
      },
      "engines": {
        "node": ">=6.0.0"
      }
    },
    "node_modules/bidi-js": {
      "version": "1.0.3",
      "resolved": "https://registry.npmjs.org/bidi-js/-/bidi-js-1.0.3.tgz",
      "integrity": "sha512-RKshQI1R3YQ+n9YJz2QQ147P66ELpa1FQEg20Dk8oW9t2KgLbpDLLp9aGZ7y8WHSshDknG0bknqGw5/tyCs5tw==",
      "license": "MIT",
      "dependencies": {
        "require-from-string": "^2.0.2"
      }
    },
    "node_modules/brace-expansion": {
      "version": "1.1.16",
      "resolved": "https://registry.npmjs.org/brace-expansion/-/brace-expansion-1.1.16.tgz",
      "integrity": "sha512-IDw48K2/2kRkg9LdJxurvq3lV3aBgq0REY89duEqFRthjlPdXHKMj7EnQOXVckxzgisinf3nHfrcE2FufFLXMw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "balanced-match": "^1.0.0",
        "concat-map": "0.0.1"
      }
    },
    "node_modules/braces": {
      "version": "3.0.3",
      "resolved": "https://registry.npmjs.org/braces/-/braces-3.0.3.tgz",
      "integrity": "sha512-yQbXgO/OSZVD2IsiLlro+7Hf6Q18EJrKSEsdoMzKePKXct3gvD8oLcOQdIzGupr5Fj+EDe8gO/lxc1BzfMpxvA==",
      "dev": true,
      "dependencies": {
        "fill-range": "^7.1.1"
      },
      "engines": {
        "node": ">=8"
      }
    },
    "node_modules/browserslist": {
      "version": "4.28.6",
      "resolved": "https://registry.npmjs.org/browserslist/-/browserslist-4.28.6.tgz",
      "integrity": "sha512-FQBYNK15VMslhLHpA7+n+n1GOlF1kId2xcCg7/j95f24AOF6VDYMNH4mFxF7KuaTdv627faazpOAjFzMrfJOUw==",
      "dev": true,
      "funding": [
        {
          "type": "opencollective",
          "url": "https://opencollective.com/browserslist"
        },
        {
          "type": "tidelift",
          "url": "https://tidelift.com/funding/github/npm/browserslist"
        },
        {
          "type": "github",
          "url": "https://github.com/sponsors/ai"
        }
      ],
      "dependencies": {
        "baseline-browser-mapping": "^2.10.42",
        "caniuse-lite": "^1.0.30001803",
        "electron-to-chromium": "^1.5.389",
        "node-releases": "^2.0.51",
        "update-browserslist-db": "^1.2.3"
      },
      "bin": {
        "browserslist": "cli.js"
      },
      "engines": {
        "node": "^6 || ^7 || ^8 || ^9 || ^10 || ^11 || ^12 || >=13.7"
      }
    },
    "node_modules/buffer": {
      "version": "6.0.3",
      "resolved": "https://registry.npmjs.org/buffer/-/buffer-6.0.3.tgz",
      "integrity": "sha512-FTiCpNxtwiZZHEZbcbTIcZjERVICn9yq/pDFkTl95/AxzD1naBctN7YO68riM/gLSDY7sdrMby8hofADYuuqOA==",
      "funding": [
        {
          "type": "github",
          "url": "https://github.com/sponsors/feross"
        },
        {
          "type": "patreon",
          "url": "https://www.patreon.com/feross"
        },
        {
          "type": "consulting",
          "url": "https://feross.org/support"
        }
      ],
      "license": "MIT",
      "dependencies": {
        "base64-js": "^1.3.1",
        "ieee754": "^1.2.1"
      }
    },
    "node_modules/call-bind": {
      "version": "1.0.9",
      "resolved": "https://registry.npmjs.org/call-bind/-/call-bind-1.0.9.tgz",
      "integrity": "sha512-a/hy+pNsFUTR+Iz8TCJvXudKVLAnz/DyeSUo10I5yvFDQJBFU2s9uqQpoSrJlroHUKoKqzg+epxyP9lqFdzfBQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind-apply-helpers": "^1.0.2",
        "es-define-property": "^1.0.1",
        "get-intrinsic": "^1.3.0",
        "set-function-length": "^1.2.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/call-bind-apply-helpers": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/call-bind-apply-helpers/-/call-bind-apply-helpers-1.0.2.tgz",
      "integrity": "sha512-Sp1ablJ0ivDkSzjcaJdxEunN5/XvksFJ2sMBFfq6x0ryhQV/2b/KwFe21cMpmHtPOSij8K99/wSfoEuTObmuMQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "es-errors": "^1.3.0",
        "function-bind": "^1.1.2"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/call-bound": {
      "version": "1.0.4",
      "resolved": "https://registry.npmjs.org/call-bound/-/call-bound-1.0.4.tgz",
      "integrity": "sha512-+ys997U96po4Kx/ABpBCqhA9EuxJaQWDQg7295H4hBphv3IZg0boBKuwYpt4YXp6MZ5AmZQnU/tyMTlRpaSejg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind-apply-helpers": "^1.0.2",
        "get-intrinsic": "^1.3.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/callsites": {
      "version": "3.1.0",
      "resolved": "https://registry.npmjs.org/callsites/-/callsites-3.1.0.tgz",
      "integrity": "sha512-P8BjAsXvZS+VIDUI11hHCQEv74YT67YUi5JJFNWIqL235sBmjX4+qx9Muvls5ivyNENctx46xQLQ3aTuE7ssaQ==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=6"
      }
    },
    "node_modules/camera-controls": {
      "version": "3.1.2",
      "resolved": "https://registry.npmjs.org/camera-controls/-/camera-controls-3.1.2.tgz",
      "integrity": "sha512-xkxfpG2ECZ6Ww5/9+kf4mfg1VEYAoe9aDSY+IwF0UEs7qEzwy0aVRfs2grImIECs/PoBtWFrh7RXsQkwG922JA==",
      "license": "MIT",
      "engines": {
        "node": ">=22.0.0",
        "npm": ">=10.5.1"
      },
      "peerDependencies": {
        "three": ">=0.126.1"
      }
    },
    "node_modules/caniuse-lite": {
      "version": "1.0.30001806",
      "resolved": "https://registry.npmjs.org/caniuse-lite/-/caniuse-lite-1.0.30001806.tgz",
      "integrity": "sha512-72Cuvd95zbSYPKq6Fhg8eDJRlzgWDf7/mtoZv6Qe/DYNCEBdNxoA3+rZAU2ZhGCpZlns3EssFavaZomckT5Uuw==",
      "funding": [
        {
          "type": "opencollective",
          "url": "https://opencollective.com/browserslist"
        },
        {
          "type": "tidelift",
          "url": "https://tidelift.com/funding/github/npm/caniuse-lite"
        },
        {
          "type": "github",
          "url": "https://github.com/sponsors/ai"
        }
      ],
      "license": "CC-BY-4.0"
    },
    "node_modules/chalk": {
      "version": "4.1.2",
      "resolved": "https://registry.npmjs.org/chalk/-/chalk-4.1.2.tgz",
      "integrity": "sha512-oKnbhFyRIXpUuez8iBMmyEa4nbj4IOQyuhc/wy9kY7/WVPcwIO9VA668Pu8RkO7+0G76SLROeyw9CpQ061i4mA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "ansi-styles": "^4.1.0",
        "supports-color": "^7.1.0"
      },
      "engines": {
        "node": ">=10"
      },
      "funding": {
        "url": "https://github.com/chalk/chalk?sponsor=1"
      }
    },
    "node_modules/client-only": {
      "version": "0.0.1",
      "resolved": "https://registry.npmjs.org/client-only/-/client-only-0.0.1.tgz",
      "integrity": "sha512-IV3Ou0jSMzZrd3pZ48nLkT9DA7Ag1pnPzaiQhpW7c3RbcqqzvzzVu+L8gfqMp/8IM2MQtSiqaCxrrcfu8I8rMA==",
      "license": "MIT"
    },
    "node_modules/color-convert": {
      "version": "2.0.1",
      "resolved": "https://registry.npmjs.org/color-convert/-/color-convert-2.0.1.tgz",
      "integrity": "sha512-RRECPsj7iu/xb5oKYcsFHSppFNnsj/52OVTRKb4zP5onXwVF3zVmmToNcOfGC+CRDpfK/U584fMg38ZHCaElKQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "color-name": "~1.1.4"
      },
      "engines": {
        "node": ">=7.0.0"
      }
    },
    "node_modules/color-name": {
      "version": "1.1.4",
      "resolved": "https://registry.npmjs.org/color-name/-/color-name-1.1.4.tgz",
      "integrity": "sha512-dOy+3AuW3a2wNbZHIuMZpTcgjGuLU/uBL/ubcZF9OXbDo8ff4O8yVp5Bf0efS8uEoYo5q4Fx7dY9OgQGXgAsQA==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/concat-map": {
      "version": "0.0.1",
      "resolved": "https://registry.npmjs.org/concat-map/-/concat-map-0.0.1.tgz",
      "integrity": "sha512-/Srv4dswyQNBfohGpz9o6Yb3Gz3SrUDqBH5rTuhGR7ahtlbYKnVxw2bCFMRljaA7EXHaXZ8wsHdodFvbkhKmqg==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/convert-source-map": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/convert-source-map/-/convert-source-map-2.0.0.tgz",
      "integrity": "sha512-Kvp459HrV2FEJ1CAsi1Ku+MY3kasH19TFykTz2xWmMeq6bk2NU3XXvfJ+Q61m0xktWwt+1HSYf3JZsTms3aRJg==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/cross-env": {
      "version": "7.0.3",
      "resolved": "https://registry.npmjs.org/cross-env/-/cross-env-7.0.3.tgz",
      "integrity": "sha512-+/HKd6EgcQCJGh2PSjZuUitQBQynKor4wrFbRg4DtAgS1aWO+gU52xpH7M9ScGgXSYmAVS9bIJ8EzuaGw0oNAw==",
      "license": "MIT",
      "dependencies": {
        "cross-spawn": "^7.0.1"
      },
      "bin": {
        "cross-env": "src/bin/cross-env.js",
        "cross-env-shell": "src/bin/cross-env-shell.js"
      },
      "engines": {
        "node": ">=10.14",
        "npm": ">=6",
        "yarn": ">=1"
      }
    },
    "node_modules/cross-spawn": {
      "version": "7.0.6",
      "resolved": "https://registry.npmjs.org/cross-spawn/-/cross-spawn-7.0.6.tgz",
      "integrity": "sha512-uV2QOWP2nWzsy2aMp8aRibhi9dlzF5Hgh5SHaB9OiTGEyDTiJJyx0uy51QXdyWbtAHNua4XJzUKca3OzKUd3vA==",
      "license": "MIT",
      "dependencies": {
        "path-key": "^3.1.0",
        "shebang-command": "^2.0.0",
        "which": "^2.0.1"
      },
      "engines": {
        "node": ">= 8"
      }
    },
    "node_modules/csstype": {
      "version": "3.2.3",
      "resolved": "https://registry.npmjs.org/csstype/-/csstype-3.2.3.tgz",
      "integrity": "sha512-z1HGKcYy2xA8AGQfwrn0PAy+PB7X/GSj3UVJW9qKyn43xWa+gl5nXmU4qqLMRzWVLFC8KusUX8T/0kCiOYpAIQ==",
      "license": "MIT"
    },
    "node_modules/damerau-levenshtein": {
      "version": "1.0.8",
      "resolved": "https://registry.npmjs.org/damerau-levenshtein/-/damerau-levenshtein-1.0.8.tgz",
      "integrity": "sha512-sdQSFB7+llfUcQHUQO3+B8ERRj0Oa4w9POWMI/puGtuf7gFywGmkaLCElnudfTiKZV+NvHqL0ifzdrI8Ro7ESA==",
      "dev": true,
      "license": "BSD-2-Clause"
    },
    "node_modules/data-view-buffer": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/data-view-buffer/-/data-view-buffer-1.0.2.tgz",
      "integrity": "sha512-EmKO5V3OLXh1rtK2wgXRansaK1/mtVdTUEiEI0W8RkvgT05kfxaH29PliLnpLP73yYO6142Q72QNa8Wx/A5CqQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.3",
        "es-errors": "^1.3.0",
        "is-data-view": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/data-view-byte-length": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/data-view-byte-length/-/data-view-byte-length-1.0.2.tgz",
      "integrity": "sha512-tuhGbE6CfTM9+5ANGf+oQb72Ky/0+s3xKUpHvShfiz2RxMFgFPjsXuRLBVMtvMs15awe45SRb83D6wH4ew6wlQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.3",
        "es-errors": "^1.3.0",
        "is-data-view": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/inspect-js"
      }
    },
    "node_modules/data-view-byte-offset": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/data-view-byte-offset/-/data-view-byte-offset-1.0.1.tgz",
      "integrity": "sha512-BS8PfmtDGnrgYdOonGZQdLZslWIeCGFP9tpan0hi1Co2Zr2NKADsvGYA8XxuG/4UWgJ6Cjtv+YJnB6MM69QGlQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.2",
        "es-errors": "^1.3.0",
        "is-data-view": "^1.0.1"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/debug": {
      "version": "4.4.3",
      "resolved": "https://registry.npmjs.org/debug/-/debug-4.4.3.tgz",
      "integrity": "sha512-RGwwWnwQvkVfavKVt22FGLw+xYSdzARwm0ru6DhTVA3umU5hZc28V3kO4stgYryrTlLpuvgI9GiijltAjNbcqA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "ms": "^2.1.3"
      },
      "engines": {
        "node": ">=6.0"
      },
      "peerDependenciesMeta": {
        "supports-color": {
          "optional": true
        }
      }
    },
    "node_modules/deep-is": {
      "version": "0.1.4",
      "resolved": "https://registry.npmjs.org/deep-is/-/deep-is-0.1.4.tgz",
      "integrity": "sha512-oIPzksmTg4/MriiaYGO+okXDT7ztn/w3Eptv/+gSIdMdKsJo0u4CfYNFJPy+4SKMuCqGw2wxnA+URMg3t8a/bQ==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/define-data-property": {
      "version": "1.1.4",
      "resolved": "https://registry.npmjs.org/define-data-property/-/define-data-property-1.1.4.tgz",
      "integrity": "sha512-rBMvIzlpA8v6E+SJZoo++HAYqsLrkg7MSfIinMPFhmkorw7X+dOXVJQs+QT69zGkzMyfDnIMN2Wid1+NbL3T+A==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "es-define-property": "^1.0.0",
        "es-errors": "^1.3.0",
        "gopd": "^1.0.1"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/define-properties": {
      "version": "1.2.1",
      "resolved": "https://registry.npmjs.org/define-properties/-/define-properties-1.2.1.tgz",
      "integrity": "sha512-8QmQKqEASLd5nx0U1B1okLElbUuuttJ/AnYmRXbbbGDWh6uS208EjD4Xqq/I9wK7u0v6O08XhTWnt5XtEbR6Dg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "define-data-property": "^1.0.1",
        "has-property-descriptors": "^1.0.0",
        "object-keys": "^1.1.1"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/detect-gpu": {
      "version": "5.0.70",
      "resolved": "https://registry.npmjs.org/detect-gpu/-/detect-gpu-5.0.70.tgz",
      "integrity": "sha512-bqerEP1Ese6nt3rFkwPnGbsUF9a4q+gMmpTVVOEzoCyeCc+y7/RvJnQZJx1JwhgQI5Ntg0Kgat8Uu7XpBqnz1w==",
      "license": "MIT",
      "dependencies": {
        "webgl-constants": "^1.1.1"
      }
    },
    "node_modules/detect-libc": {
      "version": "2.1.2",
      "resolved": "https://registry.npmjs.org/detect-libc/-/detect-libc-2.1.2.tgz",
      "integrity": "sha512-Btj2BOOO83o3WyH59e8MgXsxEQVcarkUOpEYrubB0urwnN10yQ364rsiByU11nZlqWYZm05i/of7io4mzihBtQ==",
      "license": "Apache-2.0",
      "optional": true,
      "engines": {
        "node": ">=8"
      }
    },
    "node_modules/doctrine": {
      "version": "2.1.0",
      "resolved": "https://registry.npmjs.org/doctrine/-/doctrine-2.1.0.tgz",
      "integrity": "sha512-35mSku4ZXK0vfCuHEDAwt55dg2jNajHZ1odvF+8SSr82EsZY4QmXfuWso8oEd8zRhVObSN18aM0CjSdoBX7zIw==",
      "dev": true,
      "license": "Apache-2.0",
      "dependencies": {
        "esutils": "^2.0.2"
      },
      "engines": {
        "node": ">=0.10.0"
      }
    },
    "node_modules/draco3d": {
      "version": "1.5.7",
      "resolved": "https://registry.npmjs.org/draco3d/-/draco3d-1.5.7.tgz",
      "integrity": "sha512-m6WCKt/erDXcw+70IJXnG7M3awwQPAsZvJGX5zY7beBqpELw6RDGkYVU0W43AFxye4pDZ5i2Lbyc/NNGqwjUVQ==",
      "license": "Apache-2.0"
    },
    "node_modules/dunder-proto": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/dunder-proto/-/dunder-proto-1.0.1.tgz",
      "integrity": "sha512-KIN/nDJBQRcXw0MLVhZE9iQHmG68qAVIBg9CqmUYjmQIhgij9U5MFvrqkUL5FbtyyzZuOeOt0zdeRe4UY7ct+A==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind-apply-helpers": "^1.0.1",
        "es-errors": "^1.3.0",
        "gopd": "^1.2.0"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/electron-to-chromium": {
      "version": "1.5.394",
      "resolved": "https://registry.npmjs.org/electron-to-chromium/-/electron-to-chromium-1.5.394.tgz",
      "integrity": "sha512-Wmt2Gm0o8JWBuGgmc4XZ0u9s1RaCRqhxP47phplmfg04+qypTUurpeJGP45A7Fhv7jdrrVH44PLlR9qXo37cVQ==",
      "dev": true
    },
    "node_modules/emoji-regex": {
      "version": "9.2.2",
      "resolved": "https://registry.npmjs.org/emoji-regex/-/emoji-regex-9.2.2.tgz",
      "integrity": "sha512-L18DaJsXSUk2+42pv8mLs5jJT2hqFkFE4j21wOmgbUqsZ2hL72NsUU785g9RXgo3s0ZNgVl42TiHp3ZtOv/Vyg==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/es-abstract": {
      "version": "1.24.2",
      "resolved": "https://registry.npmjs.org/es-abstract/-/es-abstract-1.24.2.tgz",
      "integrity": "sha512-2FpH9Q5i2RRwyEP1AylXe6nYLR5OhaJTZwmlcP0dL/+JCbgg7yyEo/sEK6HeGZRf3dFpWwThaRHVApXSkW3xeg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "array-buffer-byte-length": "^1.0.2",
        "arraybuffer.prototype.slice": "^1.0.4",
        "available-typed-arrays": "^1.0.7",
        "call-bind": "^1.0.8",
        "call-bound": "^1.0.4",
        "data-view-buffer": "^1.0.2",
        "data-view-byte-length": "^1.0.2",
        "data-view-byte-offset": "^1.0.1",
        "es-define-property": "^1.0.1",
        "es-errors": "^1.3.0",
        "es-object-atoms": "^1.1.1",
        "es-set-tostringtag": "^2.1.0",
        "es-to-primitive": "^1.3.0",
        "function.prototype.name": "^1.1.8",
        "get-intrinsic": "^1.3.0",
        "get-proto": "^1.0.1",
        "get-symbol-description": "^1.1.0",
        "globalthis": "^1.0.4",
        "gopd": "^1.2.0",
        "has-property-descriptors": "^1.0.2",
        "has-proto": "^1.2.0",
        "has-symbols": "^1.1.0",
        "hasown": "^2.0.2",
        "internal-slot": "^1.1.0",
        "is-array-buffer": "^3.0.5",
        "is-callable": "^1.2.7",
        "is-data-view": "^1.0.2",
        "is-negative-zero": "^2.0.3",
        "is-regex": "^1.2.1",
        "is-set": "^2.0.3",
        "is-shared-array-buffer": "^1.0.4",
        "is-string": "^1.1.1",
        "is-typed-array": "^1.1.15",
        "is-weakref": "^1.1.1",
        "math-intrinsics": "^1.1.0",
        "object-inspect": "^1.13.4",
        "object-keys": "^1.1.1",
        "object.assign": "^4.1.7",
        "own-keys": "^1.0.1",
        "regexp.prototype.flags": "^1.5.4",
        "safe-array-concat": "^1.1.3",
        "safe-push-apply": "^1.0.0",
        "safe-regex-test": "^1.1.0",
        "set-proto": "^1.0.0",
        "stop-iteration-iterator": "^1.1.0",
        "string.prototype.trim": "^1.2.10",
        "string.prototype.trimend": "^1.0.9",
        "string.prototype.trimstart": "^1.0.8",
        "typed-array-buffer": "^1.0.3",
        "typed-array-byte-length": "^1.0.3",
        "typed-array-byte-offset": "^1.0.4",
        "typed-array-length": "^1.0.7",
        "unbox-primitive": "^1.1.0",
        "which-typed-array": "^1.1.19"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/es-abstract-get": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/es-abstract-get/-/es-abstract-get-1.0.0.tgz",
      "integrity": "sha512-6PMWXpdhshVvFp+FoWYs1EvG1Nj0tvk0dZM+XcK0xMEM1czRVcP6ohqPWHy6qPagSpC8j4+p89WXlT+xXJs/fg==",
      "dev": true,
      "dependencies": {
        "es-errors": "^1.3.0",
        "es-object-atoms": "^1.1.2",
        "is-callable": "^1.2.7",
        "object-inspect": "^1.13.4"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/es-define-property": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/es-define-property/-/es-define-property-1.0.1.tgz",
      "integrity": "sha512-e3nRfgfUZ4rNGL232gUgX06QNyyez04KdjFrF+LTRoOXmrOgFKDg4BCdsjW8EnT69eqdYGmRpJwiPVYNrCaW3g==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/es-errors": {
      "version": "1.3.0",
      "resolved": "https://registry.npmjs.org/es-errors/-/es-errors-1.3.0.tgz",
      "integrity": "sha512-Zf5H2Kxt2xjTvbJvP2ZWLEICxA6j+hAmMzIlypy4xcBg1vKVnx89Wy0GbS+kf5cwCVFFzdCFh2XSCFNULS6csw==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/es-iterator-helpers": {
      "version": "1.4.0",
      "resolved": "https://registry.npmjs.org/es-iterator-helpers/-/es-iterator-helpers-1.4.0.tgz",
      "integrity": "sha512-c/A0P0oxkACDc+cKWw8evLXK83oBKgn0qPOqCYT4x9uolpCIJAcYvJC9QYKNDRPsTeGyCrQ326jrvgZWdCdK5Q==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.9",
        "call-bound": "^1.0.4",
        "define-properties": "^1.2.1",
        "es-abstract": "^1.24.2",
        "es-errors": "^1.3.0",
        "es-set-tostringtag": "^2.1.0",
        "function-bind": "^1.1.2",
        "get-intrinsic": "^1.3.0",
        "globalthis": "^1.0.4",
        "gopd": "^1.2.0",
        "has-property-descriptors": "^1.0.2",
        "has-proto": "^1.2.0",
        "has-symbols": "^1.1.0",
        "internal-slot": "^1.1.0",
        "iterator.prototype": "^1.1.5",
        "math-intrinsics": "^1.1.0"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/es-object-atoms": {
      "version": "1.1.2",
      "resolved": "https://registry.npmjs.org/es-object-atoms/-/es-object-atoms-1.1.2.tgz",
      "integrity": "sha512-HWcBoN6NileqtSydK2FqHbS/LoDd2pqrnQHLyJzBj4kOp/ky2MWMN694xOfkK8/SnUsW2DH7EfyVlydKCsm1Zw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "es-errors": "^1.3.0"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/es-set-tostringtag": {
      "version": "2.1.0",
      "resolved": "https://registry.npmjs.org/es-set-tostringtag/-/es-set-tostringtag-2.1.0.tgz",
      "integrity": "sha512-j6vWzfrGVfyXxge+O0x5sh6cvxAog0a/4Rdd2K36zCMV5eJ+/+tOAngRO8cODMNWbVRdVlmGZQL2YS3yR8bIUA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "es-errors": "^1.3.0",
        "get-intrinsic": "^1.2.6",
        "has-tostringtag": "^1.0.2",
        "hasown": "^2.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/es-shim-unscopables": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/es-shim-unscopables/-/es-shim-unscopables-1.1.0.tgz",
      "integrity": "sha512-d9T8ucsEhh8Bi1woXCf+TIKDIROLG5WCkxg8geBCbvk22kzwC5G2OnXVMO6FUsvQlgUUXQ2itephWDLqDzbeCw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "hasown": "^2.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/es-to-primitive": {
      "version": "1.3.4",
      "resolved": "https://registry.npmjs.org/es-to-primitive/-/es-to-primitive-1.3.4.tgz",
      "integrity": "sha512-yPDz7wqpg1/mmHLmS3tcfTfbw5f1eryXvyghYBffGdERwe+mV7ZcWzTR8LR17Kvqt3qfPurjlonmnq3MKXIOXw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "es-abstract-get": "^1.0.0",
        "es-define-property": "^1.0.1",
        "es-errors": "^1.3.0",
        "is-callable": "^1.2.7",
        "is-date-object": "^1.1.0",
        "is-symbol": "^1.1.1"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/escalade": {
      "version": "3.2.0",
      "resolved": "https://registry.npmjs.org/escalade/-/escalade-3.2.0.tgz",
      "integrity": "sha512-WUj2qlxaQtO4g6Pq5c29GTcWGDyd8itL8zTlipgECz3JesAiiOKotd8JU6otB3PACgG6xkJUyVhboMS+bje/jA==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=6"
      }
    },
    "node_modules/escape-string-regexp": {
      "version": "4.0.0",
      "resolved": "https://registry.npmjs.org/escape-string-regexp/-/escape-string-regexp-4.0.0.tgz",
      "integrity": "sha512-TtpcNJ3XAzx3Gq8sWRzJaVajRs0uVxA2YAkdb1jm2YkPz4G6egUFAyA3n5vtEIZefPk5Wa4UXbKuS5fKkJWdgA==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=10"
      },
      "funding": {
        "url": "https://github.com/sponsors/sindresorhus"
      }
    },
    "node_modules/eslint": {
      "version": "9.39.5",
      "resolved": "https://registry.npmjs.org/eslint/-/eslint-9.39.5.tgz",
      "integrity": "sha512-DgZS62aPLXKlnxILS/AYCoRvHaZeXceIzlXPkkGGzJWSow1aEk0lbTlxUSlyjC8jcaKxAdOnTDz+o1JFSBsyjw==",
      "dev": true,
      "dependencies": {
        "@eslint-community/eslint-utils": "^4.8.0",
        "@eslint-community/regexpp": "^4.12.1",
        "@eslint/config-array": "^0.21.2",
        "@eslint/config-helpers": "^0.4.2",
        "@eslint/core": "^0.17.0",
        "@eslint/eslintrc": "^3.3.6",
        "@eslint/js": "9.39.5",
        "@eslint/plugin-kit": "^0.4.1",
        "@humanfs/node": "^0.16.6",
        "@humanwhocodes/module-importer": "^1.0.1",
        "@humanwhocodes/retry": "^0.4.2",
        "@types/estree": "^1.0.6",
        "ajv": "^6.14.0",
        "chalk": "^4.0.0",
        "cross-spawn": "^7.0.6",
        "debug": "^4.3.2",
        "escape-string-regexp": "^4.0.0",
        "eslint-scope": "^8.4.0",
        "eslint-visitor-keys": "^4.2.1",
        "espree": "^10.4.0",
        "esquery": "^1.5.0",
        "esutils": "^2.0.2",
        "fast-deep-equal": "^3.1.3",
        "file-entry-cache": "^8.0.0",
        "find-up": "^5.0.0",
        "glob-parent": "^6.0.2",
        "ignore": "^5.2.0",
        "imurmurhash": "^0.1.4",
        "is-glob": "^4.0.0",
        "json-stable-stringify-without-jsonify": "^1.0.1",
        "lodash.merge": "^4.6.2",
        "minimatch": "^3.1.5",
        "natural-compare": "^1.4.0",
        "optionator": "^0.9.3"
      },
      "bin": {
        "eslint": "bin/eslint.js"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "url": "https://eslint.org/donate"
      },
      "peerDependencies": {
        "jiti": "*"
      },
      "peerDependenciesMeta": {
        "jiti": {
          "optional": true
        }
      }
    },
    "node_modules/eslint-config-next": {
      "version": "16.2.10",
      "resolved": "https://registry.npmjs.org/eslint-config-next/-/eslint-config-next-16.2.10.tgz",
      "integrity": "sha512-HSybLOY0QKf39i4FWUqPN0xWiNDi6A6UqJmZtgDkS3zMqjXTqULvj/sueXx3cdCG0mVG+qH6k5/qdegklH1d1w==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@next/eslint-plugin-next": "16.2.10",
        "eslint-import-resolver-node": "^0.3.6",
        "eslint-import-resolver-typescript": "^3.5.2",
        "eslint-plugin-import": "^2.32.0",
        "eslint-plugin-jsx-a11y": "^6.10.0",
        "eslint-plugin-react": "^7.37.0",
        "eslint-plugin-react-hooks": "^7.0.0",
        "globals": "16.4.0",
        "typescript-eslint": "^8.46.0"
      },
      "peerDependencies": {
        "eslint": ">=9.0.0",
        "typescript": ">=3.3.1"
      },
      "peerDependenciesMeta": {
        "typescript": {
          "optional": true
        }
      }
    },
    "node_modules/eslint-import-resolver-node": {
      "version": "0.3.10",
      "resolved": "https://registry.npmjs.org/eslint-import-resolver-node/-/eslint-import-resolver-node-0.3.10.tgz",
      "integrity": "sha512-tRrKqFyCaKict5hOd244sL6EQFNycnMQnBe+j8uqGNXYzsImGbGUU4ibtoaBmv5FLwJwcFJNeg1GeVjQfbMrDQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "debug": "^3.2.7",
        "is-core-module": "^2.16.1",
        "resolve": "^2.0.0-next.6"
      }
    },
    "node_modules/eslint-import-resolver-node/node_modules/debug": {
      "version": "3.2.7",
      "resolved": "https://registry.npmjs.org/debug/-/debug-3.2.7.tgz",
      "integrity": "sha512-CFjzYYAi4ThfiQvizrFQevTTXHtnCqWfe7x1AhgEscTz6ZbLbfoLRLPugTQyBth6f8ZERVUSyWHFD/7Wu4t1XQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "ms": "^2.1.1"
      }
    },
    "node_modules/eslint-import-resolver-typescript": {
      "version": "3.10.1",
      "resolved": "https://registry.npmjs.org/eslint-import-resolver-typescript/-/eslint-import-resolver-typescript-3.10.1.tgz",
      "integrity": "sha512-A1rHYb06zjMGAxdLSkN2fXPBwuSaQ0iO5M/hdyS0Ajj1VBaRp0sPD3dn1FhME3c/JluGFbwSxyCfqdSbtQLAHQ==",
      "dev": true,
      "license": "ISC",
      "dependencies": {
        "@nolyfill/is-core-module": "1.0.39",
        "debug": "^4.4.0",
        "get-tsconfig": "^4.10.0",
        "is-bun-module": "^2.0.0",
        "stable-hash": "^0.0.5",
        "tinyglobby": "^0.2.13",
        "unrs-resolver": "^1.6.2"
      },
      "engines": {
        "node": "^14.18.0 || >=16.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/eslint-import-resolver-typescript"
      },
      "peerDependencies": {
        "eslint": "*",
        "eslint-plugin-import": "*",
        "eslint-plugin-import-x": "*"
      },
      "peerDependenciesMeta": {
        "eslint-plugin-import": {
          "optional": true
        },
        "eslint-plugin-import-x": {
          "optional": true
        }
      }
    },
    "node_modules/eslint-module-utils": {
      "version": "2.14.0",
      "resolved": "https://registry.npmjs.org/eslint-module-utils/-/eslint-module-utils-2.14.0.tgz",
      "integrity": "sha512-W2WCRZ9Dqntd+2u8jJcVMV2PKulc6RdLgUUoh/yQr3uB6lo/ZOeGx11sv60/8S4QFFKNslAlWhr9u0Ef7ZW6Ig==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "debug": "^3.2.7"
      },
      "engines": {
        "node": ">=4"
      },
      "peerDependenciesMeta": {
        "eslint": {
          "optional": true
        }
      }
    },
    "node_modules/eslint-module-utils/node_modules/debug": {
      "version": "3.2.7",
      "resolved": "https://registry.npmjs.org/debug/-/debug-3.2.7.tgz",
      "integrity": "sha512-CFjzYYAi4ThfiQvizrFQevTTXHtnCqWfe7x1AhgEscTz6ZbLbfoLRLPugTQyBth6f8ZERVUSyWHFD/7Wu4t1XQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "ms": "^2.1.1"
      }
    },
    "node_modules/eslint-plugin-import": {
      "version": "2.32.0",
      "resolved": "https://registry.npmjs.org/eslint-plugin-import/-/eslint-plugin-import-2.32.0.tgz",
      "integrity": "sha512-whOE1HFo/qJDyX4SnXzP4N6zOWn79WhnCUY/iDR0mPfQZO8wcYE4JClzI2oZrhBnnMUCBCHZhO6VQyoBU95mZA==",
      "dev": true,
      "dependencies": {
        "@rtsao/scc": "^1.1.0",
        "array-includes": "^3.1.9",
        "array.prototype.findlastindex": "^1.2.6",
        "array.prototype.flat": "^1.3.3",
        "array.prototype.flatmap": "^1.3.3",
        "debug": "^3.2.7",
        "doctrine": "^2.1.0",
        "eslint-import-resolver-node": "^0.3.9",
        "eslint-module-utils": "^2.12.1",
        "hasown": "^2.0.2",
        "is-core-module": "^2.16.1",
        "is-glob": "^4.0.3",
        "minimatch": "^3.1.2",
        "object.fromentries": "^2.0.8",
        "object.groupby": "^1.0.3",
        "object.values": "^1.2.1",
        "semver": "^6.3.1",
        "string.prototype.trimend": "^1.0.9",
        "tsconfig-paths": "^3.15.0"
      },
      "engines": {
        "node": ">=4"
      },
      "peerDependencies": {
        "eslint": "^2 || ^3 || ^4 || ^5 || ^6 || ^7.2.0 || ^8 || ^9"
      }
    },
    "node_modules/eslint-plugin-import/node_modules/debug": {
      "version": "3.2.7",
      "resolved": "https://registry.npmjs.org/debug/-/debug-3.2.7.tgz",
      "integrity": "sha512-CFjzYYAi4ThfiQvizrFQevTTXHtnCqWfe7x1AhgEscTz6ZbLbfoLRLPugTQyBth6f8ZERVUSyWHFD/7Wu4t1XQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "ms": "^2.1.1"
      }
    },
    "node_modules/eslint-plugin-jsx-a11y": {
      "version": "6.10.2",
      "resolved": "https://registry.npmjs.org/eslint-plugin-jsx-a11y/-/eslint-plugin-jsx-a11y-6.10.2.tgz",
      "integrity": "sha512-scB3nz4WmG75pV8+3eRUQOHZlNSUhFNq37xnpgRkCCELU3XMvXAxLk1eqWWyE22Ki4Q01Fnsw9BA3cJHDPgn2Q==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "aria-query": "^5.3.2",
        "array-includes": "^3.1.8",
        "array.prototype.flatmap": "^1.3.2",
        "ast-types-flow": "^0.0.8",
        "axe-core": "^4.10.0",
        "axobject-query": "^4.1.0",
        "damerau-levenshtein": "^1.0.8",
        "emoji-regex": "^9.2.2",
        "hasown": "^2.0.2",
        "jsx-ast-utils": "^3.3.5",
        "language-tags": "^1.0.9",
        "minimatch": "^3.1.2",
        "object.fromentries": "^2.0.8",
        "safe-regex-test": "^1.0.3",
        "string.prototype.includes": "^2.0.1"
      },
      "engines": {
        "node": ">=4.0"
      },
      "peerDependencies": {
        "eslint": "^3 || ^4 || ^5 || ^6 || ^7 || ^8 || ^9"
      }
    },
    "node_modules/eslint-plugin-react": {
      "version": "7.37.5",
      "resolved": "https://registry.npmjs.org/eslint-plugin-react/-/eslint-plugin-react-7.37.5.tgz",
      "integrity": "sha512-Qteup0SqU15kdocexFNAJMvCJEfa2xUKNV4CC1xsVMrIIqEy3SQ/rqyxCWNzfrd3/ldy6HMlD2e0JDVpDg2qIA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "array-includes": "^3.1.8",
        "array.prototype.findlast": "^1.2.5",
        "array.prototype.flatmap": "^1.3.3",
        "array.prototype.tosorted": "^1.1.4",
        "doctrine": "^2.1.0",
        "es-iterator-helpers": "^1.2.1",
        "estraverse": "^5.3.0",
        "hasown": "^2.0.2",
        "jsx-ast-utils": "^2.4.1 || ^3.0.0",
        "minimatch": "^3.1.2",
        "object.entries": "^1.1.9",
        "object.fromentries": "^2.0.8",
        "object.values": "^1.2.1",
        "prop-types": "^15.8.1",
        "resolve": "^2.0.0-next.5",
        "semver": "^6.3.1",
        "string.prototype.matchall": "^4.0.12",
        "string.prototype.repeat": "^1.0.0"
      },
      "engines": {
        "node": ">=4"
      },
      "peerDependencies": {
        "eslint": "^3 || ^4 || ^5 || ^6 || ^7 || ^8 || ^9.7"
      }
    },
    "node_modules/eslint-plugin-react-hooks": {
      "version": "7.1.1",
      "resolved": "https://registry.npmjs.org/eslint-plugin-react-hooks/-/eslint-plugin-react-hooks-7.1.1.tgz",
      "integrity": "sha512-f2I7Gw6JbvCexzIInuSbZpfdQ44D7iqdWX01FKLvrPgqxoE7oMj8clOfto8U6vYiz4yd5oKu39rRSVOe1zRu0g==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@babel/core": "^7.24.4",
        "@babel/parser": "^7.24.4",
        "hermes-parser": "^0.25.1",
        "zod": "^3.25.0 || ^4.0.0",
        "zod-validation-error": "^3.5.0 || ^4.0.0"
      },
      "engines": {
        "node": ">=18"
      },
      "peerDependencies": {
        "eslint": "^3.0.0 || ^4.0.0 || ^5.0.0 || ^6.0.0 || ^7.0.0 || ^8.0.0-0 || ^9.0.0 || ^10.0.0"
      }
    },
    "node_modules/eslint-scope": {
      "version": "8.4.0",
      "resolved": "https://registry.npmjs.org/eslint-scope/-/eslint-scope-8.4.0.tgz",
      "integrity": "sha512-sNXOfKCn74rt8RICKMvJS7XKV/Xk9kA7DyJr8mJik3S7Cwgy3qlkkmyS2uQB3jiJg6VNdZd/pDBJu0nvG2NlTg==",
      "dev": true,
      "license": "BSD-2-Clause",
      "dependencies": {
        "esrecurse": "^4.3.0",
        "estraverse": "^5.2.0"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "url": "https://opencollective.com/eslint"
      }
    },
    "node_modules/eslint-visitor-keys": {
      "version": "4.2.1",
      "resolved": "https://registry.npmjs.org/eslint-visitor-keys/-/eslint-visitor-keys-4.2.1.tgz",
      "integrity": "sha512-Uhdk5sfqcee/9H/rCOJikYz67o0a2Tw2hGRPOG2Y1R2dg7brRe1uG0yaNQDHu+TO/uQPF/5eCapvYSmHUjt7JQ==",
      "dev": true,
      "license": "Apache-2.0",
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "url": "https://opencollective.com/eslint"
      }
    },
    "node_modules/espree": {
      "version": "10.4.0",
      "resolved": "https://registry.npmjs.org/espree/-/espree-10.4.0.tgz",
      "integrity": "sha512-j6PAQ2uUr79PZhBjP5C5fhl8e39FmRnOjsD5lGnWrFU8i2G776tBK7+nP8KuQUTTyAZUwfQqXAgrVH5MbH9CYQ==",
      "dev": true,
      "license": "BSD-2-Clause",
      "dependencies": {
        "acorn": "^8.15.0",
        "acorn-jsx": "^5.3.2",
        "eslint-visitor-keys": "^4.2.1"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "url": "https://opencollective.com/eslint"
      }
    },
    "node_modules/esquery": {
      "version": "1.7.0",
      "resolved": "https://registry.npmjs.org/esquery/-/esquery-1.7.0.tgz",
      "integrity": "sha512-Ap6G0WQwcU/LHsvLwON1fAQX9Zp0A2Y6Y/cJBl9r/JbW90Zyg4/zbG6zzKa2OTALELarYHmKu0GhpM5EO+7T0g==",
      "dev": true,
      "license": "BSD-3-Clause",
      "dependencies": {
        "estraverse": "^5.1.0"
      },
      "engines": {
        "node": ">=0.10"
      }
    },
    "node_modules/esrecurse": {
      "version": "4.3.0",
      "resolved": "https://registry.npmjs.org/esrecurse/-/esrecurse-4.3.0.tgz",
      "integrity": "sha512-KmfKL3b6G+RXvP8N1vr3Tq1kL/oCFgn2NYXEtqP8/L3pKapUA4G8cFVaoF3SU323CD4XypR/ffioHmkti6/Tag==",
      "dev": true,
      "license": "BSD-2-Clause",
      "dependencies": {
        "estraverse": "^5.2.0"
      },
      "engines": {
        "node": ">=4.0"
      }
    },
    "node_modules/estraverse": {
      "version": "5.3.0",
      "resolved": "https://registry.npmjs.org/estraverse/-/estraverse-5.3.0.tgz",
      "integrity": "sha512-MMdARuVEQziNTeJD8DgMqmhwR11BRQ/cBP+pLtYdSTnf3MIO8fFeiINEbX36ZdNlfU/7A9f3gUw49B3oQsvwBA==",
      "dev": true,
      "license": "BSD-2-Clause",
      "engines": {
        "node": ">=4.0"
      }
    },
    "node_modules/esutils": {
      "version": "2.0.3",
      "resolved": "https://registry.npmjs.org/esutils/-/esutils-2.0.3.tgz",
      "integrity": "sha512-kVscqXk4OCp68SZ0dkgEKVi6/8ij300KBWTJq32P/dYeWTSwK41WyTxalN1eRmA5Z9UU/LX9D7FWSmV9SAYx6g==",
      "dev": true,
      "license": "BSD-2-Clause",
      "engines": {
        "node": ">=0.10.0"
      }
    },
    "node_modules/fast-deep-equal": {
      "version": "3.1.3",
      "resolved": "https://registry.npmjs.org/fast-deep-equal/-/fast-deep-equal-3.1.3.tgz",
      "integrity": "sha512-f3qQ9oQy9j2AhBe/H9VC91wLmKBCCU/gDOnKNAYG5hswO7BLKj09Hc5HYNz9cGI++xlpDCIgDaitVs03ATR84Q==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/fast-glob": {
      "version": "3.3.1",
      "resolved": "https://registry.npmjs.org/fast-glob/-/fast-glob-3.3.1.tgz",
      "integrity": "sha512-kNFPyjhh5cKjrUltxs+wFx+ZkbRaxxmZ+X0ZU31SOsxCEtP9VPgtq2teZw1DebupL5GmDaNQ6yKMMVcM41iqDg==",
      "dev": true,
      "dependencies": {
        "@nodelib/fs.stat": "^2.0.2",
        "@nodelib/fs.walk": "^1.2.3",
        "glob-parent": "^5.1.2",
        "merge2": "^1.3.0",
        "micromatch": "^4.0.4"
      },
      "engines": {
        "node": ">=8.6.0"
      }
    },
    "node_modules/fast-glob/node_modules/glob-parent": {
      "version": "5.1.2",
      "resolved": "https://registry.npmjs.org/glob-parent/-/glob-parent-5.1.2.tgz",
      "integrity": "sha512-AOIgSQCepiJYwP3ARnGx+5VnTu2HBYdzbGP45eLw1vr3zB3vZLeyed1sC9hnbcOc9/SrMyM5RPQrkGz4aS9Zow==",
      "dev": true,
      "license": "ISC",
      "dependencies": {
        "is-glob": "^4.0.1"
      },
      "engines": {
        "node": ">= 6"
      }
    },
    "node_modules/fast-json-stable-stringify": {
      "version": "2.1.0",
      "resolved": "https://registry.npmjs.org/fast-json-stable-stringify/-/fast-json-stable-stringify-2.1.0.tgz",
      "integrity": "sha512-lhd/wF+Lk98HZoTCtlVraHtfh5XYijIjalXck7saUtuanSDyLMxnHhSXEDJqHxD7msR8D0uCmqlkwjCV8xvwHw==",
      "dev": true
    },
    "node_modules/fast-levenshtein": {
      "version": "2.0.6",
      "resolved": "https://registry.npmjs.org/fast-levenshtein/-/fast-levenshtein-2.0.6.tgz",
      "integrity": "sha512-DCXu6Ifhqcks7TZKY3Hxp3y6qphY5SJZmrWMDrKcERSOXWQdMhU9Ig/PYrzyw/ul9jOIyh0N4M0tbC5hodg8dw==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/fastq": {
      "version": "1.20.1",
      "resolved": "https://registry.npmjs.org/fastq/-/fastq-1.20.1.tgz",
      "integrity": "sha512-GGToxJ/w1x32s/D2EKND7kTil4n8OVk/9mycTc4VDza13lOvpUZTGX3mFSCtV9ksdGBVzvsyAVLM6mHFThxXxw==",
      "dev": true,
      "dependencies": {
        "reusify": "^1.0.4"
      }
    },
    "node_modules/fflate": {
      "version": "0.8.3",
      "resolved": "https://registry.npmjs.org/fflate/-/fflate-0.8.3.tgz",
      "integrity": "sha512-tbZNuJrLwGUp3zshBtdy4W+ORxZuIh8a5ilyIEQDC5rY1f3U20JMry0Ll3WBzU58EZKsEuJFXhb5gwv8CsPvgA==",
      "license": "MIT"
    },
    "node_modules/file-entry-cache": {
      "version": "8.0.0",
      "resolved": "https://registry.npmjs.org/file-entry-cache/-/file-entry-cache-8.0.0.tgz",
      "integrity": "sha512-XXTUwCvisa5oacNGRP9SfNtYBNAMi+RPwBFmblZEF7N7swHYQS6/Zfk7SRwx4D5j3CH211YNRco1DEMNVfZCnQ==",
      "dev": true,
      "dependencies": {
        "flat-cache": "^4.0.0"
      },
      "engines": {
        "node": ">=16.0.0"
      }
    },
    "node_modules/fill-range": {
      "version": "7.1.1",
      "resolved": "https://registry.npmjs.org/fill-range/-/fill-range-7.1.1.tgz",
      "integrity": "sha512-YsGpe3WHLK8ZYi4tWDg2Jy3ebRz2rXowDxnld4bkQB00cc/1Zw9AWnC0i9ztDJitivtQvaI9KaLyKrc+hBW0yg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "to-regex-range": "^5.0.1"
      },
      "engines": {
        "node": ">=8"
      }
    },
    "node_modules/find-up": {
      "version": "5.0.0",
      "resolved": "https://registry.npmjs.org/find-up/-/find-up-5.0.0.tgz",
      "integrity": "sha512-78/PXT1wlLLDgTzDs7sjq9hzz0vXD+zn+7wypEe4fXQxCmdmqfGsEPQxmiCSQI3ajFV91bVSsvNtrJRiW6nGng==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "locate-path": "^6.0.0",
        "path-exists": "^4.0.0"
      },
      "engines": {
        "node": ">=10"
      },
      "funding": {
        "url": "https://github.com/sponsors/sindresorhus"
      }
    },
    "node_modules/flat-cache": {
      "version": "4.0.1",
      "resolved": "https://registry.npmjs.org/flat-cache/-/flat-cache-4.0.1.tgz",
      "integrity": "sha512-f7ccFPK3SXFHpx15UIGyRJ/FJQctuKZ0zVuN3frBo4HnK3cay9VEW0R6yPYFHC0AgqhukPzKjq22t5DmAyqGyw==",
      "dev": true,
      "dependencies": {
        "flatted": "^3.2.9",
        "keyv": "^4.5.4"
      },
      "engines": {
        "node": ">=16"
      }
    },
    "node_modules/flatted": {
      "version": "3.4.2",
      "resolved": "https://registry.npmjs.org/flatted/-/flatted-3.4.2.tgz",
      "integrity": "sha512-PjDse7RzhcPkIJwy5t7KPWQSZ9cAbzQXcafsetQoD7sOJRQlGikNbx7yZp2OotDnJyrDcbyRq3Ttb18iYOqkxA==",
      "dev": true
    },
    "node_modules/for-each": {
      "version": "0.3.5",
      "resolved": "https://registry.npmjs.org/for-each/-/for-each-0.3.5.tgz",
      "integrity": "sha512-dKx12eRCVIzqCxFGplyFKJMPvLEWgmNtUrpTiJIR5u97zEhRG8ySrtboPHZXx7daLxQVrl643cTzbab2tkQjxg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "is-callable": "^1.2.7"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/framer-motion": {
      "version": "12.42.2",
      "resolved": "https://registry.npmjs.org/framer-motion/-/framer-motion-12.42.2.tgz",
      "integrity": "sha512-5XY9luDiu0oHfHBjpDthFMh0ES+122w6p/papSJBweMkO8Sn+PW2QaEgRblQBpWFnuvZS5qvarpt/hO2pjGmnw==",
      "license": "MIT",
      "dependencies": {
        "motion-dom": "^12.42.2",
        "motion-utils": "^12.39.0",
        "tslib": "^2.4.0"
      },
      "peerDependencies": {
        "@emotion/is-prop-valid": "*",
        "react": "^18.0.0 || ^19.0.0",
        "react-dom": "^18.0.0 || ^19.0.0"
      },
      "peerDependenciesMeta": {
        "@emotion/is-prop-valid": {
          "optional": true
        },
        "react": {
          "optional": true
        },
        "react-dom": {
          "optional": true
        }
      }
    },
    "node_modules/function-bind": {
      "version": "1.1.2",
      "resolved": "https://registry.npmjs.org/function-bind/-/function-bind-1.1.2.tgz",
      "integrity": "sha512-7XHNxH7qX9xG5mIwxkhumTox/MIRNcOgDrxWsMt2pAr23WHp6MrRlN7FBSFpCpr+oVO0F744iUgR82nJMfG2SA==",
      "dev": true,
      "license": "MIT",
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/function.prototype.name": {
      "version": "1.2.0",
      "resolved": "https://registry.npmjs.org/function.prototype.name/-/function.prototype.name-1.2.0.tgz",
      "integrity": "sha512-jObKIik1P2QjPHP5nz5BaOtUlfgS0fWo8IUByNXkM+o+02sJOi94em77GwJKQSJ3gfPHdgzLNrHc1uokV4P/ew==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.9",
        "call-bound": "^1.0.4",
        "es-define-property": "^1.0.1",
        "es-errors": "^1.3.0",
        "functions-have-names": "^1.2.3",
        "has-property-descriptors": "^1.0.2",
        "hasown": "^2.0.4",
        "is-callable": "^1.2.7",
        "is-document.all": "^1.0.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/functions-have-names": {
      "version": "1.2.3",
      "resolved": "https://registry.npmjs.org/functions-have-names/-/functions-have-names-1.2.3.tgz",
      "integrity": "sha512-xckBUXyTIqT97tq2x2AMb+g163b5JFysYk0x4qxNFwbfQkmNZoiRHb6sPzI9/QV33WeuvVYBUIiD4NzNIyqaRQ==",
      "dev": true,
      "license": "MIT",
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/generator-function": {
      "version": "2.0.1",
      "resolved": "https://registry.npmjs.org/generator-function/-/generator-function-2.0.1.tgz",
      "integrity": "sha512-SFdFmIJi+ybC0vjlHN0ZGVGHc3lgE0DxPAT0djjVg+kjOnSqclqmj0KQ7ykTOLP6YxoqOvuAODGdcHJn+43q3g==",
      "dev": true,
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/gensync": {
      "version": "1.0.0-beta.2",
      "resolved": "https://registry.npmjs.org/gensync/-/gensync-1.0.0-beta.2.tgz",
      "integrity": "sha512-3hN7NaskYvMDLQY55gnW3NQ+mesEAepTqlg+VEbj7zzqEMBVNhzcGYYeqFo/TlYz6eQiFcp1HcsCZO+nGgS8zg==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=6.9.0"
      }
    },
    "node_modules/get-intrinsic": {
      "version": "1.3.0",
      "resolved": "https://registry.npmjs.org/get-intrinsic/-/get-intrinsic-1.3.0.tgz",
      "integrity": "sha512-9fSjSaos/fRIVIp+xSJlE6lfwhES7LNtKaCBIamHsjr2na1BiABJPo0mOjjz8GJDURarmCPGqaiVg5mfjb98CQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind-apply-helpers": "^1.0.2",
        "es-define-property": "^1.0.1",
        "es-errors": "^1.3.0",
        "es-object-atoms": "^1.1.1",
        "function-bind": "^1.1.2",
        "get-proto": "^1.0.1",
        "gopd": "^1.2.0",
        "has-symbols": "^1.1.0",
        "hasown": "^2.0.2",
        "math-intrinsics": "^1.1.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/get-proto": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/get-proto/-/get-proto-1.0.1.tgz",
      "integrity": "sha512-sTSfBjoXBp89JvIKIefqw7U2CCebsc74kiY6awiGogKtoSGbgjYE/G/+l9sF3MWFPNc9IcoOC4ODfKHfxFmp0g==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "dunder-proto": "^1.0.1",
        "es-object-atoms": "^1.0.0"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/get-symbol-description": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/get-symbol-description/-/get-symbol-description-1.1.0.tgz",
      "integrity": "sha512-w9UMqWwJxHNOvoNzSJ2oPF5wvYcvP7jUvYzhp67yEhTi17ZDBBC1z9pTdGuzjD+EFIqLSYRweZjqfiPzQ06Ebg==",
      "dev": true,
      "dependencies": {
        "call-bound": "^1.0.3",
        "es-errors": "^1.3.0",
        "get-intrinsic": "^1.2.6"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/get-tsconfig": {
      "version": "4.14.0",
      "resolved": "https://registry.npmjs.org/get-tsconfig/-/get-tsconfig-4.14.0.tgz",
      "integrity": "sha512-yTb+8DXzDREzgvYmh6s9vHsSVCHeC0G3PI5bEXNBHtmshPnO+S5O7qgLEOn0I5QvMy6kpZN8K1NKGyilLb93wA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "resolve-pkg-maps": "^1.0.0"
      },
      "funding": {
        "url": "https://github.com/privatenumber/get-tsconfig?sponsor=1"
      }
    },
    "node_modules/glob-parent": {
      "version": "6.0.2",
      "resolved": "https://registry.npmjs.org/glob-parent/-/glob-parent-6.0.2.tgz",
      "integrity": "sha512-XxwI8EOhVQgWp6iDL+3b0r86f4d6AX6zSU55HfB4ydCEuXLXc5FcYeOu+nnGftS4TEju/11rt4KJPTMgbfmv4A==",
      "dev": true,
      "license": "ISC",
      "dependencies": {
        "is-glob": "^4.0.3"
      },
      "engines": {
        "node": ">=10.13.0"
      }
    },
    "node_modules/globals": {
      "version": "16.4.0",
      "resolved": "https://registry.npmjs.org/globals/-/globals-16.4.0.tgz",
      "integrity": "sha512-ob/2LcVVaVGCYN+r14cnwnoDPUufjiYgSqRhiFD0Q1iI4Odora5RE8Iv1D24hAz5oMophRGkGz+yuvQmmUMnMw==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=18"
      },
      "funding": {
        "url": "https://github.com/sponsors/sindresorhus"
      }
    },
    "node_modules/globalthis": {
      "version": "1.0.4",
      "resolved": "https://registry.npmjs.org/globalthis/-/globalthis-1.0.4.tgz",
      "integrity": "sha512-DpLKbNU4WylpxJykQujfCcwYWiV/Jhm50Goo0wrVILAv5jOr9d+H+UR3PhSCD2rCCEIg0uc+G+muBTwD54JhDQ==",
      "dev": true,
      "dependencies": {
        "define-properties": "^1.2.1",
        "gopd": "^1.0.1"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/glsl-noise": {
      "version": "0.0.0",
      "resolved": "https://registry.npmjs.org/glsl-noise/-/glsl-noise-0.0.0.tgz",
      "integrity": "sha512-b/ZCF6amfAUb7dJM/MxRs7AetQEahYzJ8PtgfrmEdtw6uyGOr+ZSGtgjFm6mfsBkxJ4d2W7kg+Nlqzqvn3Bc0w==",
      "license": "MIT"
    },
    "node_modules/gopd": {
      "version": "1.2.0",
      "resolved": "https://registry.npmjs.org/gopd/-/gopd-1.2.0.tgz",
      "integrity": "sha512-ZUKRh6/kUFoAiTAtTYPZJ3hw9wNxx+BIBOijnlG9PnrJsCcSjs1wyyD6vJpaYtgnzDrKYRSqf3OO6Rfa93xsRg==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/has-bigints": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/has-bigints/-/has-bigints-1.1.0.tgz",
      "integrity": "sha512-R3pbpkcIqv2Pm3dUwgjclDRVmWpTJW2DcMzcIhEXEx1oh/CEMObMm3KLmRJOdvhM7o4uQBnwr8pzRK2sJWIqfg==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/has-flag": {
      "version": "4.0.0",
      "resolved": "https://registry.npmjs.org/has-flag/-/has-flag-4.0.0.tgz",
      "integrity": "sha512-EykJT/Q1KjTWctppgIAgfSO0tKVuZUjhgMr17kqTumMl6Afv3EISleU7qZUzoXDFTAHTDC4NOoG/ZxU3EvlMPQ==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=8"
      }
    },
    "node_modules/has-property-descriptors": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/has-property-descriptors/-/has-property-descriptors-1.0.2.tgz",
      "integrity": "sha512-55JNKuIW+vq4Ke1BjOTjM2YctQIvCT7GFzHwmfZPGo5wnrgkid0YQtnAleFSqumZm4az3n2BS+erby5ipJdgrg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "es-define-property": "^1.0.0"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/has-proto": {
      "version": "1.2.0",
      "resolved": "https://registry.npmjs.org/has-proto/-/has-proto-1.2.0.tgz",
      "integrity": "sha512-KIL7eQPfHQRC8+XluaIw7BHUwwqL19bQn4hzNgdr+1wXoU0KKj6rufu47lhY7KbJR2C6T6+PfyN0Ea7wkSS+qQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "dunder-proto": "^1.0.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/has-symbols": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/has-symbols/-/has-symbols-1.1.0.tgz",
      "integrity": "sha512-1cDNdwJ2Jaohmb3sg4OmKaMBwuC48sYni5HUw2DvsC8LjGTLK9h+eb1X6RyuOHe4hT0ULCW68iomhjUoKUqlPQ==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/has-tostringtag": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/has-tostringtag/-/has-tostringtag-1.0.2.tgz",
      "integrity": "sha512-NqADB8VjPFLM2V0VvHUewwwsw0ZWBaIdgo+ieHtK3hasLz4qeCRjYcqfB6AQrBggRKppKF8L52/VqdVsO47Dlw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "has-symbols": "^1.0.3"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/hasown": {
      "version": "2.0.4",
      "resolved": "https://registry.npmjs.org/hasown/-/hasown-2.0.4.tgz",
      "integrity": "sha512-T2UbfbBEF32wiepXIsMlTW9+dDYC6wMh/t/vYA4tuOMKqWz/n3vr1NFSxQiyP+zk2mXsoMA/i/7qV6LKut1t1A==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "function-bind": "^1.1.2"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/hermes-estree": {
      "version": "0.25.1",
      "resolved": "https://registry.npmjs.org/hermes-estree/-/hermes-estree-0.25.1.tgz",
      "integrity": "sha512-0wUoCcLp+5Ev5pDW2OriHC2MJCbwLwuRx+gAqMTOkGKJJiBCLjtrvy4PWUGn6MIVefecRpzoOZ/UV6iGdOr+Cw==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/hermes-parser": {
      "version": "0.25.1",
      "resolved": "https://registry.npmjs.org/hermes-parser/-/hermes-parser-0.25.1.tgz",
      "integrity": "sha512-6pEjquH3rqaI6cYAXYPcz9MS4rY6R4ngRgrgfDshRptUZIc3lw0MCIJIGDj9++mfySOuPTHB4nrSW99BCvOPIA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "hermes-estree": "0.25.1"
      }
    },
    "node_modules/hls.js": {
      "version": "1.6.16",
      "resolved": "https://registry.npmjs.org/hls.js/-/hls.js-1.6.16.tgz",
      "integrity": "sha512-VSIRpLfRwlAAdGL4wiTucx2ScRipo0ed1FBatWkyt832jC4CReKstga6yIhYVwGu9LOBjuX9wzmRMeQdBJtzEA==",
      "license": "Apache-2.0"
    },
    "node_modules/ieee754": {
      "version": "1.2.1",
      "resolved": "https://registry.npmjs.org/ieee754/-/ieee754-1.2.1.tgz",
      "integrity": "sha512-dcyqhDvX1C46lXZcVqCpK+FtMRQVdIMN6/Df5js2zouUsqG7I6sFxitIC+7KYK29KdXOLHdu9zL4sFnoVQnqaA==",
      "funding": [
        {
          "type": "github",
          "url": "https://github.com/sponsors/feross"
        },
        {
          "type": "patreon",
          "url": "https://www.patreon.com/feross"
        },
        {
          "type": "consulting",
          "url": "https://feross.org/support"
        }
      ]
    },
    "node_modules/ignore": {
      "version": "5.3.2",
      "resolved": "https://registry.npmjs.org/ignore/-/ignore-5.3.2.tgz",
      "integrity": "sha512-hsBTNUqQTDwkWtcdYI2i06Y/nUBEsNEDJKjWdigLvegy8kDuJAS8uRlpkkcQpyEXL0Z/pjDy5HBmMjRCJ2gq+g==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 4"
      }
    },
    "node_modules/immediate": {
      "version": "3.0.6",
      "resolved": "https://registry.npmjs.org/immediate/-/immediate-3.0.6.tgz",
      "integrity": "sha512-XXOFtyqDjNDAQxVfYxuF7g9Il/IbWmmlQg2MYKOH8ExIT1qg6xc4zyS3HaEEATgs1btfzxq15ciUiY7gjSXRGQ==",
      "license": "MIT"
    },
    "node_modules/import-fresh": {
      "version": "3.3.1",
      "resolved": "https://registry.npmjs.org/import-fresh/-/import-fresh-3.3.1.tgz",
      "integrity": "sha512-TR3KfrTZTYLPB6jUjfx6MF9WcWrHL9su5TObK4ZkYgBdWKPOFoSoQIdEuTuR82pmtxH2spWG9h6etwfr1pLBqQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "parent-module": "^1.0.0",
        "resolve-from": "^4.0.0"
      },
      "engines": {
        "node": ">=6"
      },
      "funding": {
        "url": "https://github.com/sponsors/sindresorhus"
      }
    },
    "node_modules/imurmurhash": {
      "version": "0.1.4",
      "resolved": "https://registry.npmjs.org/imurmurhash/-/imurmurhash-0.1.4.tgz",
      "integrity": "sha512-JmXMZ6wuvDmLiHEml9ykzqO6lwFbof0GG4IkcGaENdCRDDmMVnny7s5HsIgHCbaq0w2MyPhDqkhTUgS2LU2PHA==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=0.8.19"
      }
    },
    "node_modules/internal-slot": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/internal-slot/-/internal-slot-1.1.0.tgz",
      "integrity": "sha512-4gd7VpWNQNB4UKKCFFVcp1AVv+FMOgs9NKzjHKusc8jTMhd5eL1NqQqOpE0KzMds804/yHlglp3uxgluOqAPLw==",
      "dev": true,
      "dependencies": {
        "es-errors": "^1.3.0",
        "hasown": "^2.0.2",
        "side-channel": "^1.1.0"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/is-array-buffer": {
      "version": "3.0.5",
      "resolved": "https://registry.npmjs.org/is-array-buffer/-/is-array-buffer-3.0.5.tgz",
      "integrity": "sha512-DDfANUiiG2wC1qawP66qlTugJeL5HyzMpfr8lLK+jMQirGzNod0B12cFB/9q838Ru27sBwfw78/rdoU7RERz6A==",
      "dev": true,
      "dependencies": {
        "call-bind": "^1.0.8",
        "call-bound": "^1.0.3",
        "get-intrinsic": "^1.2.6"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-async-function": {
      "version": "2.1.1",
      "resolved": "https://registry.npmjs.org/is-async-function/-/is-async-function-2.1.1.tgz",
      "integrity": "sha512-9dgM/cZBnNvjzaMYHVoxxfPj2QXt22Ev7SuuPrs+xav0ukGB0S6d4ydZdEiM48kLx5kDV+QBPrpVnFyefL8kkQ==",
      "dev": true,
      "dependencies": {
        "async-function": "^1.0.0",
        "call-bound": "^1.0.3",
        "get-proto": "^1.0.1",
        "has-tostringtag": "^1.0.2",
        "safe-regex-test": "^1.1.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-bigint": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/is-bigint/-/is-bigint-1.1.0.tgz",
      "integrity": "sha512-n4ZT37wG78iz03xPRKJrHTdZbe3IicyucEtdRsV5yglwc3GyUfbAfpSeD0FJ41NbUNSt5wbhqfp1fS+BgnvDFQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "has-bigints": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-boolean-object": {
      "version": "1.2.2",
      "resolved": "https://registry.npmjs.org/is-boolean-object/-/is-boolean-object-1.2.2.tgz",
      "integrity": "sha512-wa56o2/ElJMYqjCjGkXri7it5FbebW5usLw/nPmCMs5DeZ7eziSYZhSmPRn0txqeW4LnAmQQU7FgqLpsEFKM4A==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.3",
        "has-tostringtag": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-bun-module": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/is-bun-module/-/is-bun-module-2.0.0.tgz",
      "integrity": "sha512-gNCGbnnnnFAUGKeZ9PdbyeGYJqewpmc2aKHUEMO5nQPWU9lOmv7jcmQIv+qHD8fXW6W7qfuCwX4rY9LNRjXrkQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "semver": "^7.7.1"
      }
    },
    "node_modules/is-bun-module/node_modules/semver": {
      "version": "7.8.5",
      "resolved": "https://registry.npmjs.org/semver/-/semver-7.8.5.tgz",
      "integrity": "sha512-Y7/KDsb8LjooZpwaqGyulO6DQlksgCncchHGk+sZIY4SBvUocMBEFH5Ur1fI4dV+Jvl0w6cjvucaIi40puRioA==",
      "dev": true,
      "license": "ISC",
      "bin": {
        "semver": "bin/semver.js"
      },
      "engines": {
        "node": ">=10"
      }
    },
    "node_modules/is-callable": {
      "version": "1.2.7",
      "resolved": "https://registry.npmjs.org/is-callable/-/is-callable-1.2.7.tgz",
      "integrity": "sha512-1BC0BVFhS/p0qtw6enp8e+8OD0UrK0oFLztSjNzhcKA3WDuJxxAPXzPuPtKkjEY9UUoEWlX/8fgKeu2S8i9JTA==",
      "dev": true,
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-core-module": {
      "version": "2.16.2",
      "resolved": "https://registry.npmjs.org/is-core-module/-/is-core-module-2.16.2.tgz",
      "integrity": "sha512-evOr8xfXKxE6qSR0hSXL2r3sd7ALj8+7jQEUvPYcm5sgZFdJ+AYzT6yNmJenvIYQBgIGwfwz08sL8zoL7yq2BA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "hasown": "^2.0.3"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-data-view": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/is-data-view/-/is-data-view-1.0.2.tgz",
      "integrity": "sha512-RKtWF8pGmS87i2D6gqQu/l7EYRlVdfzemCJN/P3UOs//x1QE7mfhvzHIApBTRf7axvT6DMGwSwBXYCT0nfB9xw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.2",
        "get-intrinsic": "^1.2.6",
        "is-typed-array": "^1.1.13"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-date-object": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/is-date-object/-/is-date-object-1.1.0.tgz",
      "integrity": "sha512-PwwhEakHVKTdRNVOw+/Gyh0+MzlCl4R6qKvkhuvLtPMggI1WAHt9sOwZxQLSGpUaDnrdyDsomoRgNnCfKNSXXg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.2",
        "has-tostringtag": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-document.all": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/is-document.all/-/is-document.all-1.0.0.tgz",
      "integrity": "sha512-+XSoyS05OdBbhFuELhgTCpFNHkpBOJqtsZfUFFpe5QTw+9Sjbh8zitxhQkYAo6wV7e1Vb8cAPvpCk9jGam/82g==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.4"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-extglob": {
      "version": "2.1.1",
      "resolved": "https://registry.npmjs.org/is-extglob/-/is-extglob-2.1.1.tgz",
      "integrity": "sha512-SbKbANkN603Vi4jEZv49LeVJMn4yGwsbzZworEoyEiutsN3nJYdbO36zfhGJ6QEDpOZIFkDtnq5JRxmvl3jsoQ==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=0.10.0"
      }
    },
    "node_modules/is-finalizationregistry": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/is-finalizationregistry/-/is-finalizationregistry-1.1.1.tgz",
      "integrity": "sha512-1pC6N8qWJbWoPtEjgcL2xyhQOP491EQjeUo3qTKcmV8YSDDJrOepfG8pcC7h/QgnQHYSv0mJ3Z/ZWxmatVrysg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.3"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-generator-function": {
      "version": "1.1.2",
      "resolved": "https://registry.npmjs.org/is-generator-function/-/is-generator-function-1.1.2.tgz",
      "integrity": "sha512-upqt1SkGkODW9tsGNG5mtXTXtECizwtS2kA161M+gJPc1xdb/Ax629af6YrTwcOeQHbewrPNlE5Dx7kzvXTizA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.4",
        "generator-function": "^2.0.0",
        "get-proto": "^1.0.1",
        "has-tostringtag": "^1.0.2",
        "safe-regex-test": "^1.1.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-glob": {
      "version": "4.0.3",
      "resolved": "https://registry.npmjs.org/is-glob/-/is-glob-4.0.3.tgz",
      "integrity": "sha512-xelSayHH36ZgE7ZWhli7pW34hNbNl8Ojv5KVmkJD4hBdD3th8Tfk9vYasLM+mXWOZhFkgZfxhLSnrwRr4elSSg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "is-extglob": "^2.1.1"
      },
      "engines": {
        "node": ">=0.10.0"
      }
    },
    "node_modules/is-map": {
      "version": "2.0.3",
      "resolved": "https://registry.npmjs.org/is-map/-/is-map-2.0.3.tgz",
      "integrity": "sha512-1Qed0/Hr2m+YqxnM09CjA2d/i6YZNfF6R2oRAOj36eUdS6qIV/huPJNSEpKbupewFs+ZsJlxsjjPbc0/afW6Lw==",
      "dev": true,
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-negative-zero": {
      "version": "2.0.3",
      "resolved": "https://registry.npmjs.org/is-negative-zero/-/is-negative-zero-2.0.3.tgz",
      "integrity": "sha512-5KoIu2Ngpyek75jXodFvnafB6DJgr3u8uuK0LEZJjrU19DrMD3EVERaR8sjz8CCGgpZvxPl9SuE1GMVPFHx1mw==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-number": {
      "version": "7.0.0",
      "resolved": "https://registry.npmjs.org/is-number/-/is-number-7.0.0.tgz",
      "integrity": "sha512-41Cifkg6e8TylSpdtTpeLVMqvSBEVzTttHvERD741+pnZ8ANv0004MRL43QKPDlK9cGvNp6NZWZUBlbGXYxxng==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=0.12.0"
      }
    },
    "node_modules/is-number-object": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/is-number-object/-/is-number-object-1.1.1.tgz",
      "integrity": "sha512-lZhclumE1G6VYD8VHe35wFaIif+CTy5SJIi5+3y4psDgWu4wPDoBhF8NxUOinEc7pHgiTsT6MaBb92rKhhD+Xw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.3",
        "has-tostringtag": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-promise": {
      "version": "2.2.2",
      "resolved": "https://registry.npmjs.org/is-promise/-/is-promise-2.2.2.tgz",
      "integrity": "sha512-+lP4/6lKUBfQjZ2pdxThZvLUAafmZb8OAxFb8XXtiQmS35INgr85hdOGoEs124ez1FCnZJt6jau/T+alh58QFQ==",
      "license": "MIT"
    },
    "node_modules/is-regex": {
      "version": "1.2.1",
      "resolved": "https://registry.npmjs.org/is-regex/-/is-regex-1.2.1.tgz",
      "integrity": "sha512-MjYsKHO5O7mCsmRGxWcLWheFqN9DJ/2TmngvjKXihe6efViPqc274+Fx/4fYj/r03+ESvBdTXK0V6tA3rgez1g==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.2",
        "gopd": "^1.2.0",
        "has-tostringtag": "^1.0.2",
        "hasown": "^2.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-set": {
      "version": "2.0.3",
      "resolved": "https://registry.npmjs.org/is-set/-/is-set-2.0.3.tgz",
      "integrity": "sha512-iPAjerrse27/ygGLxw+EBR9agv9Y6uLeYVJMu+QNCoouJ1/1ri0mGrcWpfCqFZuzzx3WjtwxG098X+n4OuRkPg==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-shared-array-buffer": {
      "version": "1.0.4",
      "resolved": "https://registry.npmjs.org/is-shared-array-buffer/-/is-shared-array-buffer-1.0.4.tgz",
      "integrity": "sha512-ISWac8drv4ZGfwKl5slpHG9OwPNty4jOWPRIhBpxOoD+hqITiwuipOQ2bNthAzwA3B4fIjO4Nln74N0S9byq8A==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.3"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-string": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/is-string/-/is-string-1.1.1.tgz",
      "integrity": "sha512-BtEeSsoaQjlSPBemMQIrY1MY0uM6vnS1g5fmufYOtnxLGUZM2178PKbhsk7Ffv58IX+ZtcvoGwccYsh0PglkAA==",
      "dev": true,
      "dependencies": {
        "call-bound": "^1.0.3",
        "has-tostringtag": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-symbol": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/is-symbol/-/is-symbol-1.1.1.tgz",
      "integrity": "sha512-9gGx6GTtCQM73BgmHQXfDmLtfjjTUDSyoxTCbp5WtoixAhfgsDirWIcVQ/IHpvI5Vgd5i/J5F7B9cN/WlVbC/w==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.2",
        "has-symbols": "^1.1.0",
        "safe-regex-test": "^1.1.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-typed-array": {
      "version": "1.1.15",
      "resolved": "https://registry.npmjs.org/is-typed-array/-/is-typed-array-1.1.15.tgz",
      "integrity": "sha512-p3EcsicXjit7SaskXHs1hA91QxgTw46Fv6EFKKGS5DRFLD8yKnohjF3hxoju94b/OcMZoQukzpPpBE9uLVKzgQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "which-typed-array": "^1.1.16"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-weakmap": {
      "version": "2.0.2",
      "resolved": "https://registry.npmjs.org/is-weakmap/-/is-weakmap-2.0.2.tgz",
      "integrity": "sha512-K5pXYOm9wqY1RgjpL3YTkF39tni1XajUIkawTLUo9EZEVUFga5gSQJF8nNS7ZwJQ02y+1YCNYcMh+HIf1ZqE+w==",
      "dev": true,
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-weakref": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/is-weakref/-/is-weakref-1.1.1.tgz",
      "integrity": "sha512-6i9mGWSlqzNMEqpCp93KwRS1uUOodk2OJ6b+sq7ZPDSy2WuI5NFIxp/254TytR8ftefexkWn5xNiHUNpPOfSew==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.3"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/is-weakset": {
      "version": "2.0.4",
      "resolved": "https://registry.npmjs.org/is-weakset/-/is-weakset-2.0.4.tgz",
      "integrity": "sha512-mfcwb6IzQyOKTs84CQMrOwW4gQcaTOAWJ0zzJCl2WSPDrWk/OzDaImWFH3djXhb24g4eudZfLRozAvPGw4d9hQ==",
      "dev": true,
      "dependencies": {
        "call-bound": "^1.0.3",
        "get-intrinsic": "^1.2.6"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/isarray": {
      "version": "2.0.5",
      "resolved": "https://registry.npmjs.org/isarray/-/isarray-2.0.5.tgz",
      "integrity": "sha512-xHjhDr3cNBK0BzdUJSPXZntQUx/mwMS5Rw4A7lPJ90XGAO6ISP/ePDNuo0vhqOZU+UD5JoodwCAAoZQd3FeAKw==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/isexe": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/isexe/-/isexe-2.0.0.tgz",
      "integrity": "sha512-RHxMLp9lnKHGHRng9QFhRCMbYAcVpn69smSGcq3f36xjgVVWThj4qqLbTLlq7Ssj8B+fIQ1EuCEGI2lKsyQeIw==",
      "license": "ISC"
    },
    "node_modules/iterator.prototype": {
      "version": "1.1.5",
      "resolved": "https://registry.npmjs.org/iterator.prototype/-/iterator.prototype-1.1.5.tgz",
      "integrity": "sha512-H0dkQoCa3b2VEeKQBOxFph+JAbcrQdE7KC0UkqwpLmv2EC4P41QXP+rqo9wYodACiG5/WM5s9oDApTU8utwj9g==",
      "dev": true,
      "dependencies": {
        "define-data-property": "^1.1.4",
        "es-object-atoms": "^1.0.0",
        "get-intrinsic": "^1.2.6",
        "get-proto": "^1.0.0",
        "has-symbols": "^1.1.0",
        "set-function-name": "^2.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/its-fine": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/its-fine/-/its-fine-2.0.0.tgz",
      "integrity": "sha512-KLViCmWx94zOvpLwSlsx6yOCeMhZYaxrJV87Po5k/FoZzcPSahvK5qJ7fYhS61sZi5ikmh2S3Hz55A2l3U69ng==",
      "dependencies": {
        "@types/react-reconciler": "^0.28.9"
      },
      "peerDependencies": {
        "react": "^19.0.0"
      }
    },
    "node_modules/js-tokens": {
      "version": "4.0.0",
      "resolved": "https://registry.npmjs.org/js-tokens/-/js-tokens-4.0.0.tgz",
      "integrity": "sha512-RdJUflcE3cUzKiMqQgsCu06FPu9UdIJO0beYbPhHN4k6apgJtifcoCtT9bcxOpYBtpD2kCM6Sbzg4CausW/PKQ==",
      "dev": true
    },
    "node_modules/js-yaml": {
      "version": "4.3.0",
      "resolved": "https://registry.npmjs.org/js-yaml/-/js-yaml-4.3.0.tgz",
      "integrity": "sha512-1td788aAnnZ5qs7V2QIRl1owjtYpbKt749Y3xauqQgwIIGF/xXWz1wMTEBx5O3LK3lXLVuqXPdPxj2BoFHaW9Q==",
      "dev": true,
      "funding": [
        {
          "type": "github",
          "url": "https://github.com/sponsors/puzrin"
        },
        {
          "type": "github",
          "url": "https://github.com/sponsors/nodeca"
        }
      ],
      "dependencies": {
        "argparse": "^2.0.1"
      },
      "bin": {
        "js-yaml": "bin/js-yaml.js"
      }
    },
    "node_modules/jsesc": {
      "version": "3.1.0",
      "resolved": "https://registry.npmjs.org/jsesc/-/jsesc-3.1.0.tgz",
      "integrity": "sha512-/sM3dO2FOzXjKQhJuo0Q173wf2KOo8t4I8vHy6lF9poUp7bKT0/NHE8fPX23PwfhnykfqnC2xRxOnVw5XuGIaA==",
      "dev": true,
      "license": "MIT",
      "bin": {
        "jsesc": "bin/jsesc"
      },
      "engines": {
        "node": ">=6"
      }
    },
    "node_modules/json-buffer": {
      "version": "3.0.1",
      "resolved": "https://registry.npmjs.org/json-buffer/-/json-buffer-3.0.1.tgz",
      "integrity": "sha512-4bV5BfR2mqfQTJm+V5tPPdf+ZpuhiIvTuAB5g8kcrXOZpTT/QwwVRWBywX1ozr6lEuPdbHxwaJlm9G6mI2sfSQ==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/json-schema-traverse": {
      "version": "0.4.1",
      "resolved": "https://registry.npmjs.org/json-schema-traverse/-/json-schema-traverse-0.4.1.tgz",
      "integrity": "sha512-xbbCH5dCYU5T8LcEhhuh7HJ88HXuW3qsI3Y0zOZFKfZEHcpWiHU/Jxzk629Brsab/mMiHQti9wMP+845RPe3Vg==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/json-stable-stringify-without-jsonify": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/json-stable-stringify-without-jsonify/-/json-stable-stringify-without-jsonify-1.0.1.tgz",
      "integrity": "sha512-Bdboy+l7tA3OGW6FjyFHWkP5LuByj1Tk33Ljyq0axyzdk9//JSi2u3fP1QSmd1KNwq6VOKYGlAu87CisVir6Pw==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/json5": {
      "version": "2.2.3",
      "resolved": "https://registry.npmjs.org/json5/-/json5-2.2.3.tgz",
      "integrity": "sha512-XmOWe7eyHYH14cLdVPoyg+GOH3rYX++KpzrylJwSW98t3Nk+U8XOl8FWKOgwtzdb8lXGf6zYwDUzeHMWfxasyg==",
      "dev": true,
      "license": "MIT",
      "bin": {
        "json5": "lib/cli.js"
      },
      "engines": {
        "node": ">=6"
      }
    },
    "node_modules/jsx-ast-utils": {
      "version": "3.3.5",
      "resolved": "https://registry.npmjs.org/jsx-ast-utils/-/jsx-ast-utils-3.3.5.tgz",
      "integrity": "sha512-ZZow9HBI5O6EPgSJLUb8n2NKgmVWTwCvHGwFuJlMjvLFqlGG6pjirPhtdsseaLZjSibD8eegzmYpUZwoIlj2cQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "array-includes": "^3.1.6",
        "array.prototype.flat": "^1.3.1",
        "object.assign": "^4.1.4",
        "object.values": "^1.1.6"
      },
      "engines": {
        "node": ">=4.0"
      }
    },
    "node_modules/keyv": {
      "version": "4.5.4",
      "resolved": "https://registry.npmjs.org/keyv/-/keyv-4.5.4.tgz",
      "integrity": "sha512-oxVHkHR/EJf2CNXnWxRLW6mg7JyCCUcG0DtEGmL2ctUo1PNTin1PUil+r/+4r5MpVgC/fn1kjsx7mjSujKqIpw==",
      "dev": true,
      "dependencies": {
        "json-buffer": "3.0.1"
      }
    },
    "node_modules/language-subtag-registry": {
      "version": "0.3.23",
      "resolved": "https://registry.npmjs.org/language-subtag-registry/-/language-subtag-registry-0.3.23.tgz",
      "integrity": "sha512-0K65Lea881pHotoGEa5gDlMxt3pctLi2RplBb7Ezh4rRdLEOtgi7n4EwK9lamnUCkKBqaeKRVebTq6BAxSkpXQ==",
      "dev": true
    },
    "node_modules/language-tags": {
      "version": "1.0.9",
      "resolved": "https://registry.npmjs.org/language-tags/-/language-tags-1.0.9.tgz",
      "integrity": "sha512-MbjN408fEndfiQXbFQ1vnd+1NoLDsnQW41410oQBXiyXDMYH5z505juWa4KUE1LqxRC7DgOgZDbKLxHIwm27hA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "language-subtag-registry": "^0.3.20"
      },
      "engines": {
        "node": ">=0.10"
      }
    },
    "node_modules/levn": {
      "version": "0.4.1",
      "resolved": "https://registry.npmjs.org/levn/-/levn-0.4.1.tgz",
      "integrity": "sha512-+bT2uH4E5LGE7h/n3evcS/sQlJXCpIp6ym8OWJ5eV6+67Dsql/LaaT7qJBAt2rzfoa/5QBGBhxDix1dMt2kQKQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "prelude-ls": "^1.2.1",
        "type-check": "~0.4.0"
      },
      "engines": {
        "node": ">= 0.8.0"
      }
    },
    "node_modules/lie": {
      "version": "3.3.0",
      "resolved": "https://registry.npmjs.org/lie/-/lie-3.3.0.tgz",
      "integrity": "sha512-UaiMJzeWRlEujzAuw5LokY1L5ecNQYZKfmyZ9L7wDHb/p5etKaxXhohBcrw0EYby+G/NA52vRSN4N39dxHAIwQ==",
      "license": "MIT",
      "dependencies": {
        "immediate": "~3.0.5"
      }
    },
    "node_modules/locate-path": {
      "version": "6.0.0",
      "resolved": "https://registry.npmjs.org/locate-path/-/locate-path-6.0.0.tgz",
      "integrity": "sha512-iPZK6eYjbxRu3uB4/WZ3EsEIMJFMqAoopl3R+zuq0UjcAm/MO6KCweDgPfP3elTztoKP3KtnVHxTn2NHBSDVUw==",
      "dev": true,
      "dependencies": {
        "p-locate": "^5.0.0"
      },
      "engines": {
        "node": ">=10"
      },
      "funding": {
        "url": "https://github.com/sponsors/sindresorhus"
      }
    },
    "node_modules/lodash.merge": {
      "version": "4.6.2",
      "resolved": "https://registry.npmjs.org/lodash.merge/-/lodash.merge-4.6.2.tgz",
      "integrity": "sha512-0KpjqXRVvrYyCsX1swR/XTK0va6VQkQM6MNo7PqW77ByjAhoARA8EfrP1N4+KlKj8YS0ZUCtRT/YUuhyYDujIQ==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/loose-envify": {
      "version": "1.4.0",
      "resolved": "https://registry.npmjs.org/loose-envify/-/loose-envify-1.4.0.tgz",
      "integrity": "sha512-lyuxPGr/Wfhrlem2CL/UcnUc1zcqKAImBDzukY7Y5F/yQiNdko6+fRLevlw1HgMySw7f611UIY408EtxRSoK3Q==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "js-tokens": "^3.0.0 || ^4.0.0"
      },
      "bin": {
        "loose-envify": "cli.js"
      }
    },
    "node_modules/lru-cache": {
      "version": "5.1.1",
      "resolved": "https://registry.npmjs.org/lru-cache/-/lru-cache-5.1.1.tgz",
      "integrity": "sha512-KpNARQA3Iwv+jTA0utUVVbrh+Jlrr1Fv0e56GGzAFOXN7dk/FviaDW8LHmK52DlcH4WP2n6gI8vN1aesBFgo9w==",
      "dev": true,
      "license": "ISC",
      "dependencies": {
        "yallist": "^3.0.2"
      }
    },
    "node_modules/lucide-react": {
      "version": "1.25.0",
      "resolved": "https://registry.npmjs.org/lucide-react/-/lucide-react-1.25.0.tgz",
      "integrity": "sha512-/mdJTRbiwcLOQ1NZZK1amZF9rIZyvO18D6r9TngE6TG1NmqHgFuT4eE7Xrkm9UsXMbBJD1NlfwHVltCDWHrOTw==",
      "license": "ISC",
      "peerDependencies": {
        "react": "^16.5.1 || ^17.0.0 || ^18.0.0 || ^19.0.0"
      }
    },
    "node_modules/maath": {
      "version": "0.10.8",
      "resolved": "https://registry.npmjs.org/maath/-/maath-0.10.8.tgz",
      "integrity": "sha512-tRvbDF0Pgqz+9XUa4jjfgAQ8/aPKmQdWXilFu2tMy4GWj4NOsx99HlULO4IeREfbO3a0sA145DZYyvXPkybm0g==",
      "peerDependencies": {
        "@types/three": ">=0.134.0",
        "three": ">=0.134.0"
      }
    },
    "node_modules/math-intrinsics": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/math-intrinsics/-/math-intrinsics-1.1.0.tgz",
      "integrity": "sha512-/IXtbwEk5HTPyEwyKX6hGkYXxM9nbj64B+ilVJnC/R6B0pH5G4V3b0pVbL7DBj4tkhBAppbQUlf6F6Xl9LHu1g==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/merge2": {
      "version": "1.4.1",
      "resolved": "https://registry.npmjs.org/merge2/-/merge2-1.4.1.tgz",
      "integrity": "sha512-8q7VEgMJW4J8tcfVPy8g09NcQwZdbwFEqhe/WZkoIzjn/3TGDwtOCYtXGxA3O8tPzpczCCDgv+P2P5y00ZJOOg==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 8"
      }
    },
    "node_modules/meshline": {
      "version": "3.3.1",
      "resolved": "https://registry.npmjs.org/meshline/-/meshline-3.3.1.tgz",
      "integrity": "sha512-/TQj+JdZkeSUOl5Mk2J7eLcYTLiQm2IDzmlSvYm7ov15anEcDJ92GHqqazxTSreeNgfnYu24kiEvvv0WlbCdFQ==",
      "peerDependencies": {
        "three": ">=0.137"
      }
    },
    "node_modules/meshoptimizer": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/meshoptimizer/-/meshoptimizer-1.1.1.tgz",
      "integrity": "sha512-oRFNWJRDA/WTrVj7NWvqa5HqE1t9MYDj2VaWirQCzCCrAd2GHrqR/sQezCxiWATPNlKTcRaPRHPJwIRoPBAp5g==",
      "license": "MIT"
    },
    "node_modules/micromatch": {
      "version": "4.0.8",
      "resolved": "https://registry.npmjs.org/micromatch/-/micromatch-4.0.8.tgz",
      "integrity": "sha512-PXwfBhYu0hBCPw8Dn0E+WDYb7af3dSLVWKi3HGv84IdF4TyFoC0ysxFd0Goxw7nSv4T/PzEJQxsYsEiFCKo2BA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "braces": "^3.0.3",
        "picomatch": "^2.3.1"
      },
      "engines": {
        "node": ">=8.6"
      }
    },
    "node_modules/minimatch": {
      "version": "3.1.5",
      "resolved": "https://registry.npmjs.org/minimatch/-/minimatch-3.1.5.tgz",
      "integrity": "sha512-VgjWUsnnT6n+NUk6eZq77zeFdpW2LWDzP6zFGrCbHXiYNul5Dzqk2HHQ5uFH2DNW5Xbp8+jVzaeNt94ssEEl4w==",
      "dev": true,
      "license": "ISC",
      "dependencies": {
        "brace-expansion": "^1.1.7"
      },
      "engines": {
        "node": "*"
      }
    },
    "node_modules/minimist": {
      "version": "1.2.8",
      "resolved": "https://registry.npmjs.org/minimist/-/minimist-1.2.8.tgz",
      "integrity": "sha512-2yyAR8qBkN3YuheJanUpWC5U3bb5osDywNB8RzDVlDwDHbocAJveqqj1u8+SVD7jkWT4yvsHCpWqqWqAxb0zCA==",
      "dev": true,
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/motion": {
      "version": "12.42.2",
      "resolved": "https://registry.npmjs.org/motion/-/motion-12.42.2.tgz",
      "integrity": "sha512-Atvv11yUKIid41cVrRBDVX5m8tF8kNpExRSlbpt6APClhDjtwQssgFHhQzejxw7/7YYbjHSPKBVbHo05BuJT5Q==",
      "dependencies": {
        "framer-motion": "^12.42.2",
        "tslib": "^2.4.0"
      },
      "peerDependencies": {
        "@emotion/is-prop-valid": "*",
        "react": "^18.0.0 || ^19.0.0",
        "react-dom": "^18.0.0 || ^19.0.0"
      },
      "peerDependenciesMeta": {
        "@emotion/is-prop-valid": {
          "optional": true
        },
        "react": {
          "optional": true
        },
        "react-dom": {
          "optional": true
        }
      }
    },
    "node_modules/motion-dom": {
      "version": "12.42.2",
      "resolved": "https://registry.npmjs.org/motion-dom/-/motion-dom-12.42.2.tgz",
      "integrity": "sha512-5gIMWLp/PycBtJRJWRgjxke5n8dlvkSn2DrYW+tr3XcqAZY1xZh6BJyooJXCM8wdfM7wfMjkBJNLge1CKPUIRA==",
      "dependencies": {
        "motion-utils": "^12.39.0"
      }
    },
    "node_modules/motion-utils": {
      "version": "12.39.0",
      "resolved": "https://registry.npmjs.org/motion-utils/-/motion-utils-12.39.0.tgz",
      "integrity": "sha512-8nadJAJjTtqRkmRF36FoJTrywK9nnFmnPwnSMyxaOCU7GDjN9RTMJIxx9De8ErM+vpPhMccr/6fo5WciyQLnMQ==",
      "license": "MIT"
    },
    "node_modules/ms": {
      "version": "2.1.3",
      "resolved": "https://registry.npmjs.org/ms/-/ms-2.1.3.tgz",
      "integrity": "sha512-6FlzubTLZG3J2a/NVCAleEhjzq5oxgHyaCU9yYXvcLsvoVaHJq/s5xXI6/XXP6tz7R9xAOtHnSO/tXtF3WRTlA==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/nanoid": {
      "version": "3.3.16",
      "resolved": "https://registry.npmjs.org/nanoid/-/nanoid-3.3.16.tgz",
      "integrity": "sha512-bzlKTyNJ7+LdGIIwy8ijFpIqEQIvafahV7eYykJ8Cvh42EdJeODoJ6gUJXpQJvej1BddH8OqTXZNE/KfbWAu8Q==",
      "funding": [
        {
          "type": "github",
          "url": "https://github.com/sponsors/ai"
        }
      ],
      "bin": {
        "nanoid": "bin/nanoid.cjs"
      },
      "engines": {
        "node": "^10 || ^12 || ^13.7 || ^14 || >=15.0.1"
      }
    },
    "node_modules/napi-postinstall": {
      "version": "0.3.4",
      "resolved": "https://registry.npmjs.org/napi-postinstall/-/napi-postinstall-0.3.4.tgz",
      "integrity": "sha512-PHI5f1O0EP5xJ9gQmFGMS6IZcrVvTjpXjz7Na41gTE7eE2hK11lg04CECCYEEjdc17EV4DO+fkGEtt7TpTaTiQ==",
      "dev": true,
      "license": "MIT",
      "bin": {
        "napi-postinstall": "lib/cli.js"
      },
      "engines": {
        "node": "^12.20.0 || ^14.18.0 || >=16.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/napi-postinstall"
      }
    },
    "node_modules/natural-compare": {
      "version": "1.4.0",
      "resolved": "https://registry.npmjs.org/natural-compare/-/natural-compare-1.4.0.tgz",
      "integrity": "sha512-OWND8ei3VtNC9h7V60qff3SVobHr996CTwgxubgyQYEpg290h9J0buyECNNJexkFm5sOajh5G116RYA1c8ZMSw==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/next": {
      "version": "16.2.10",
      "resolved": "https://registry.npmjs.org/next/-/next-16.2.10.tgz",
      "integrity": "sha512-2som5AVXb3kE6Yjine3/mNbBayYF58eguBWIVVUdr1y/L426xyVEgYxgBG+1QC34P2x5E+tcDup6XkuOAX3dCA==",
      "dependencies": {
        "@next/env": "16.2.10",
        "@swc/helpers": "0.5.15",
        "baseline-browser-mapping": "^2.9.19",
        "caniuse-lite": "^1.0.30001579",
        "postcss": "8.4.31",
        "styled-jsx": "5.1.6"
      },
      "bin": {
        "next": "dist/bin/next"
      },
      "engines": {
        "node": ">=20.9.0"
      },
      "optionalDependencies": {
        "@next/swc-darwin-arm64": "16.2.10",
        "@next/swc-darwin-x64": "16.2.10",
        "@next/swc-linux-arm64-gnu": "16.2.10",
        "@next/swc-linux-arm64-musl": "16.2.10",
        "@next/swc-linux-x64-gnu": "16.2.10",
        "@next/swc-linux-x64-musl": "16.2.10",
        "@next/swc-win32-arm64-msvc": "16.2.10",
        "@next/swc-win32-x64-msvc": "16.2.10",
        "sharp": "^0.34.5"
      },
      "peerDependencies": {
        "@opentelemetry/api": "^1.1.0",
        "@playwright/test": "^1.51.1",
        "babel-plugin-react-compiler": "*",
        "react": "^18.2.0 || 19.0.0-rc-de68d2f4-20241204 || ^19.0.0",
        "react-dom": "^18.2.0 || 19.0.0-rc-de68d2f4-20241204 || ^19.0.0",
        "sass": "^1.3.0"
      },
      "peerDependenciesMeta": {
        "@opentelemetry/api": {
          "optional": true
        },
        "@playwright/test": {
          "optional": true
        },
        "babel-plugin-react-compiler": {
          "optional": true
        },
        "sass": {
          "optional": true
        }
      }
    },
    "node_modules/node-exports-info": {
      "version": "1.6.2",
      "resolved": "https://registry.npmjs.org/node-exports-info/-/node-exports-info-1.6.2.tgz",
      "integrity": "sha512-kXs9Go0cah0qHVV2v389IXQLdLCeE1xfFtjOAF+iobu0OIoG1pje8At2vMHyaPMiPMnG/LWP50twML21eMcAag==",
      "dev": true,
      "dependencies": {
        "array.prototype.flatmap": "^1.3.3",
        "es-errors": "^1.3.0",
        "object.entries": "^1.1.9",
        "semver": "^6.3.1"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/node-releases": {
      "version": "2.0.51",
      "resolved": "https://registry.npmjs.org/node-releases/-/node-releases-2.0.51.tgz",
      "integrity": "sha512-wRNIrw4DmVLKQlbgOMdkMx27Wrpzes2hh5Jtbi2bjPd+4wJstWIqP5A+lscnqbm0xxmT5Bpg8Lec5ItEBwx6BQ==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=18"
      }
    },
    "node_modules/object-assign": {
      "version": "4.1.1",
      "resolved": "https://registry.npmjs.org/object-assign/-/object-assign-4.1.1.tgz",
      "integrity": "sha512-rJgTQnkUnH1sFw8yT6VSU3zD3sWmu6sZhIseY8VX+GRu3P6F7Fu+JNDoXfklElbLJSnc3FUQHVe4cU5hj+BcUg==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=0.10.0"
      }
    },
    "node_modules/object-inspect": {
      "version": "1.13.4",
      "resolved": "https://registry.npmjs.org/object-inspect/-/object-inspect-1.13.4.tgz",
      "integrity": "sha512-W67iLl4J2EXEGTbfeHCffrjDfitvLANg0UlX3wFUUSTx92KXRFegMHUVgSqE+wvhAbi4WqjGg9czysTV2Epbew==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/object-keys": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/object-keys/-/object-keys-1.1.1.tgz",
      "integrity": "sha512-NuAESUOUMrlIXOfHKzD6bpPu3tYt3xvjNdRIQ+FeT0lNb4K8WR70CaDxhuNguS2XG+GjkyMwOzsN5ZktImfhLA==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/object.assign": {
      "version": "4.1.7",
      "resolved": "https://registry.npmjs.org/object.assign/-/object.assign-4.1.7.tgz",
      "integrity": "sha512-nK28WOo+QIjBkDduTINE4JkF/UJJKyf2EJxvJKfblDpyg0Q+pkOHNTL0Qwy6NP6FhE/EnzV73BxxqcJaXY9anw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.8",
        "call-bound": "^1.0.3",
        "define-properties": "^1.2.1",
        "es-object-atoms": "^1.0.0",
        "has-symbols": "^1.1.0",
        "object-keys": "^1.1.1"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/object.entries": {
      "version": "1.1.9",
      "resolved": "https://registry.npmjs.org/object.entries/-/object.entries-1.1.9.tgz",
      "integrity": "sha512-8u/hfXFRBD1O0hPUjioLhoWFHRmt6tKA4/vZPyckBr18l1KE9uHrFaFaUi8MDRTpi4uak2goyPTSNJLXX2k2Hw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.8",
        "call-bound": "^1.0.4",
        "define-properties": "^1.2.1",
        "es-object-atoms": "^1.1.1"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/object.fromentries": {
      "version": "2.0.8",
      "resolved": "https://registry.npmjs.org/object.fromentries/-/object.fromentries-2.0.8.tgz",
      "integrity": "sha512-k6E21FzySsSK5a21KRADBd/NGneRegFO5pLHfdQLpRDETUNJueLXs3WCzyQ3tFRDYgbq3KHGXfTbi2bs8WQ6rQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.7",
        "define-properties": "^1.2.1",
        "es-abstract": "^1.23.2",
        "es-object-atoms": "^1.0.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/object.groupby": {
      "version": "1.0.3",
      "resolved": "https://registry.npmjs.org/object.groupby/-/object.groupby-1.0.3.tgz",
      "integrity": "sha512-+Lhy3TQTuzXI5hevh8sBGqbmurHbbIjAi0Z4S63nthVLmLxfbj4T54a4CfZrXIrt9iP4mVAPYMo/v99taj3wjQ==",
      "dev": true,
      "dependencies": {
        "call-bind": "^1.0.7",
        "define-properties": "^1.2.1",
        "es-abstract": "^1.23.2"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/object.values": {
      "version": "1.2.1",
      "resolved": "https://registry.npmjs.org/object.values/-/object.values-1.2.1.tgz",
      "integrity": "sha512-gXah6aZrcUxjWg2zR2MwouP2eHlCBzdV4pygudehaKXSGW4v2AsRQUK+lwwXhii6KFZcunEnmSUoYp5CXibxtA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.8",
        "call-bound": "^1.0.3",
        "define-properties": "^1.2.1",
        "es-object-atoms": "^1.0.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/optionator": {
      "version": "0.9.4",
      "resolved": "https://registry.npmjs.org/optionator/-/optionator-0.9.4.tgz",
      "integrity": "sha512-6IpQ7mKUxRcZNLIObR0hz7lxsapSSIYNZJwXPGeF0mTVqGKFIXj1DQcMoT22S3ROcLyY/rz0PWaWZ9ayWmad9g==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "deep-is": "^0.1.3",
        "fast-levenshtein": "^2.0.6",
        "levn": "^0.4.1",
        "prelude-ls": "^1.2.1",
        "type-check": "^0.4.0",
        "word-wrap": "^1.2.5"
      },
      "engines": {
        "node": ">= 0.8.0"
      }
    },
    "node_modules/own-keys": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/own-keys/-/own-keys-1.0.1.tgz",
      "integrity": "sha512-qFOyK5PjiWZd+QQIh+1jhdb9LpxTF0qs7Pm8o5QHYZ0M3vKqSqzsZaEB6oWlxZ+q2sJBMI/Ktgd2N5ZwQoRHfg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "get-intrinsic": "^1.2.6",
        "object-keys": "^1.1.1",
        "safe-push-apply": "^1.0.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/p-limit": {
      "version": "3.1.0",
      "resolved": "https://registry.npmjs.org/p-limit/-/p-limit-3.1.0.tgz",
      "integrity": "sha512-TYOanM3wGwNGsZN2cVTYPArw454xnXj5qmWF1bEoAc4+cU/ol7GVh7odevjp1FNHduHc3KZMcFduxU5Xc6uJRQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "yocto-queue": "^0.1.0"
      },
      "engines": {
        "node": ">=10"
      },
      "funding": {
        "url": "https://github.com/sponsors/sindresorhus"
      }
    },
    "node_modules/p-locate": {
      "version": "5.0.0",
      "resolved": "https://registry.npmjs.org/p-locate/-/p-locate-5.0.0.tgz",
      "integrity": "sha512-LaNjtRWUBY++zB5nE/NwcaoMylSPk+S+ZHNB1TzdbMJMny6dynpAGt7X/tl/QYq3TIeE6nxHppbo2LGymrG5Pw==",
      "dev": true,
      "dependencies": {
        "p-limit": "^3.0.2"
      },
      "engines": {
        "node": ">=10"
      },
      "funding": {
        "url": "https://github.com/sponsors/sindresorhus"
      }
    },
    "node_modules/parent-module": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/parent-module/-/parent-module-1.0.1.tgz",
      "integrity": "sha512-GQ2EWRpQV8/o+Aw8YqtfZZPfNRWZYkbidE9k5rpl/hC3vtHHBfGm2Ifi6qWV+coDGkrUKZAxE3Lot5kcsRlh+g==",
      "dev": true,
      "dependencies": {
        "callsites": "^3.0.0"
      },
      "engines": {
        "node": ">=6"
      }
    },
    "node_modules/path-exists": {
      "version": "4.0.0",
      "resolved": "https://registry.npmjs.org/path-exists/-/path-exists-4.0.0.tgz",
      "integrity": "sha512-ak9Qy5Q7jYb2Wwcey5Fpvg2KoAc/ZIhLSLOSBmRmygPsGwkVVt0fZa0qrtMz+m6tJTAHfZQ8FnmB4MG4LWy7/w==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=8"
      }
    },
    "node_modules/path-key": {
      "version": "3.1.1",
      "resolved": "https://registry.npmjs.org/path-key/-/path-key-3.1.1.tgz",
      "integrity": "sha512-ojmeN0qd+y0jszEtoY48r0Peq5dwMEkIlCOu6Q5f41lfkswXuKtYrhgoTpLnyIcHm24Uhqx+5Tqm2InSwLhE6Q==",
      "license": "MIT",
      "engines": {
        "node": ">=8"
      }
    },
    "node_modules/path-parse": {
      "version": "1.0.7",
      "resolved": "https://registry.npmjs.org/path-parse/-/path-parse-1.0.7.tgz",
      "integrity": "sha512-LDJzPVEEEPR+y48z93A0Ed0yXb8pAByGWo/k5YYdYgpY2/2EsOsksJrq7lOHxryrVOn1ejG6oAp8ahvOIQD8sw==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/picocolors": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/picocolors/-/picocolors-1.1.1.tgz",
      "integrity": "sha512-xceH2snhtb5M9liqDsmEw56le376mTZkEX/jEb/RxNFyegNul7eNslCXP9FDj/Lcu0X8KEyMceP2ntpaHrDEVA==",
      "license": "ISC"
    },
    "node_modules/picomatch": {
      "version": "2.3.2",
      "resolved": "https://registry.npmjs.org/picomatch/-/picomatch-2.3.2.tgz",
      "integrity": "sha512-V7+vQEJ06Z+c5tSye8S+nHUfI51xoXIXjHQ99cQtKUkQqqO1kO/KCJUfZXuB47h/YBlDhah2H3hdUGXn8ie0oA==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=8.6"
      },
      "funding": {
        "url": "https://github.com/sponsors/jonschlinkert"
      }
    },
    "node_modules/possible-typed-array-names": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/possible-typed-array-names/-/possible-typed-array-names-1.1.0.tgz",
      "integrity": "sha512-/+5VFTchJDoVj3bhoqi6UeymcD00DAwb1nJwamzPvHEszJ4FpF6SNNbUbOS8yI56qHzdV8eK0qEfOSiodkTdxg==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/postcss": {
      "version": "8.5.10",
      "resolved": "https://registry.npmjs.org/postcss/-/postcss-8.5.10.tgz",
      "integrity": "sha512-pMMHxBOZKFU6HgAZ4eyGnwXF/EvPGGqUr0MnZ5+99485wwW41kW91A4LOGxSHhgugZmSChL5AlElNdwlNgcnLQ==",
      "funding": [
        {
          "type": "opencollective",
          "url": "https://opencollective.com/postcss/"
        },
        {
          "type": "tidelift",
          "url": "https://tidelift.com/funding/github/npm/postcss"
        },
        {
          "type": "github",
          "url": "https://github.com/sponsors/ai"
        }
      ],
      "license": "MIT",
      "dependencies": {
        "nanoid": "^3.3.11",
        "picocolors": "^1.1.1",
        "source-map-js": "^1.2.1"
      },
      "engines": {
        "node": "^10 || ^12 || >=14"
      }
    },
    "node_modules/potpack": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/potpack/-/potpack-1.0.2.tgz",
      "integrity": "sha512-choctRBIV9EMT9WGAZHn3V7t0Z2pMQyl0EZE6pFc/6ml3ssw7Dlf/oAOvFwjm1HVsqfQN8GfeFyJ+d8tRzqueQ==",
      "license": "ISC"
    },
    "node_modules/prelude-ls": {
      "version": "1.2.1",
      "resolved": "https://registry.npmjs.org/prelude-ls/-/prelude-ls-1.2.1.tgz",
      "integrity": "sha512-vkcDPrRZo1QZLbn5RLGPpg/WmIQ65qoWWhcGKf/b5eplkkarX0m9z8ppCat4mlOqUsWpyNuYgO3VRyrYHSzX5g==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 0.8.0"
      }
    },
    "node_modules/promise-worker-transferable": {
      "version": "1.0.4",
      "resolved": "https://registry.npmjs.org/promise-worker-transferable/-/promise-worker-transferable-1.0.4.tgz",
      "integrity": "sha512-bN+0ehEnrXfxV2ZQvU2PetO0n4gqBD4ulq3MI1WOPLgr7/Mg9yRQkX5+0v1vagr74ZTsl7XtzlaYDo2EuCeYJw==",
      "license": "Apache-2.0",
      "dependencies": {
        "is-promise": "^2.1.0",
        "lie": "^3.0.2"
      }
    },
    "node_modules/prop-types": {
      "version": "15.8.1",
      "resolved": "https://registry.npmjs.org/prop-types/-/prop-types-15.8.1.tgz",
      "integrity": "sha512-oj87CgZICdulUohogVAR7AjlC0327U4el4L6eAvOqCeudMDVU0NThNaV+b9Df4dXgSP1gXMTnPdhfe/2qDH5cg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "loose-envify": "^1.4.0",
        "object-assign": "^4.1.1",
        "react-is": "^16.13.1"
      }
    },
    "node_modules/punycode": {
      "version": "2.3.1",
      "resolved": "https://registry.npmjs.org/punycode/-/punycode-2.3.1.tgz",
      "integrity": "sha512-vYt7UD1U9Wg6138shLtLOvdAu+8DsC/ilFtEVHcH+wydcSpNE20AfSOduf6MkRFahL5FY7X1oU7nKVZFtfq8Fg==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=6"
      }
    },
    "node_modules/queue-microtask": {
      "version": "1.2.3",
      "resolved": "https://registry.npmjs.org/queue-microtask/-/queue-microtask-1.2.3.tgz",
      "integrity": "sha512-NuaNSa6flKT5JaSYQzJok04JzTL1CA6aGhv5rfLW3PgqA+M2ChpZQnAC8h8i4ZFkBS8X5RqkDBHA7r4hej3K9A==",
      "dev": true,
      "funding": [
        {
          "type": "github",
          "url": "https://github.com/sponsors/feross"
        },
        {
          "type": "patreon",
          "url": "https://www.patreon.com/feross"
        },
        {
          "type": "consulting",
          "url": "https://feross.org/support"
        }
      ],
      "license": "MIT"
    },
    "node_modules/react": {
      "version": "19.2.7",
      "resolved": "https://registry.npmjs.org/react/-/react-19.2.7.tgz",
      "integrity": "sha512-HNe9WslTbXmFK8o8cmwgAeJFSBvt1bPdHCVKtaaV+WlAN36mpT4hcRpwbf3fY56ar2oIXzsBpOAiIRHAdY0OlQ==",
      "license": "MIT",
      "engines": {
        "node": ">=0.10.0"
      }
    },
    "node_modules/react-dom": {
      "version": "19.2.7",
      "resolved": "https://registry.npmjs.org/react-dom/-/react-dom-19.2.7.tgz",
      "integrity": "sha512-t0BRVXvbiE/o20Hfw669rLbMCDWtYZLvmJigy2f0MxsXF+71pxhR3xOkspmsO8h3ZlNzyibAmtCa3l4lYKk6gQ==",
      "license": "MIT",
      "dependencies": {
        "scheduler": "^0.27.0"
      },
      "peerDependencies": {
        "react": "^19.2.7"
      }
    },
    "node_modules/react-is": {
      "version": "16.13.1",
      "resolved": "https://registry.npmjs.org/react-is/-/react-is-16.13.1.tgz",
      "integrity": "sha512-24e6ynE2H+OKt4kqsOvNd8kBpV65zoxbA4BVsEOB3ARVWQki/DHzaUoC5KuON/BiccDaCCTZBuOcfZs70kR8bQ==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/react-use-measure": {
      "version": "2.1.7",
      "resolved": "https://registry.npmjs.org/react-use-measure/-/react-use-measure-2.1.7.tgz",
      "integrity": "sha512-KrvcAo13I/60HpwGO5jpW7E9DfusKyLPLvuHlUyP5zqnmAPhNc6qTRjUQrdTADl0lpPpDVU2/Gg51UlOGHXbdg==",
      "license": "MIT",
      "peerDependencies": {
        "react": ">=16.13",
        "react-dom": ">=16.13"
      },
      "peerDependenciesMeta": {
        "react-dom": {
          "optional": true
        }
      }
    },
    "node_modules/reflect.getprototypeof": {
      "version": "1.0.10",
      "resolved": "https://registry.npmjs.org/reflect.getprototypeof/-/reflect.getprototypeof-1.0.10.tgz",
      "integrity": "sha512-00o4I+DVrefhv+nX0ulyi3biSHCPDe+yLv5o/p6d/UVlirijB8E16FtfwSAi4g3tcqrQ4lRAqQSoFEZJehYEcw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.8",
        "define-properties": "^1.2.1",
        "es-abstract": "^1.23.9",
        "es-errors": "^1.3.0",
        "es-object-atoms": "^1.0.0",
        "get-intrinsic": "^1.2.7",
        "get-proto": "^1.0.1",
        "which-builtin-type": "^1.2.1"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/regexp.prototype.flags": {
      "version": "1.5.4",
      "resolved": "https://registry.npmjs.org/regexp.prototype.flags/-/regexp.prototype.flags-1.5.4.tgz",
      "integrity": "sha512-dYqgNSZbDwkaJ2ceRd9ojCGjBq+mOm9LmtXnAnEGyHhN/5R7iDW2TRw3h+o/jCFxus3P2LfWIIiwowAjANm7IA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.8",
        "define-properties": "^1.2.1",
        "es-errors": "^1.3.0",
        "get-proto": "^1.0.1",
        "gopd": "^1.2.0",
        "set-function-name": "^2.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/require-from-string": {
      "version": "2.0.2",
      "resolved": "https://registry.npmjs.org/require-from-string/-/require-from-string-2.0.2.tgz",
      "integrity": "sha512-Xf0nWe6RseziFMu+Ap9biiUbmplq6S9/p+7w7YXP/JBHhrUDDUhwa+vANyubuqfZWTveU//DYVGsDG7RKL/vEw==",
      "license": "MIT",
      "engines": {
        "node": ">=0.10.0"
      }
    },
    "node_modules/resolve": {
      "version": "2.0.0-next.7",
      "resolved": "https://registry.npmjs.org/resolve/-/resolve-2.0.0-next.7.tgz",
      "integrity": "sha512-tqt+NBWwyaMgw3zDsnygx4CByWjQEJHOPMdslYhppaQSJUtL/D4JO9CcBBlhPoI8lz9oJIDXkwXfhF4aWqP8xQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "es-errors": "^1.3.0",
        "is-core-module": "^2.16.2",
        "node-exports-info": "^1.6.0",
        "object-keys": "^1.1.1",
        "path-parse": "^1.0.7",
        "supports-preserve-symlinks-flag": "^1.0.0"
      },
      "bin": {
        "resolve": "bin/resolve"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/resolve-from": {
      "version": "4.0.0",
      "resolved": "https://registry.npmjs.org/resolve-from/-/resolve-from-4.0.0.tgz",
      "integrity": "sha512-pb/MYmXstAkysRFx8piNI1tGFNQIFA3vkE3Gq4EuA1dF6gHp/+vgZqsCGJapvy8N3Q+4o7FwvquPJcnZ7RYy4g==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=4"
      }
    },
    "node_modules/resolve-pkg-maps": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/resolve-pkg-maps/-/resolve-pkg-maps-1.0.0.tgz",
      "integrity": "sha512-seS2Tj26TBVOC2NIc2rOe2y2ZO7efxITtLZcGSOnHHNOQ7CkiUBfw0Iw2ck6xkIhPwLhKNLS8BO+hEpngQlqzw==",
      "dev": true,
      "license": "MIT",
      "funding": {
        "url": "https://github.com/privatenumber/resolve-pkg-maps?sponsor=1"
      }
    },
    "node_modules/reusify": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/reusify/-/reusify-1.1.0.tgz",
      "integrity": "sha512-g6QUff04oZpHs0eG5p83rFLhHeV00ug/Yf9nZM6fLeUrPguBTkTQOdpAWWspMh55TZfVQDPaN3NQJfbVRAxdIw==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "iojs": ">=1.0.0",
        "node": ">=0.10.0"
      }
    },
    "node_modules/run-parallel": {
      "version": "1.2.0",
      "resolved": "https://registry.npmjs.org/run-parallel/-/run-parallel-1.2.0.tgz",
      "integrity": "sha512-5l4VyZR86LZ/lDxZTR6jqL8AFE2S0IFLMP26AbjsLVADxHdhB/c0GUsH+y39UfCi3dzz8OlQuPmnaJOMoDHQBA==",
      "dev": true,
      "funding": [
        {
          "type": "github",
          "url": "https://github.com/sponsors/feross"
        },
        {
          "type": "patreon",
          "url": "https://www.patreon.com/feross"
        },
        {
          "type": "consulting",
          "url": "https://feross.org/support"
        }
      ],
      "license": "MIT",
      "dependencies": {
        "queue-microtask": "^1.2.2"
      }
    },
    "node_modules/safe-array-concat": {
      "version": "1.1.4",
      "resolved": "https://registry.npmjs.org/safe-array-concat/-/safe-array-concat-1.1.4.tgz",
      "integrity": "sha512-wtZlHyOje6OZTGqAoaDKxFkgRtkF9CnHAVnCHKfuj200wAgL+bSJhdsCD2l0Qx/2ekEXjPWcyKkfGb5CPboslg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.9",
        "call-bound": "^1.0.4",
        "get-intrinsic": "^1.3.0",
        "has-symbols": "^1.1.0",
        "isarray": "^2.0.5"
      },
      "engines": {
        "node": ">=0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/safe-push-apply": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/safe-push-apply/-/safe-push-apply-1.0.0.tgz",
      "integrity": "sha512-iKE9w/Z7xCzUMIZqdBsp6pEQvwuEebH4vdpjcDWnyzaI6yl6O9FHvVpmGelvEHNsoY6wGblkxR6Zty/h00WiSA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "es-errors": "^1.3.0",
        "isarray": "^2.0.5"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/safe-regex-test": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/safe-regex-test/-/safe-regex-test-1.1.0.tgz",
      "integrity": "sha512-x/+Cz4YrimQxQccJf5mKEbIa1NzeCRNI5Ecl/ekmlYaampdNLPalVyIcCZNNH3MvmqBugV5TMYZXv0ljslUlaw==",
      "dev": true,
      "dependencies": {
        "call-bound": "^1.0.2",
        "es-errors": "^1.3.0",
        "is-regex": "^1.2.1"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/scheduler": {
      "version": "0.27.0",
      "resolved": "https://registry.npmjs.org/scheduler/-/scheduler-0.27.0.tgz",
      "integrity": "sha512-eNv+WrVbKu1f3vbYJT/xtiF5syA5HPIMtf9IgY/nKg0sWqzAUEvqY/xm7OcZc/qafLx/iO9FgOmeSAp4v5ti/Q==",
      "license": "MIT"
    },
    "node_modules/semver": {
      "version": "6.3.1",
      "resolved": "https://registry.npmjs.org/semver/-/semver-6.3.1.tgz",
      "integrity": "sha512-BR7VvDCVHO+q2xBEWskxS6DJE1qRnb7DxzUrogb71CWoSficBxYsiAGd+Kl0mmq/MprG9yArRkyrQxTO6XjMzA==",
      "dev": true,
      "license": "ISC",
      "bin": {
        "semver": "bin/semver.js"
      }
    },
    "node_modules/set-function-length": {
      "version": "1.2.2",
      "resolved": "https://registry.npmjs.org/set-function-length/-/set-function-length-1.2.2.tgz",
      "integrity": "sha512-pgRc4hJ4/sNjWCSS9AmnS40x3bNMDTknHgL5UaMBTMyJnU90EgWh1Rz+MC9eFu4BuN/UwZjKQuY/1v3rM7HMfg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "define-data-property": "^1.1.4",
        "es-errors": "^1.3.0",
        "function-bind": "^1.1.2",
        "get-intrinsic": "^1.2.4",
        "gopd": "^1.0.1",
        "has-property-descriptors": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/set-function-name": {
      "version": "2.0.2",
      "resolved": "https://registry.npmjs.org/set-function-name/-/set-function-name-2.0.2.tgz",
      "integrity": "sha512-7PGFlmtwsEADb0WYyvCMa1t+yke6daIG4Wirafur5kcf+MhUnPms1UeR0CKQdTZD81yESwMHbtn+TR+dMviakQ==",
      "dev": true,
      "dependencies": {
        "define-data-property": "^1.1.4",
        "es-errors": "^1.3.0",
        "functions-have-names": "^1.2.3",
        "has-property-descriptors": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/set-proto": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/set-proto/-/set-proto-1.0.0.tgz",
      "integrity": "sha512-RJRdvCo6IAnPdsvP/7m6bsQqNnn1FCBX5ZNtFL98MmFF/4xAIJTIg1YbHW5DC2W5SKZanrC6i4HsJqlajw/dZw==",
      "dev": true,
      "dependencies": {
        "dunder-proto": "^1.0.1",
        "es-errors": "^1.3.0",
        "es-object-atoms": "^1.0.0"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/sharp": {
      "version": "0.34.5",
      "resolved": "https://registry.npmjs.org/sharp/-/sharp-0.34.5.tgz",
      "integrity": "sha512-Ou9I5Ft9WNcCbXrU9cMgPBcCK8LiwLqcbywW3t4oDV37n1pzpuNLsYiAV8eODnjbtQlSDwZ2cUEeQz4E54Hltg==",
      "hasInstallScript": true,
      "optional": true,
      "dependencies": {
        "@img/colour": "^1.0.0",
        "detect-libc": "^2.1.2",
        "semver": "^7.7.3"
      },
      "engines": {
        "node": "^18.17.0 || ^20.3.0 || >=21.0.0"
      },
      "funding": {
        "url": "https://opencollective.com/libvips"
      },
      "optionalDependencies": {
        "@img/sharp-darwin-arm64": "0.34.5",
        "@img/sharp-darwin-x64": "0.34.5",
        "@img/sharp-libvips-darwin-arm64": "1.2.4",
        "@img/sharp-libvips-darwin-x64": "1.2.4",
        "@img/sharp-libvips-linux-arm": "1.2.4",
        "@img/sharp-libvips-linux-arm64": "1.2.4",
        "@img/sharp-libvips-linux-ppc64": "1.2.4",
        "@img/sharp-libvips-linux-riscv64": "1.2.4",
        "@img/sharp-libvips-linux-s390x": "1.2.4",
        "@img/sharp-libvips-linux-x64": "1.2.4",
        "@img/sharp-libvips-linuxmusl-arm64": "1.2.4",
        "@img/sharp-libvips-linuxmusl-x64": "1.2.4",
        "@img/sharp-linux-arm": "0.34.5",
        "@img/sharp-linux-arm64": "0.34.5",
        "@img/sharp-linux-ppc64": "0.34.5",
        "@img/sharp-linux-riscv64": "0.34.5",
        "@img/sharp-linux-s390x": "0.34.5",
        "@img/sharp-linux-x64": "0.34.5",
        "@img/sharp-linuxmusl-arm64": "0.34.5",
        "@img/sharp-linuxmusl-x64": "0.34.5",
        "@img/sharp-wasm32": "0.34.5",
        "@img/sharp-win32-arm64": "0.34.5",
        "@img/sharp-win32-ia32": "0.34.5",
        "@img/sharp-win32-x64": "0.34.5"
      }
    },
    "node_modules/sharp/node_modules/semver": {
      "version": "7.8.5",
      "resolved": "https://registry.npmjs.org/semver/-/semver-7.8.5.tgz",
      "integrity": "sha512-Y7/KDsb8LjooZpwaqGyulO6DQlksgCncchHGk+sZIY4SBvUocMBEFH5Ur1fI4dV+Jvl0w6cjvucaIi40puRioA==",
      "license": "ISC",
      "optional": true,
      "bin": {
        "semver": "bin/semver.js"
      },
      "engines": {
        "node": ">=10"
      }
    },
    "node_modules/shebang-command": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/shebang-command/-/shebang-command-2.0.0.tgz",
      "integrity": "sha512-kHxr2zZpYtdmrN1qDjrrX/Z1rR1kG8Dx+gkpK1G4eXmvXswmcE1hTWBWYUzlraYw1/yZp6YuDY77YtvbN0dmDA==",
      "license": "MIT",
      "dependencies": {
        "shebang-regex": "^3.0.0"
      },
      "engines": {
        "node": ">=8"
      }
    },
    "node_modules/shebang-regex": {
      "version": "3.0.0",
      "resolved": "https://registry.npmjs.org/shebang-regex/-/shebang-regex-3.0.0.tgz",
      "integrity": "sha512-7++dFhtcx3353uBaq8DDR4NuxBetBzC7ZQOhmTQInHEd6bSrXdiEyzCvG07Z44UYdLShWUyXt5M/yhz8ekcb1A==",
      "license": "MIT",
      "engines": {
        "node": ">=8"
      }
    },
    "node_modules/side-channel": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/side-channel/-/side-channel-1.1.1.tgz",
      "integrity": "sha512-6x6dK6zJdpTzF4sQeNYxwtvBzf6Eg4GtlesS94HOvTudUeyK2WXAaIfmDgsyslYrRBeFIlsi54AYsFGUuhmvrQ==",
      "dev": true,
      "dependencies": {
        "es-errors": "^1.3.0",
        "object-inspect": "^1.13.4",
        "side-channel-list": "^1.0.1",
        "side-channel-map": "^1.0.1",
        "side-channel-weakmap": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/side-channel-list": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/side-channel-list/-/side-channel-list-1.0.1.tgz",
      "integrity": "sha512-mjn/0bi/oUURjc5Xl7IaWi/OJJJumuoJFQJfDDyO46+hBWsfaVM65TBHq2eoZBhzl9EchxOijpkbRC8SVBQU0w==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "es-errors": "^1.3.0",
        "object-inspect": "^1.13.4"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/side-channel-map": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/side-channel-map/-/side-channel-map-1.0.1.tgz",
      "integrity": "sha512-VCjCNfgMsby3tTdo02nbjtM/ewra6jPHmpThenkTYh8pG9ucZ/1P8So4u4FGBek/BjpOVsDCMoLA/iuBKIFXRA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.2",
        "es-errors": "^1.3.0",
        "get-intrinsic": "^1.2.5",
        "object-inspect": "^1.13.3"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/side-channel-weakmap": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/side-channel-weakmap/-/side-channel-weakmap-1.0.2.tgz",
      "integrity": "sha512-WPS/HvHQTYnHisLo9McqBHOJk2FkHO/tlpvldyrnem4aeQp4hai3gythswg6p01oSoTl58rcpiFAjF2br2Ak2A==",
      "dev": true,
      "dependencies": {
        "call-bound": "^1.0.2",
        "es-errors": "^1.3.0",
        "get-intrinsic": "^1.2.5",
        "object-inspect": "^1.13.3",
        "side-channel-map": "^1.0.1"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/source-map-js": {
      "version": "1.2.1",
      "resolved": "https://registry.npmjs.org/source-map-js/-/source-map-js-1.2.1.tgz",
      "integrity": "sha512-UXWMKhLOwVKb728IUtQPXxfYU+usdybtUrK/8uGE8CQMvrhOpwvzDBwj0QhSL7MQc7vIsISBG8VQ8+IDQxpfQA==",
      "license": "BSD-3-Clause",
      "engines": {
        "node": ">=0.10.0"
      }
    },
    "node_modules/stable-hash": {
      "version": "0.0.5",
      "resolved": "https://registry.npmjs.org/stable-hash/-/stable-hash-0.0.5.tgz",
      "integrity": "sha512-+L3ccpzibovGXFK+Ap/f8LOS0ahMrHTf3xu7mMLSpEGU0EO9ucaysSylKo9eRDFNhWve/y275iPmIZ4z39a9iA==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/stats-gl": {
      "version": "2.4.2",
      "resolved": "https://registry.npmjs.org/stats-gl/-/stats-gl-2.4.2.tgz",
      "integrity": "sha512-g5O9B0hm9CvnM36+v7SFl39T7hmAlv541tU81ME8YeSb3i1CIP5/QdDeSB3A0la0bKNHpxpwxOVRo2wFTYEosQ==",
      "dependencies": {
        "@types/three": "*",
        "three": "^0.170.0"
      },
      "peerDependencies": {
        "@types/three": "*",
        "three": "*"
      }
    },
    "node_modules/stats-gl/node_modules/three": {
      "version": "0.170.0",
      "resolved": "https://registry.npmjs.org/three/-/three-0.170.0.tgz",
      "integrity": "sha512-FQK+LEpYc0fBD+J8g6oSEyyNzjp+Q7Ks1C568WWaoMRLW+TkNNWmenWeGgJjV105Gd+p/2ql1ZcjYvNiPZBhuQ==",
      "license": "MIT"
    },
    "node_modules/stats.js": {
      "version": "0.17.0",
      "resolved": "https://registry.npmjs.org/stats.js/-/stats.js-0.17.0.tgz",
      "integrity": "sha512-hNKz8phvYLPEcRkeG1rsGmV5ChMjKDAWU7/OJJdDErPBNChQXxCo3WZurGpnWc6gZhAzEPFad1aVgyOANH1sMw=="
    },
    "node_modules/stop-iteration-iterator": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/stop-iteration-iterator/-/stop-iteration-iterator-1.1.0.tgz",
      "integrity": "sha512-eLoXW/DHyl62zxY4SCaIgnRhuMr6ri4juEYARS8E6sCEqzKpOiE521Ucofdx+KnDZl5xmvGYaaKCk5FEOxJCoQ==",
      "dev": true,
      "dependencies": {
        "es-errors": "^1.3.0",
        "internal-slot": "^1.1.0"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/string.prototype.includes": {
      "version": "2.0.1",
      "resolved": "https://registry.npmjs.org/string.prototype.includes/-/string.prototype.includes-2.0.1.tgz",
      "integrity": "sha512-o7+c9bW6zpAdJHTtujeePODAhkuicdAryFsfVKwA+wGw89wJ4GTY484WTucM9hLtDEOpOvI+aHnzqnC5lHp4Rg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.7",
        "define-properties": "^1.2.1",
        "es-abstract": "^1.23.3"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/string.prototype.matchall": {
      "version": "4.0.12",
      "resolved": "https://registry.npmjs.org/string.prototype.matchall/-/string.prototype.matchall-4.0.12.tgz",
      "integrity": "sha512-6CC9uyBL+/48dYizRf7H7VAYCMCNTBeM78x/VTUe9bFEaxBepPJDa1Ow99LqI/1yF7kuy7Q3cQsYMrcjGUcskA==",
      "dev": true,
      "dependencies": {
        "call-bind": "^1.0.8",
        "call-bound": "^1.0.3",
        "define-properties": "^1.2.1",
        "es-abstract": "^1.23.6",
        "es-errors": "^1.3.0",
        "es-object-atoms": "^1.0.0",
        "get-intrinsic": "^1.2.6",
        "gopd": "^1.2.0",
        "has-symbols": "^1.1.0",
        "internal-slot": "^1.1.0",
        "regexp.prototype.flags": "^1.5.3",
        "set-function-name": "^2.0.2",
        "side-channel": "^1.1.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/string.prototype.repeat": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/string.prototype.repeat/-/string.prototype.repeat-1.0.0.tgz",
      "integrity": "sha512-0u/TldDbKD8bFCQ/4f5+mNRrXwZ8hg2w7ZR8wa16e8z9XpePWl3eGEcUD0OXpEH/VJH/2G3gjUtR3ZOiBe2S/w==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "define-properties": "^1.1.3",
        "es-abstract": "^1.17.5"
      }
    },
    "node_modules/string.prototype.trim": {
      "version": "1.2.11",
      "resolved": "https://registry.npmjs.org/string.prototype.trim/-/string.prototype.trim-1.2.11.tgz",
      "integrity": "sha512-PwvK7BU+CMTJGYQCTZb5RWXIML92lftJLhQz1tBzgKiqGxJaMlBAa48POXaNAC2s4y8jr3EFqrkF9+44neS46w==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.9",
        "call-bound": "^1.0.4",
        "define-data-property": "^1.1.4",
        "define-properties": "^1.2.1",
        "es-abstract": "^1.24.2",
        "es-object-atoms": "^1.1.2",
        "has-property-descriptors": "^1.0.2",
        "safe-regex-test": "^1.1.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/string.prototype.trimend": {
      "version": "1.0.10",
      "resolved": "https://registry.npmjs.org/string.prototype.trimend/-/string.prototype.trimend-1.0.10.tgz",
      "integrity": "sha512-2+3aDAOmPTmuFwjDnmJG2ctEkQKVki7vOSqaxkv42Mowj1V6PnvuwFCRrR5lChUux1TBskPjfkeTOhqczDMxTw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.9",
        "call-bound": "^1.0.4",
        "define-properties": "^1.2.1",
        "es-object-atoms": "^1.1.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/string.prototype.trimstart": {
      "version": "1.0.8",
      "resolved": "https://registry.npmjs.org/string.prototype.trimstart/-/string.prototype.trimstart-1.0.8.tgz",
      "integrity": "sha512-UXSH262CSZY1tfu3G3Secr6uGLCFVPMhIqHjlgCUtCCcgihYc/xKs9djMTMUOb2j1mVSeU8EU6NWc/iQKU6Gfg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.7",
        "define-properties": "^1.2.1",
        "es-object-atoms": "^1.0.0"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/strip-bom": {
      "version": "3.0.0",
      "resolved": "https://registry.npmjs.org/strip-bom/-/strip-bom-3.0.0.tgz",
      "integrity": "sha512-vavAMRXOgBVNF6nyEEmL3DBK19iRpDcoIwW+swQ+CbGiu7lju6t+JklA1MHweoWtadgt4ISVUsXLyDq34ddcwA==",
      "dev": true,
      "engines": {
        "node": ">=4"
      }
    },
    "node_modules/strip-json-comments": {
      "version": "3.1.1",
      "resolved": "https://registry.npmjs.org/strip-json-comments/-/strip-json-comments-3.1.1.tgz",
      "integrity": "sha512-6fPc+R4ihwqP6N/aIv2f1gMH8lOVtWQHoqC4yK6oSDVVocumAsfCqjkXnqiYMhmMwS/mEHLp7Vehlt3ql6lEig==",
      "dev": true,
      "engines": {
        "node": ">=8"
      },
      "funding": {
        "url": "https://github.com/sponsors/sindresorhus"
      }
    },
    "node_modules/styled-jsx": {
      "version": "5.1.6",
      "resolved": "https://registry.npmjs.org/styled-jsx/-/styled-jsx-5.1.6.tgz",
      "integrity": "sha512-qSVyDTeMotdvQYoHWLNGwRFJHC+i+ZvdBRYosOFgC+Wg1vx4frN2/RG/NA7SYqqvKNLf39P2LSRA2pu6n0XYZA==",
      "license": "MIT",
      "dependencies": {
        "client-only": "0.0.1"
      },
      "engines": {
        "node": ">= 12.0.0"
      },
      "peerDependencies": {
        "react": ">= 16.8.0 || 17.x.x || ^18.0.0-0 || ^19.0.0-0"
      },
      "peerDependenciesMeta": {
        "@babel/core": {
          "optional": true
        },
        "babel-plugin-macros": {
          "optional": true
        }
      }
    },
    "node_modules/supports-color": {
      "version": "7.2.0",
      "resolved": "https://registry.npmjs.org/supports-color/-/supports-color-7.2.0.tgz",
      "integrity": "sha512-qpCAvRl9stuOHveKsn7HncJRvv501qIacKzQlO/+Lwxc9+0q2wLyv4Dfvt80/DPn2pqOBsJdDiogXGR9+OvwRw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "has-flag": "^4.0.0"
      },
      "engines": {
        "node": ">=8"
      }
    },
    "node_modules/supports-preserve-symlinks-flag": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/supports-preserve-symlinks-flag/-/supports-preserve-symlinks-flag-1.0.0.tgz",
      "integrity": "sha512-ot0WnXS9fgdkgIcePe6RHNk1WA8+muPa6cSjeR3V8K27q9BB1rTE3R1p7Hv0z1ZyAc8s6Vvv8DIyWf681MAt0w==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/suspend-react": {
      "version": "0.1.3",
      "resolved": "https://registry.npmjs.org/suspend-react/-/suspend-react-0.1.3.tgz",
      "integrity": "sha512-aqldKgX9aZqpoDp3e8/BZ8Dm7x1pJl+qI3ZKxDN0i/IQTWUwBx/ManmlVJ3wowqbno6c2bmiIfs+Um6LbsjJyQ==",
      "license": "MIT",
      "peerDependencies": {
        "react": ">=17.0"
      }
    },
    "node_modules/three": {
      "version": "0.185.1",
      "resolved": "https://registry.npmjs.org/three/-/three-0.185.1.tgz",
      "integrity": "sha512-5aojFCXKwnjBRZvUnt3WFfEcvUJgkN5LlijRFN95hMy8WVkG4I0QNcJE+OuWvuJ0bOdStrbfXn0pkd6/QyiAlg==",
      "license": "MIT"
    },
    "node_modules/three-mesh-bvh": {
      "version": "0.8.3",
      "resolved": "https://registry.npmjs.org/three-mesh-bvh/-/three-mesh-bvh-0.8.3.tgz",
      "integrity": "sha512-4G5lBaF+g2auKX3P0yqx+MJC6oVt6sB5k+CchS6Ob0qvH0YIhuUk1eYr7ktsIpY+albCqE80/FVQGV190PmiAg==",
      "license": "MIT",
      "peerDependencies": {
        "three": ">= 0.159.0"
      }
    },
    "node_modules/three-stdlib": {
      "version": "2.36.1",
      "resolved": "https://registry.npmjs.org/three-stdlib/-/three-stdlib-2.36.1.tgz",
      "integrity": "sha512-XyGQrFmNQ5O/IoKm556ftwKsBg11TIb301MB5dWNicziQBEs2g3gtOYIf7pFiLa0zI2gUwhtCjv9fmjnxKZ1Cg==",
      "license": "MIT",
      "dependencies": {
        "@types/draco3d": "^1.4.0",
        "@types/offscreencanvas": "^2019.6.4",
        "@types/webxr": "^0.5.2",
        "draco3d": "^1.4.1",
        "fflate": "^0.6.9",
        "potpack": "^1.0.1"
      },
      "peerDependencies": {
        "three": ">=0.128.0"
      }
    },
    "node_modules/three-stdlib/node_modules/fflate": {
      "version": "0.6.11",
      "resolved": "https://registry.npmjs.org/fflate/-/fflate-0.6.11.tgz",
      "integrity": "sha512-3JyEFWGjFn7zHmoa9+zG1BmW7X2okcmAB+0Cnu9UFbVs/jCBnl2A8o065ZlXiw145K3eBM3uLuzrYXC0RK7eDg==",
      "license": "MIT"
    },
    "node_modules/tinyglobby": {
      "version": "0.2.17",
      "resolved": "https://registry.npmjs.org/tinyglobby/-/tinyglobby-0.2.17.tgz",
      "integrity": "sha512-wXR/dYpcqKmfWpEdZjiKJOwCNFndD0DMnrW/cYjVGttEkBfVgcLFHoNrlj47mjOVic9yyNu65alsgF4NQyTa2g==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "fdir": "^6.5.0",
        "picomatch": "^4.0.4"
      },
      "engines": {
        "node": ">=12.0.0"
      },
      "funding": {
        "url": "https://github.com/sponsors/SuperchupuDev"
      }
    },
    "node_modules/tinyglobby/node_modules/fdir": {
      "version": "6.5.0",
      "resolved": "https://registry.npmjs.org/fdir/-/fdir-6.5.0.tgz",
      "integrity": "sha512-tIbYtZbucOs0BRGqPJkshJUYdL+SDH7dVM8gjy+ERp3WAUjLEFJE+02kanyHtwjWOnwrKYBiwAmM0p4kLJAnXg==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=12.0.0"
      },
      "peerDependencies": {
        "picomatch": "^3 || ^4"
      },
      "peerDependenciesMeta": {
        "picomatch": {
          "optional": true
        }
      }
    },
    "node_modules/tinyglobby/node_modules/picomatch": {
      "version": "4.0.5",
      "resolved": "https://registry.npmjs.org/picomatch/-/picomatch-4.0.5.tgz",
      "integrity": "sha512-RvwwcruNjI1ncT5xRakeyS9Lf8lcItv34KD+aif+VH9kduAyfYBipGh12274xtenIPZ119/R9BdTBa8gAwSh0A==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=12"
      },
      "funding": {
        "url": "https://github.com/sponsors/jonschlinkert"
      }
    },
    "node_modules/to-regex-range": {
      "version": "5.0.1",
      "resolved": "https://registry.npmjs.org/to-regex-range/-/to-regex-range-5.0.1.tgz",
      "integrity": "sha512-65P7iz6X5yEr1cwcgvQxbbIw7Uk3gOy5dIdtZ4rDveLqhrdJP+Li/Hx6tyK0NEb+2GCyneCMJiGqrADCSNk8sQ==",
      "dev": true,
      "dependencies": {
        "is-number": "^7.0.0"
      },
      "engines": {
        "node": ">=8.0"
      }
    },
    "node_modules/troika-three-text": {
      "version": "0.52.4",
      "resolved": "https://registry.npmjs.org/troika-three-text/-/troika-three-text-0.52.4.tgz",
      "integrity": "sha512-V50EwcYGruV5rUZ9F4aNsrytGdKcXKALjEtQXIOBfhVoZU9VAqZNIoGQ3TMiooVqFAbR1w15T+f+8gkzoFzawg==",
      "license": "MIT",
      "dependencies": {
        "bidi-js": "^1.0.2",
        "troika-three-utils": "^0.52.4",
        "troika-worker-utils": "^0.52.0",
        "webgl-sdf-generator": "1.1.1"
      },
      "peerDependencies": {
        "three": ">=0.125.0"
      }
    },
    "node_modules/troika-three-utils": {
      "version": "0.52.4",
      "resolved": "https://registry.npmjs.org/troika-three-utils/-/troika-three-utils-0.52.4.tgz",
      "integrity": "sha512-NORAStSVa/BDiG52Mfudk4j1FG4jC4ILutB3foPnfGbOeIs9+G5vZLa0pnmnaftZUGm4UwSoqEpWdqvC7zms3A==",
      "license": "MIT",
      "peerDependencies": {
        "three": ">=0.125.0"
      }
    },
    "node_modules/troika-worker-utils": {
      "version": "0.52.0",
      "resolved": "https://registry.npmjs.org/troika-worker-utils/-/troika-worker-utils-0.52.0.tgz",
      "integrity": "sha512-W1CpvTHykaPH5brv5VHLfQo9D1OYuo0cSBEUQFFT/nBUzM8iD6Lq2/tgG/f1OelbAS1WtaTPQzE5uM49egnngw==",
      "license": "MIT"
    },
    "node_modules/ts-api-utils": {
      "version": "2.5.0",
      "resolved": "https://registry.npmjs.org/ts-api-utils/-/ts-api-utils-2.5.0.tgz",
      "integrity": "sha512-OJ/ibxhPlqrMM0UiNHJ/0CKQkoKF243/AEmplt3qpRgkW8VG7IfOS41h7V8TjITqdByHzrjcS/2si+y4lIh8NA==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=18.12"
      },
      "peerDependencies": {
        "typescript": ">=4.8.4"
      }
    },
    "node_modules/tsconfig-paths": {
      "version": "3.15.0",
      "resolved": "https://registry.npmjs.org/tsconfig-paths/-/tsconfig-paths-3.15.0.tgz",
      "integrity": "sha512-2Ac2RgzDe/cn48GvOe3M+o82pEFewD3UPbyoUHHdKasHwJKjds4fLXWf/Ux5kATBKN20oaFGu+jbElp1pos0mg==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@types/json5": "^0.0.29",
        "json5": "^1.0.2",
        "minimist": "^1.2.6",
        "strip-bom": "^3.0.0"
      }
    },
    "node_modules/tsconfig-paths/node_modules/json5": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/json5/-/json5-1.0.2.tgz",
      "integrity": "sha512-g1MWMLBiz8FKi1e4w0UyVL3w+iJceWAFBAaBnnGKOpNa5f8TLktkbre1+s6oICydWAm+HRUGTmI+//xv2hvXYA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "minimist": "^1.2.0"
      },
      "bin": {
        "json5": "lib/cli.js"
      }
    },
    "node_modules/tslib": {
      "version": "2.8.1",
      "resolved": "https://registry.npmjs.org/tslib/-/tslib-2.8.1.tgz",
      "integrity": "sha512-oJFu94HQb+KVduSUQL7wnpmqnfmLsOA/nAh6b6EH0wCEoK0/mPeXU6c3wKDV83MkOuHPRHtSXKKU99IBazS/2w==",
      "license": "0BSD"
    },
    "node_modules/tunnel-rat": {
      "version": "0.1.2",
      "resolved": "https://registry.npmjs.org/tunnel-rat/-/tunnel-rat-0.1.2.tgz",
      "integrity": "sha512-lR5VHmkPhzdhrM092lI2nACsLO4QubF0/yoOhzX7c+wIpbN1GjHNzCc91QlpxBi+cnx8vVJ+Ur6vL5cEoQPFpQ==",
      "license": "MIT",
      "dependencies": {
        "zustand": "^4.3.2"
      }
    },
    "node_modules/tunnel-rat/node_modules/zustand": {
      "version": "4.5.7",
      "resolved": "https://registry.npmjs.org/zustand/-/zustand-4.5.7.tgz",
      "integrity": "sha512-CHOUy7mu3lbD6o6LJLfllpjkzhHXSBlX8B9+qPddUsIfeF5S/UZ5q0kmCsnRqT1UHFQZchNFDDzMbQsuesHWlw==",
      "license": "MIT",
      "dependencies": {
        "use-sync-external-store": "^1.2.2"
      },
      "engines": {
        "node": ">=12.7.0"
      },
      "peerDependencies": {
        "@types/react": ">=16.8",
        "immer": ">=9.0.6",
        "react": ">=16.8"
      },
      "peerDependenciesMeta": {
        "@types/react": {
          "optional": true
        },
        "immer": {
          "optional": true
        },
        "react": {
          "optional": true
        }
      }
    },
    "node_modules/type-check": {
      "version": "0.4.0",
      "resolved": "https://registry.npmjs.org/type-check/-/type-check-0.4.0.tgz",
      "integrity": "sha512-XleUoc9uwGXqjWwXaUTZAmzMcFZ5858QA2vvx1Ur5xIcixXIP+8LnFDgRplU30us6teqdlskFfu+ae4K79Ooew==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "prelude-ls": "^1.2.1"
      },
      "engines": {
        "node": ">= 0.8.0"
      }
    },
    "node_modules/typed-array-buffer": {
      "version": "1.0.3",
      "resolved": "https://registry.npmjs.org/typed-array-buffer/-/typed-array-buffer-1.0.3.tgz",
      "integrity": "sha512-nAYYwfY3qnzX30IkA6AQZjVbtK6duGontcQm1WSG1MD94YLqK0515GNApXkoxKOWMusVssAHWLh9SeaoefYFGw==",
      "dev": true,
      "dependencies": {
        "call-bound": "^1.0.3",
        "es-errors": "^1.3.0",
        "is-typed-array": "^1.1.14"
      },
      "engines": {
        "node": ">= 0.4"
      }
    },
    "node_modules/typed-array-byte-length": {
      "version": "1.0.3",
      "resolved": "https://registry.npmjs.org/typed-array-byte-length/-/typed-array-byte-length-1.0.3.tgz",
      "integrity": "sha512-BaXgOuIxz8n8pIq3e7Atg/7s+DpiYrxn4vdot3w9KbnBhcRQq6o3xemQdIfynqSeXeDrF32x+WvfzmOjPiY9lg==",
      "dev": true,
      "dependencies": {
        "call-bind": "^1.0.8",
        "for-each": "^0.3.3",
        "gopd": "^1.2.0",
        "has-proto": "^1.2.0",
        "is-typed-array": "^1.1.14"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/typed-array-byte-offset": {
      "version": "1.0.4",
      "resolved": "https://registry.npmjs.org/typed-array-byte-offset/-/typed-array-byte-offset-1.0.4.tgz",
      "integrity": "sha512-bTlAFB/FBYMcuX81gbL4OcpH5PmlFHqlCCpAl8AlEzMz5k53oNDvN8p1PNOWLEmI2x4orp3raOFB51tv9X+MFQ==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "available-typed-arrays": "^1.0.7",
        "call-bind": "^1.0.8",
        "for-each": "^0.3.3",
        "gopd": "^1.2.0",
        "has-proto": "^1.2.0",
        "is-typed-array": "^1.1.15",
        "reflect.getprototypeof": "^1.0.9"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/typed-array-length": {
      "version": "1.0.8",
      "resolved": "https://registry.npmjs.org/typed-array-length/-/typed-array-length-1.0.8.tgz",
      "integrity": "sha512-phPGCwqr2+Qo0fwniCE8e4pKnGu/yFb5nD5Y8bf0EEeiI5GklnACYA9GFy/DrAeRrKHXvHn+1SUsOWgJp6RO+g==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bind": "^1.0.9",
        "for-each": "^0.3.5",
        "gopd": "^1.2.0",
        "is-typed-array": "^1.1.15",
        "possible-typed-array-names": "^1.1.0",
        "reflect.getprototypeof": "^1.0.10"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/typescript": {
      "version": "6.0.3",
      "resolved": "https://registry.npmjs.org/typescript/-/typescript-6.0.3.tgz",
      "integrity": "sha512-y2TvuxSZPDyQakkFRPZHKFm+KKVqIisdg9/CZwm9ftvKXLP8NRWj38/ODjNbr43SsoXqNuAisEf1GdCxqWcdBw==",
      "dev": true,
      "license": "Apache-2.0",
      "bin": {
        "tsc": "bin/tsc",
        "tsserver": "bin/tsserver"
      },
      "engines": {
        "node": ">=14.17"
      }
    },
    "node_modules/typescript-eslint": {
      "version": "8.65.0",
      "resolved": "https://registry.npmjs.org/typescript-eslint/-/typescript-eslint-8.65.0.tgz",
      "integrity": "sha512-/ggrHAwyjENDusvyxbuqxAC2dTnZg/Z8F+fgQtYIz+L6n/9HfSlEZcFGV/NsMNa6CkGk0xUjUAFwC0vHOflvIA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "@typescript-eslint/eslint-plugin": "8.65.0",
        "@typescript-eslint/parser": "8.65.0",
        "@typescript-eslint/typescript-estree": "8.65.0",
        "@typescript-eslint/utils": "8.65.0"
      },
      "engines": {
        "node": "^18.18.0 || ^20.9.0 || >=21.1.0"
      },
      "funding": {
        "type": "opencollective",
        "url": "https://opencollective.com/typescript-eslint"
      },
      "peerDependencies": {
        "eslint": "^8.57.0 || ^9.0.0 || ^10.0.0",
        "typescript": ">=4.8.4 <6.1.0"
      }
    },
    "node_modules/unbox-primitive": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/unbox-primitive/-/unbox-primitive-1.1.0.tgz",
      "integrity": "sha512-nWJ91DjeOkej/TA8pXQ3myruKpKEYgqvpw9lz4OPHj/NWFNluYrjbz9j01CJ8yKQd2g4jFoOkINCTW2I5LEEyw==",
      "dev": true,
      "dependencies": {
        "call-bound": "^1.0.3",
        "has-bigints": "^1.0.2",
        "has-symbols": "^1.1.0",
        "which-boxed-primitive": "^1.1.1"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/undici-types": {
      "version": "8.3.0",
      "resolved": "https://registry.npmjs.org/undici-types/-/undici-types-8.3.0.tgz",
      "integrity": "sha512-j375ScV60dom+YkPFIfTLcOiPxkN/buHz5GobjLhixFuANaNs3C9l4GmrWqejgXWJ7BbJcFYpTEUkS1Ge8bpZQ==",
      "dev": true,
      "license": "MIT"
    },
    "node_modules/unrs-resolver": {
      "version": "1.12.2",
      "resolved": "https://registry.npmjs.org/unrs-resolver/-/unrs-resolver-1.12.2.tgz",
      "integrity": "sha512-dmlRxBJJayXjqTwC+JtF1HhJmgf3ftQ3YejFcZrf4+KKtJv0qDsK1pjqaaVjG7wJ5NJ6UVP1OqRMQ71Z4C3rxQ==",
      "dev": true,
      "hasInstallScript": true,
      "license": "MIT",
      "dependencies": {
        "napi-postinstall": "^0.3.4"
      },
      "funding": {
        "url": "https://opencollective.com/unrs-resolver"
      },
      "optionalDependencies": {
        "@unrs/resolver-binding-android-arm-eabi": "1.12.2",
        "@unrs/resolver-binding-android-arm64": "1.12.2",
        "@unrs/resolver-binding-darwin-arm64": "1.12.2",
        "@unrs/resolver-binding-darwin-x64": "1.12.2",
        "@unrs/resolver-binding-freebsd-x64": "1.12.2",
        "@unrs/resolver-binding-linux-arm-gnueabihf": "1.12.2",
        "@unrs/resolver-binding-linux-arm-musleabihf": "1.12.2",
        "@unrs/resolver-binding-linux-arm64-gnu": "1.12.2",
        "@unrs/resolver-binding-linux-arm64-musl": "1.12.2",
        "@unrs/resolver-binding-linux-loong64-gnu": "1.12.2",
        "@unrs/resolver-binding-linux-loong64-musl": "1.12.2",
        "@unrs/resolver-binding-linux-ppc64-gnu": "1.12.2",
        "@unrs/resolver-binding-linux-riscv64-gnu": "1.12.2",
        "@unrs/resolver-binding-linux-riscv64-musl": "1.12.2",
        "@unrs/resolver-binding-linux-s390x-gnu": "1.12.2",
        "@unrs/resolver-binding-linux-x64-gnu": "1.12.2",
        "@unrs/resolver-binding-linux-x64-musl": "1.12.2",
        "@unrs/resolver-binding-openharmony-arm64": "1.12.2",
        "@unrs/resolver-binding-wasm32-wasi": "1.12.2",
        "@unrs/resolver-binding-win32-arm64-msvc": "1.12.2",
        "@unrs/resolver-binding-win32-ia32-msvc": "1.12.2",
        "@unrs/resolver-binding-win32-x64-msvc": "1.12.2"
      }
    },
    "node_modules/update-browserslist-db": {
      "version": "1.2.3",
      "resolved": "https://registry.npmjs.org/update-browserslist-db/-/update-browserslist-db-1.2.3.tgz",
      "integrity": "sha512-Js0m9cx+qOgDxo0eMiFGEueWztz+d4+M3rGlmKPT+T4IS/jP4ylw3Nwpu6cpTTP8R1MAC1kF4VbdLt3ARf209w==",
      "dev": true,
      "funding": [
        {
          "type": "opencollective",
          "url": "https://opencollective.com/browserslist"
        },
        {
          "type": "tidelift",
          "url": "https://tidelift.com/funding/github/npm/browserslist"
        },
        {
          "type": "github",
          "url": "https://github.com/sponsors/ai"
        }
      ],
      "dependencies": {
        "escalade": "^3.2.0",
        "picocolors": "^1.1.1"
      },
      "bin": {
        "update-browserslist-db": "cli.js"
      },
      "peerDependencies": {
        "browserslist": ">= 4.21.0"
      }
    },
    "node_modules/uri-js": {
      "version": "4.4.1",
      "resolved": "https://registry.npmjs.org/uri-js/-/uri-js-4.4.1.tgz",
      "integrity": "sha512-7rKUyy33Q1yc98pQ1DAmLtwX109F7TIfWlW1Ydo8Wl1ii1SeHieeh0HHfPeL2fMXK6z0s8ecKs9frCuLJvndBg==",
      "dev": true,
      "license": "BSD-2-Clause",
      "dependencies": {
        "punycode": "^2.1.0"
      }
    },
    "node_modules/use-sync-external-store": {
      "version": "1.6.0",
      "resolved": "https://registry.npmjs.org/use-sync-external-store/-/use-sync-external-store-1.6.0.tgz",
      "integrity": "sha512-Pp6GSwGP/NrPIrxVFAIkOQeyw8lFenOHijQWkUTrDvrF4ALqylP2C/KCkeS9dpUM3KvYRQhna5vt7IL95+ZQ9w==",
      "license": "MIT",
      "peerDependencies": {
        "react": "^16.8.0 || ^17.0.0 || ^18.0.0 || ^19.0.0"
      }
    },
    "node_modules/utility-types": {
      "version": "3.11.0",
      "resolved": "https://registry.npmjs.org/utility-types/-/utility-types-3.11.0.tgz",
      "integrity": "sha512-6Z7Ma2aVEWisaL6TvBCy7P8rm2LQoPv6dJ7ecIaIixHcwfbJ0x7mWdbcwlIM5IGQxPZSFYeqRCqlOOeKoJYMkw==",
      "license": "MIT",
      "engines": {
        "node": ">= 4"
      }
    },
    "node_modules/webgl-constants": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/webgl-constants/-/webgl-constants-1.1.1.tgz",
      "integrity": "sha512-LkBXKjU5r9vAW7Gcu3T5u+5cvSvh5WwINdr0C+9jpzVB41cjQAP5ePArDtk/WHYdVj0GefCgM73BA7FlIiNtdg=="
    },
    "node_modules/webgl-sdf-generator": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/webgl-sdf-generator/-/webgl-sdf-generator-1.1.1.tgz",
      "integrity": "sha512-9Z0JcMTFxeE+b2x1LJTdnaT8rT8aEp7MVxkNwoycNmJWwPdzoXzMh0BjJSh/AEFP+KPYZUli814h8bJZFIZ2jA==",
      "license": "MIT"
    },
    "node_modules/which": {
      "version": "2.0.2",
      "resolved": "https://registry.npmjs.org/which/-/which-2.0.2.tgz",
      "integrity": "sha512-BLI3Tl1TW3Pvl70l3yq3Y64i+awpwXqsGBYWkkqMtnbXgrMD+yj7rhW0kuEDxzJaYXGjEW5ogapKNMEKNMjibA==",
      "license": "ISC",
      "dependencies": {
        "isexe": "^2.0.0"
      },
      "bin": {
        "node-which": "bin/node-which"
      },
      "engines": {
        "node": ">= 8"
      }
    },
    "node_modules/which-boxed-primitive": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/which-boxed-primitive/-/which-boxed-primitive-1.1.1.tgz",
      "integrity": "sha512-TbX3mj8n0odCBFVlY8AxkqcHASw3L60jIuF8jFP78az3C2YhmGvqbHBpAjTRH2/xqYunrJ9g1jSyjCjpoWzIAA==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "is-bigint": "^1.1.0",
        "is-boolean-object": "^1.2.1",
        "is-number-object": "^1.1.1",
        "is-string": "^1.1.1",
        "is-symbol": "^1.1.1"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/which-builtin-type": {
      "version": "1.2.1",
      "resolved": "https://registry.npmjs.org/which-builtin-type/-/which-builtin-type-1.2.1.tgz",
      "integrity": "sha512-6iBczoX+kDQ7a3+YJBnh3T+KZRxM/iYNPXicqk66/Qfm1b93iu+yOImkg0zHbj5LNOcNv1TEADiZ0xa34B4q6Q==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "call-bound": "^1.0.2",
        "function.prototype.name": "^1.1.6",
        "has-tostringtag": "^1.0.2",
        "is-async-function": "^2.0.0",
        "is-date-object": "^1.1.0",
        "is-finalizationregistry": "^1.1.0",
        "is-generator-function": "^1.0.10",
        "is-regex": "^1.2.1",
        "is-weakref": "^1.0.2",
        "isarray": "^2.0.5",
        "which-boxed-primitive": "^1.1.0",
        "which-collection": "^1.0.2",
        "which-typed-array": "^1.1.16"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/which-collection": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/which-collection/-/which-collection-1.0.2.tgz",
      "integrity": "sha512-K4jVyjnBdgvc86Y6BkaLZEN933SwYOuBFkdmBu9ZfkcAbdVbpITnDmjvZ/aQjRXQrv5EPkTnD1s39GiiqbngCw==",
      "dev": true,
      "license": "MIT",
      "dependencies": {
        "is-map": "^2.0.3",
        "is-set": "^2.0.3",
        "is-weakmap": "^2.0.2",
        "is-weakset": "^2.0.3"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/which-typed-array": {
      "version": "1.1.22",
      "resolved": "https://registry.npmjs.org/which-typed-array/-/which-typed-array-1.1.22.tgz",
      "integrity": "sha512-fvO4ExWMFsqyhG3AiPAObMuY1lxaqgYcxbc49CNdWDDECOJNgQyvsOWVwbZc+qf3rzRtxojBK+CMEv0Ld5CYpw==",
      "dev": true,
      "dependencies": {
        "available-typed-arrays": "^1.0.7",
        "call-bind": "^1.0.9",
        "call-bound": "^1.0.4",
        "for-each": "^0.3.5",
        "get-proto": "^1.0.1",
        "gopd": "^1.2.0",
        "has-tostringtag": "^1.0.2"
      },
      "engines": {
        "node": ">= 0.4"
      },
      "funding": {
        "url": "https://github.com/sponsors/ljharb"
      }
    },
    "node_modules/word-wrap": {
      "version": "1.2.5",
      "resolved": "https://registry.npmjs.org/word-wrap/-/word-wrap-1.2.5.tgz",
      "integrity": "sha512-BN22B5eaMMI9UMtjrGd5g5eCYPpCPDUy0FJXbYsaT5zYxjFOckS53SQDE3pWkVoWpHXVb3BrYcEN4Twa55B5cA==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=0.10.0"
      }
    },
    "node_modules/yallist": {
      "version": "3.1.1",
      "resolved": "https://registry.npmjs.org/yallist/-/yallist-3.1.1.tgz",
      "integrity": "sha512-a4UGQaWPH59mOXUYnAG2ewncQS4i4F43Tv3JoAM+s2VDAmS9NsK8GpDMLrCHPksFT7h3K6TOoUNn2pb7RoXx4g==",
      "dev": true,
      "license": "ISC"
    },
    "node_modules/yocto-queue": {
      "version": "0.1.0",
      "resolved": "https://registry.npmjs.org/yocto-queue/-/yocto-queue-0.1.0.tgz",
      "integrity": "sha512-rVksvsnNCdJ/ohGc6xgPwyN8eheCxsiLM8mxuE/t/mOVqJewPuO1miLpTHQiRgTKCLexL4MeAFVagts7HmNZ2Q==",
      "dev": true,
      "engines": {
        "node": ">=10"
      },
      "funding": {
        "url": "https://github.com/sponsors/sindresorhus"
      }
    },
    "node_modules/zod": {
      "version": "4.4.3",
      "resolved": "https://registry.npmjs.org/zod/-/zod-4.4.3.tgz",
      "integrity": "sha512-ytENFjIJFl2UwYglde2jchW2Hwm4GJFLDiSXWdTrJQBIN9Fcyp7n4DhxJEiWNAJMV1/BqWfW/kkg71UDcHJyTQ==",
      "dev": true,
      "license": "MIT",
      "funding": {
        "url": "https://github.com/sponsors/colinhacks"
      }
    },
    "node_modules/zod-validation-error": {
      "version": "4.0.2",
      "resolved": "https://registry.npmjs.org/zod-validation-error/-/zod-validation-error-4.0.2.tgz",
      "integrity": "sha512-Q6/nZLe6jxuU80qb/4uJ4t5v2VEZ44lzQjPDhYJNztRQ4wyWc6VF3D3Kb/fAuPetZQnhS3hnajCf9CsWesghLQ==",
      "dev": true,
      "license": "MIT",
      "engines": {
        "node": ">=18.0.0"
      },
      "peerDependencies": {
        "zod": "^3.25.0 || ^4.0.0"
      }
    },
    "node_modules/zustand": {
      "version": "5.0.14",
      "resolved": "https://registry.npmjs.org/zustand/-/zustand-5.0.14.tgz",
      "integrity": "sha512-/8tAspM5LMPr28b3fwLYrtdj77ECpfZviaP75CMTnwO8ISyaE4GDIG/9rDDYq/cH9D2Xw2A2RXglLInmVBQB/g==",
      "license": "MIT",
      "engines": {
        "node": ">=12.20.0"
      },
      "peerDependencies": {
        "@types/react": ">=18.0.0",
        "immer": ">=9.0.6",
        "react": ">=18.0.0",
        "use-sync-external-store": ">=1.2.0"
      },
      "peerDependenciesMeta": {
        "@types/react": {
          "optional": true
        },
        "immer": {
          "optional": true
        },
        "react": {
          "optional": true
        },
        "use-sync-external-store": {
          "optional": true
        }
      }
    }
  }
}
```

## `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": [
      "dom",
      "dom.iterable",
      "esnext"
    ],
    "allowJs": false,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "react-jsx",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": [
        "./*"
      ]
    }
  },
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts",
    ".next/dev/types/**/*.ts"
  ],
  "exclude": [
    "node_modules"
  ]
}
```
