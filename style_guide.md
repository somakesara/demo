# Enterprise Microsite Style Guide

## Design Principles

**Hierarchy:** Typography drives content priority. Hero eyebrow → H1 → lead → body creates clear information flow.

**Contrast:** WCAG AA minimum (4.5:1 text/background). Primary accent on neutral backgrounds preserves readability. Dark hero uses light text with high opacity for subtlety.

**Spacing:** Grid-based (6px unit). Padding 18-22px in cards. Sections use 72px vertical (34px gap). Tightens to 24px on mobile.

**Motion:** Smooth scroll behavior. Transitions on hover (0.2s). Toggle rotation 45deg signals interaction. Hero terminal uses `fadeUp` + `blink` animations. Every motion has purpose.

**Accessibility:** `tabindex="0"` + `focus-within` on interactive headers. `aria-label` on toggles. Semantic HTML. Color not the only differentiator (tags have uppercase text, callouts have border + background).

---

## Color System (Fixed — No Dynamic Theming)

The production microsites use a fixed full palette, not a two-variable dynamic system. Always include all variables:

```css
:root {
  --ink: #191714;       /* Near-black text */
  --paper: #fbfaf7;     /* Near-white bg */
  --warm: #f4efe6;      /* Card backgrounds */
  --warm2: #ebe4d8;     /* kbd/code inline bg */
  --line: #ded6c8;      /* Borders, dividers */
  --muted: #766f64;     /* Secondary text, nav default */
  --muted2: #9a9184;    /* Tertiary text, status lines */
  --dark: #15120f;      /* Dark panels, toggle filled state */
  --code: #f0ebe2;      /* Code block background */

  /* SEMANTIC — choose 1–2 as accent per microsite */
  --clay: #b65f34;      /* Warm orange-brown (agents, warnings) */
  --blue: #245c8c;      /* Cool blue (info, nav accent) */
  --green: #26724c;     /* Green (harness, success) */
  --plum: #6a3f83;      /* Purple (advanced, output) */
  --gold: #a77a25;      /* Amber (highlights, invocations) */
  --red: #9e3830;       /* Red (errors, danger, warn) */

  --serif: "Cormorant Garamond", Georgia, serif;
  --sans: "DM Sans", system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  --mono: "DM Mono", Consolas, monospace;
}
```

**Per-site accent:** Pick one or two colors as the primary accent. Wire them to nav hover/active, `.section-no`, `.section-label`, `.eyebrow`, `.tag` defaults, `.callout` border, `.scene` border, table `th` color, `.fc-label` default. Everything else inherits from the neutral palette.

**Theme examples:**
```
Harness Engineering:  green + gold
Claude Skills:        blue + gold
Claude Code Training: blue + plum
Claude Agents:        clay + gold
```

---

## Typography

```html
<!-- Add to <head> -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,300;0,400;0,600;1,300;1,400&family=DM+Mono:wght@300;400&family=DM+Sans:wght@300;400;500;600&display=swap" rel="stylesheet">
```

```css
body {
  font-family: var(--sans);
  font-size: 17px;
  font-weight: 300;
  line-height: 1.72;
  color: var(--ink);
  background: var(--paper);
  margin: 0;
  overflow-x: hidden;
}

/* Reset */
* { box-sizing: border-box; }
html { scroll-behavior: smooth; }
a { color: inherit; }
pre { margin: 0; white-space: pre; overflow-x: auto; }
code, kbd {
  font-family: var(--mono);
  font-size: .88em;
  background: var(--warm2);
  border: 1px solid var(--line);
  border-radius: 5px;
  padding: .08rem .32rem;
}

h1, h2, h3 { font-family: var(--serif); font-weight: 300; line-height: 1.06; margin: 0; }
h1 { font-size: clamp(44px, 8vw, 96px); max-width: 850px; }
h2 { font-size: clamp(34px, 4.4vw, 56px); max-width: 880px; }
h3 { font-size: 27px; }

h1 em { color: var(--accent-light); font-style: italic; }   /* hero emphasis */

.eyebrow { font-family: var(--mono); font-size: 11px; letter-spacing: .24em; text-transform: uppercase; color: var(--accent-light); margin-bottom: 18px; }
.section-label { font-family: var(--mono); font-size: 11px; letter-spacing: .2em; text-transform: uppercase; color: var(--accent); margin: 8px 0; }
.lead { max-width: 760px; color: #4c453d; font-size: 18px; margin: 18px 0 0; }
.section-status { margin-top: 12px; color: var(--muted2); font-family: var(--mono); font-size: 11px; letter-spacing: .08em; text-transform: uppercase; }
```

---

## Hero Section

The hero is the defining visual moment. It must earn attention.

```css
.hero {
  min-height: 92vh;
  position: relative;
  overflow: hidden;
  display: grid;
  grid-template-rows: 1fr auto;
  color: #fbf5eb;
  /* Customize radial gradients per theme — adjust hue/opacity */
  background:
    radial-gradient(ellipse 64% 58% at 14% 48%, rgba(36,92,140,.32), transparent 62%),
    radial-gradient(ellipse 52% 68% at 83% 22%, rgba(106,63,131,.28), transparent 60%),
    linear-gradient(135deg, #0c0f18 0%, #10100f 52%, #14100c 100%);
}

.hero-grid {
  position: absolute;
  inset: 0;
  background-image:
    linear-gradient(rgba(255,255,255,.035) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255,255,255,.035) 1px, transparent 1px);
  background-size: 64px 64px;
  mask-image: radial-gradient(ellipse 85% 75% at 50% 45%, black 35%, transparent 100%);
}

.hero-inner {
  position: relative;
  z-index: 2;
  width: min(1160px, 100%);
  margin: 0 auto;
  padding: 92px 44px 48px;
  display: grid;
  align-content: center;
}

.hero-sub {
  max-width: 640px;
  margin-top: 24px;
  color: rgba(251,245,235,.66);
  font-size: clamp(16px, 2vw, 20px);
  line-height: 1.62;
}

.hero-actions { display: flex; flex-wrap: wrap; gap: 12px; margin-top: 30px; }

.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-height: 42px;
  padding: 10px 16px;
  border: 1px solid rgba(251,245,235,.22);
  border-radius: 8px;
  color: #fbf5eb;
  background: rgba(251,245,235,.07);
  text-decoration: none;
  font-family: var(--mono);
  font-size: 12px;
  letter-spacing: .04em;
}
.btn.primary { background: var(--accent-light); color: #17130f; border-color: var(--accent-light); }

.scroll-hint {
  position: relative;
  z-index: 2;
  padding: 0 44px 34px;
  text-align: center;
  color: rgba(251,245,235,.34);
  font-family: var(--mono);
  font-size: 11px;
  letter-spacing: .14em;
  text-transform: uppercase;
}
```

### Hero Terminal / Flow Card Panel

Position a floating panel (terminal or conversation trace) on the right side of the hero. It gives visual proof of what the product does before the user scrolls.

```css
/* Floating panel — position absolute, right-anchored */
.hero-terminal {
  position: absolute;
  inset: auto 5vw 7vh auto;
  width: min(460px, 40vw);
  min-width: 290px;
  opacity: .86;
}

/* Terminal window chrome */
.term-window {
  border: 1px solid rgba(251,245,235,.14);
  background: rgba(14,16,20,.82);
  backdrop-filter: blur(10px);
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 28px 72px rgba(0,0,0,.38);
}

.term-bar {
  background: rgba(255,255,255,.06);
  padding: 10px 14px;
  display: flex;
  align-items: center;
  gap: 8px;
  border-bottom: 1px solid rgba(255,255,255,.07);
}

.term-dot { width: 10px; height: 10px; border-radius: 50%; }
.term-dot.r { background: #ff5f57; }
.term-dot.y { background: #ffbd2e; }
.term-dot.g { background: #28c840; }

.term-title {
  font-family: var(--mono);
  font-size: 11px;
  color: rgba(251,245,235,.38);
  margin-left: 6px;
  letter-spacing: .06em;
}

.term-body {
  padding: 16px 18px;
  font-family: var(--mono);
  font-size: 11.5px;
  line-height: 1.7;
  color: rgba(251,245,235,.72);
}

/* Syntax colors for terminal content */
.t-prompt { color: #7ec8f5; }
.t-cmd    { color: #fbf5eb; }
.t-ok     { color: #82d9a8; }
.t-dim    { color: rgba(251,245,235,.34); }
.t-plum   { color: #c4a4f0; }
.t-gold   { color: #f0c46d; }
.t-err    { color: #f08080; }

/* Blinking cursor */
.cursor {
  display: inline-block;
  width: 7px;
  height: 1em;
  background: var(--accent-light);
  vertical-align: text-bottom;
  animation: blink 1s step-end infinite;
  margin-left: 1px;
}
@keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }

/* Staggered fade-in for terminal lines */
@keyframes fadeUp { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }
.term-line { animation: fadeUp .4s ease forwards; opacity: 0; }
.term-line:nth-child(1) { animation-delay: .1s; }
.term-line:nth-child(2) { animation-delay: .55s; }
/* etc — increment by ~0.45s per line */
```

**Why:** The terminal panel shows, not tells. A visitor sees a real conversation with Claude Code before reading a word of copy. Position it right-anchored so it does not obscure the headline. Use `opacity: .86` so it recedes behind the headline visually.

---

## Navigation

```css
.topnav {
  position: sticky;
  top: 0;
  z-index: 20;
  display: flex;
  gap: 0;
  overflow-x: auto;
  background: rgba(251,250,247,.94);
  backdrop-filter: blur(14px);
  border-bottom: 1px solid var(--line);
  scrollbar-width: none;
}
.topnav::-webkit-scrollbar { display: none; }

.topnav a {
  flex: 0 0 auto;
  padding: 15px 18px;
  text-decoration: none;
  color: var(--muted);
  border-bottom: 2px solid transparent;
  font-family: var(--mono);
  font-size: 11px;
  letter-spacing: .1em;
  text-transform: uppercase;
}
.topnav a:hover, .topnav a.active {
  color: var(--accent);
  border-color: var(--accent);
}
```

**Nav labels:** Use numbered prefixes — `01 What it is`, `02 Architecture`. They signal sequence without forcing it.

---

## Collapsible Sections

Every content section uses this pattern. The `collapsed` class hides content but keeps the section head visible.

```css
.section { border-bottom: 1px solid var(--line); }
.section.collapsed .container > :not(.section-head) { display: none; }
.section.collapsed .section-head { padding-bottom: 0; border-bottom: 0; }
.section.collapsed .container { padding-top: 34px; padding-bottom: 34px; }

.section-head {
  cursor: pointer;
  position: relative;
  padding-right: 54px;
  display: grid;
  grid-template-columns: 84px 1fr;
  gap: 28px;
  padding-bottom: 36px;
  border-bottom: 1px solid var(--line);
}
.section-head:focus-within { outline: 2px solid rgba(36,92,140,.55); outline-offset: 6px; }
.section-head:hover .section-title { color: var(--accent); }

.section-no {
  color: var(--accent);
  opacity: .16;
  font-family: var(--serif);
  font-size: 78px;
  line-height: 1;
}

.section-toggle {
  cursor: pointer;
  padding: 0;
  position: absolute;
  right: 0;
  top: 8px;
  width: 34px;
  height: 34px;
  border: 1px solid var(--line);
  border-radius: 50%;
  display: grid;
  place-items: center;
  color: var(--muted);
  background: var(--paper);
  font-family: var(--mono);
  font-size: 18px;
  transition: background .2s, color .2s, transform .2s, border-color .2s;
}
.section:not(.collapsed) .section-toggle {
  transform: rotate(45deg);
  background: var(--dark);
  color: var(--paper);
  border-color: var(--dark);
}
```

```js
function toggleSection(header) {
  const section = header.closest('.section');
  section.classList.toggle('collapsed');
}
```

**HTML pattern:**
```html
<section class="section" id="slug">
<div class="container">
<div class="section-head" onclick="toggleSection(this)" tabindex="0">
  <div class="section-no">01</div>
  <div>
    <div class="section-label">Short label</div>
    <h2 class="section-title">Full declarative title as a complete sentence.</h2>
    <p class="lead">Opening paragraph that frames the section.</p>
    <div class="section-status">Collapse ↑ / Expand ↓</div>
  </div>
  <button class="section-toggle" aria-label="toggle section">+</button>
</div>
<!-- content -->
</div>
</section>
```

---

## Layout

```css
.container { width: min(1120px, 100%); margin: 0 auto; padding: 72px 44px; }
.grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 24px; margin-top: 34px; }
.grid-3 { display: grid; grid-template-columns: repeat(3, 1fr); gap: 18px; margin-top: 30px; }
.stacked { display: grid; grid-template-columns: 1fr; gap: 18px; margin-top: 30px; }
```

---

## Panels

```css
.panel { border: 1px solid var(--line); background: var(--warm); border-radius: 8px; padding: 22px; }
.panel.white { background: #fffdfa; }
.panel.dark { background: #1b1713; color: #fbf5eb; border-color: #302720; }
.panel.blue { background: #eaf0f7; border-color: rgba(36,92,140,.2); }
.panel h3 { font-size: 27px; margin-bottom: 10px; }
.panel p { margin: 0; color: #50483f; }
.panel.dark p { color: rgba(251,245,235,.66); }
.panel.blue p { color: #1e3a52; }
```

---

## Tags

```css
.tag { display: inline-flex; margin-bottom: 12px; color: var(--accent); font-family: var(--mono); font-size: 10px; letter-spacing: .16em; text-transform: uppercase; }
.tag.green { color: var(--green); }
.tag.clay  { color: var(--clay); }
.tag.plum  { color: var(--plum); }
.tag.red   { color: var(--red); }
.tag.gold  { color: var(--gold); }
.tag.blue  { color: var(--blue); }
```

---

## Callouts

```css
.callout { margin-top: 26px; border-left: 3px solid var(--green); background: #edf4ee; padding: 16px 18px; border-radius: 8px; color: #253d2d; line-height: 1.7; }
.warn     { border-left-color: var(--red);  background: #f7ece9; color: #4a2b28; }
.info     { border-left-color: var(--blue); background: #eaf0f7; color: #1e3a52; }
.gold-callout { border-left-color: var(--gold); background: #faf5e8; color: #3d2e08; }
```

**Four types:** default (green/tip), `.warn` (red/danger), `.info` (blue/info), `.gold-callout` (gold/insight).

---

## Framework Cards

Deep-dive cards with a warm header, label pill, and multi-section body. Best for comparing concepts, architectures, or threat models.

```css
.framework-cards { display: grid; gap: 18px; margin-top: 28px; }
.framework-card { border: 1px solid var(--line); border-radius: 8px; overflow: hidden; background: #fffdfa; }

.fc-head {
  padding: 18px 20px;
  background: var(--warm);
  border-bottom: 1px solid var(--line);
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 16px;
  align-items: start;
}
.fc-title { font-family: var(--serif); font-size: 24px; line-height: 1.1; }

/* Label pill — color variants */
.fc-label { font-family: var(--mono); font-size: 10px; letter-spacing: .16em; text-transform: uppercase; padding: 4px 10px; border-radius: 12px; white-space: nowrap; }
.fc-label        { color: var(--blue);  border: 1px solid rgba(36,92,140,.4);  background: #eaf0f7; }
.fc-label.green  { color: var(--green); border: 1px solid rgba(38,114,76,.4);  background: #edf4ee; }
.fc-label.red    { color: var(--red);   border: 1px solid rgba(158,56,48,.4);  background: #f7ece9; }
.fc-label.gold   { color: var(--gold);  border: 1px solid rgba(167,122,37,.4); background: #faf5e8; }
.fc-label.plum   { color: var(--plum);  border: 1px solid rgba(106,63,131,.4); background: #f6f2f9; }

.fc-body { padding: 18px 20px; display: grid; gap: 14px; }
.fc-body h4 { font-family: var(--mono); font-size: 10px; letter-spacing: .16em; text-transform: uppercase; color: var(--accent); margin: 0 0 6px; }
.fc-body p  { font-size: 13.5px; color: #433d36; line-height: 1.65; margin: 0; }
```

---

## Scene Block (Narrative)

For real-world scenarios, case studies, first-person stories. Left border anchors it as a call-out without the warning connotation.

```css
.scene {
  margin-top: 0;
  border-left: 3px solid var(--accent);
  padding: 18px 22px;
  background: linear-gradient(to right, #eaf0f7, #fffdfa); /* adjust per theme */
  border-radius: 0 8px 8px 0;
  font-size: 16px;
  color: #3a3028;
  line-height: 1.8;
}
.scene + .scene { margin-top: 14px; }
.scene-label {
  font-family: var(--mono);
  font-size: 10px;
  letter-spacing: .2em;
  text-transform: uppercase;
  color: var(--accent);
  margin-bottom: 6px;
}
```

---

## Trace Component

Visualizes a conversation or execution flow with color-coded actor rows. Use to show exactly what happens inside a system interaction.

```css
.trace { margin-top: 28px; border: 1px solid var(--line); border-radius: 8px; overflow: hidden; }

.trace-head {
  padding: 14px 18px;
  background: var(--dark);
  color: #fbf5eb;
  font-family: var(--mono);
  font-size: 11px;
  letter-spacing: .12em;
  text-transform: uppercase;
  display: flex;
  gap: 10px;
  align-items: center;
}

/* Animated status dot */
.trace-dot { width: 8px; height: 8px; border-radius: 50%; background: #7ec8f5; flex-shrink: 0; }
.trace-dot.pulse { animation: pulse 1.8s ease-in-out infinite; }
@keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: .3; } }

.trace-row { display: grid; grid-template-columns: 130px 1fr; border-bottom: 1px solid var(--line); }
.trace-row:last-child { border-bottom: 0; }

/* Actor colors */
.trace-who { padding: 14px 16px; font-family: var(--mono); font-size: 11px; letter-spacing: .08em; text-transform: uppercase; border-right: 1px solid var(--line); display: flex; align-items: flex-start; padding-top: 16px; flex-shrink: 0; }
.trace-who.user   { color: var(--blue);  background: #f2f6fb; }
.trace-who.claude { color: var(--clay);  background: #fff8f2; }
.trace-who.tool   { color: var(--green); background: #f0f7f2; }
.trace-who.out    { color: var(--plum);  background: #f6f2f9; }
.trace-who.warn   { color: var(--red);   background: #fdf1ef; }

.trace-what { padding: 14px 18px; font-size: 14px; color: #3a3028; line-height: 1.65; background: #fffdfa; }
.trace-what code { font-size: .83em; }

/* Outcome pills */
.outcome { display: inline-block; margin-top: 6px; font-family: var(--mono); font-size: 10px; letter-spacing: .1em; padding: 2px 8px; border-radius: 10px; }
.outcome.pass    { background: #edf4ee; color: var(--green); border: 1px solid rgba(38,114,76,.3); }
.outcome.active  { background: #eaf0f7; color: var(--blue);  border: 1px solid rgba(36,92,140,.3); }
.outcome.blocked { background: #fdf1ef; color: var(--red);   border: 1px solid rgba(158,56,48,.3); }
```

**HTML pattern:**
```html
<div class="trace">
  <div class="trace-head">
    <div class="trace-dot pulse"></div>
    Execution trace — description
  </div>
  <div class="trace-row">
    <div class="trace-who user">You</div>
    <div class="trace-what">Your message or action.</div>
  </div>
  <div class="trace-row">
    <div class="trace-who claude">Claude</div>
    <div class="trace-what">What Claude reasons or decides.</div>
  </div>
  <div class="trace-row">
    <div class="trace-who tool">Tool</div>
    <div class="trace-what">Tool result. <span class="outcome active">parallel</span></div>
  </div>
  <div class="trace-row">
    <div class="trace-who out">Output</div>
    <div class="trace-what">Final result. <span class="outcome pass">done</span></div>
  </div>
</div>
```

**Why:** The trace makes invisible system behavior visible. Readers follow the exact sequence of events, see which actor does what, and understand the system as a flow — not a description. Essential for technical training content, API walkthroughs, and debugging guides.

---

## Principle List

Numbered principles or levels. Use for maturity models, design principles, failure modes, or ordered insights.

```css
.principle-list { margin-top: 28px; display: grid; gap: 16px; }
.principle { border: 1px solid var(--line); border-radius: 8px; padding: 20px; background: #fffdfa; }
.principle h4 { font-family: var(--serif); font-size: 22px; margin: 0 0 8px; color: var(--ink); }
.principle p  { margin: 0; color: #514a42; font-size: 14px; line-height: 1.7; }
```

---

## Compare Row

Side-by-side comparison with shared column headers. Use for with/without, before/after, or approach contrasts.

```css
.compare-row { display: grid; grid-template-columns: 1fr 1fr; gap: 1px; background: var(--line); border: 1px solid var(--line); border-radius: 8px; overflow: hidden; margin-top: 24px; }
.compare-cell { background: var(--paper); padding: 20px 22px; font-size: 14px; color: #3a3028; line-height: 1.65; }
.compare-cell.head { background: var(--warm); font-family: var(--mono); font-size: 11px; letter-spacing: .14em; text-transform: uppercase; color: var(--accent); padding: 13px 22px; }
.compare-cell.head.red { color: var(--red); }
```

**HTML pattern:**
```html
<div class="compare-row">
  <div class="compare-cell head">Approach A</div>
  <div class="compare-cell head red">Approach B</div>
  <div class="compare-cell">Row 1 A content.</div>
  <div class="compare-cell">Row 1 B content.</div>
  <div class="compare-cell">Row 2 A content.</div>
  <div class="compare-cell">Row 2 B content.</div>
</div>
```

---

## Workflow Steps

Numbered sequential steps with circular step numbers. Use for how-to guides, setup flows, migration plans.

```css
.workflow { margin-top: 32px; display: grid; gap: 14px; }
.step { display: grid; grid-template-columns: 54px 1fr; gap: 18px; border: 1px solid var(--line); border-radius: 8px; background: #fffdfa; padding: 18px; }
.step-no { width: 42px; height: 42px; border-radius: 50%; display: grid; place-items: center; background: var(--dark); color: var(--paper); font-family: var(--serif); font-size: 24px; flex-shrink: 0; }
.step h3  { font-size: 22px; margin-bottom: 6px; }
.step p   { margin: 0; color: #514a42; font-size: 14px; line-height: 1.65; }
```

---

## Chapter Bridge

A transitional callout that links sections conceptually. Use between major topic shifts to maintain narrative continuity.

```css
.chapter-bridge {
  margin: 40px 0 0;
  padding: 22px 24px;
  border: 1px solid var(--line);
  border-radius: 8px;
  background: linear-gradient(135deg, #f9f5ef, #f0f6ff);
  font-size: 16px;
  color: #3a3028;
  line-height: 1.82;
}
.chapter-bridge strong { color: var(--accent); }
.chapter-num {
  font-family: var(--mono);
  font-size: 10px;
  letter-spacing: .22em;
  text-transform: uppercase;
  color: var(--muted2);
  margin-bottom: 8px;
}
```

---

## Tables

```css
.table-wrap { overflow-x: auto; margin-top: 28px; }
table { width: 100%; border-collapse: collapse; min-width: 600px; font-size: 14px; }
th, td { border-bottom: 1px solid var(--line); padding: 14px 12px; text-align: left; vertical-align: top; }
th { color: var(--accent); font-family: var(--mono); font-size: 11px; letter-spacing: .14em; text-transform: uppercase; font-weight: 400; }
td { color: #423c35; }
```

---

## Code Blocks

```css
pre.codeblock, div.codeblock {
  margin: 0;
  border: 0;
  border-radius: 0;
  padding: 18px 22px;
  overflow-x: auto;
  font-family: var(--mono);
  font-size: 12.5px;
  line-height: 1.72;
  color: #2a2520;
  background: var(--code);
  white-space: pre;
  word-break: normal;
  overflow-wrap: normal;
}

/* Syntax tokens */
.codeblock .dim    { color: #918779; }
.codeblock .key    { color: #8f3d16; font-weight: 400; }
.codeblock .ok     { color: #1f7045; }
.codeblock .blue   { color: #1f5d8d; }
.codeblock .accent { color: #a07020; }
.codeblock .red    { color: #9e3830; }
.codeblock .str    { color: #2d6b2d; }
.codeblock .cmt    { color: #9a9184; font-style: italic; }
.codeblock .plum   { color: #6a3f83; }

/* Inside a panel — add border and radius */
.panel pre.codeblock, .panel div.codeblock {
  border: 1px solid var(--line);
  border-radius: 6px;
  margin-top: 14px;
  padding: 16px 18px;
}
```

---

## Disclaimer + Footer

Use at the bottom of every microsite. The disclaimer protects against over-interpretation; the footer provides attribution and date.

```css
.disclaimer { background: #f7ece9; border-top: 1px solid var(--line); padding: 28px 44px; }
.disclaimer-inner { width: min(1120px, 100%); margin: 0 auto; font-size: 14px; color: #4a2b28; line-height: 1.7; }
.disclaimer-inner h3 { font-family: var(--serif); margin: 0 0 14px; font-size: 22px; font-weight: 400; }
.disclaimer-inner p { margin: 0 0 12px; }
.disclaimer-inner p:last-child { margin-bottom: 0; }

.footer { padding: 28px 44px; text-align: center; color: var(--muted); font-family: var(--mono); font-size: 11px; letter-spacing: .08em; text-transform: uppercase; }
```

---

## Responsive Breakpoints

```css
@media(max-width: 880px) {
  .hero-terminal { position: relative; inset: auto; width: auto; margin: 0 24px 34px; }
  .hero-inner { padding: 76px 28px 28px; }
  .grid-2, .grid-3 { grid-template-columns: 1fr; }
  .trace-row { grid-template-columns: 1fr; }
  .trace-who { border-right: 0; border-bottom: 1px solid var(--line); padding-top: 12px; }
  .compare-row { grid-template-columns: 1fr; }
  .fc-head { grid-template-columns: 1fr; }
}

@media(max-width: 680px) {
  body { font-size: 16px; }
  .container { padding: 54px 24px; }
  .section-head { grid-template-columns: 1fr; gap: 8px; }
  .section-no { font-size: 58px; }
  .topnav a { padding: 13px 14px; }
}
```

---

## Navigation JS

```js
function toggleSection(header) {
  const section = header.closest('.section');
  section.classList.toggle('collapsed');
}

document.addEventListener('DOMContentLoaded', function() {
  const links = document.querySelector('.topnav').querySelectorAll('a');
  window.addEventListener('scroll', function() {
    let current = '';
    document.querySelectorAll('section[id]').forEach(section => {
      if (pageYOffset >= section.offsetTop - 200) current = section.getAttribute('id');
    });
    links.forEach(link => {
      link.classList.remove('active');
      if (link.getAttribute('href').slice(1) === current) link.classList.add('active');
    });
  });
});
```

---

## Accessibility Checklist

- **Color contrast:** All text/bg meets WCAG AA (4.5:1 minimum).
- **Keyboard nav:** `tabindex="0"` on `.section-head`. Toggle uses `<button>` (native keyboard support).
- **Focus:** `focus-within` outline on `.section-head`. Nav links show `border-color` on active.
- **ARIA:** `aria-label="toggle section"` on toggle buttons. `aria-label="Page sections"` on nav. `aria-hidden="true"` on decorative hero panels.
- **Semantic HTML:** `<header>`, `<nav>`, `<main>`, `<section>`, `<table>`, `<footer>` used correctly.
- **Color independence:** Tags use uppercase text. Callouts use border + background. Trace actors use both color and uppercase label.
- **Font sizing:** `clamp()` on all headings. Body never below 16px on mobile.

---

## Performance

- **CSS:** ~10–12KB inline (minified). No external dependencies except Google Fonts.
- **Fonts:** Preconnect + implicit font-display: swap prevents FOUT.
- **Images:** None. Hero uses CSS gradients + backdrop-filter glass.
- **JavaScript:** ~40 lines for nav scroll tracking + section toggle. No frameworks.
- **Total page weight:** 70–150KB depending on content length.

---

## How to Create a New Microsite

1. Copy `harness_engineering_microsite.html` as your template base
2. Choose 1–2 accent colors from the palette. Update nav hover, section-no, section-label, eyebrow, callout borders, table th, fc-label default
3. Update hero background radial gradient colors to match the accent palette
4. Write the hero terminal or flow panel content (show don't tell)
5. Add numbered nav links with `01 Label` format
6. Build sections: each gets a `.section-head` with a complete declarative sentence as the title
7. Mix components: trace for execution flows, framework-cards for deep dives, compare-row for with/without, principle-list for ordered insights
8. Add a `.disclaimer` + `.footer` at the bottom
9. Test at 880px (tablet) and 680px (mobile). Verify all grids stack correctly
10. Check contrast with [WebAIM](https://webaim.org/resources/contrastchecker/)

---

## Component Selection Guide

| Content type | Component | Notes |
|---|---|---|
| Real-world story, scenario | `.scene` | Use a specific narrative, name a character |
| Execution / conversation flow | `.trace` | Color-code each actor row |
| Concept with depth | `.framework-card` | 3 h4 subsections max per card |
| With vs without | `.compare-row` | Headers in `.head`, alternating rows |
| Steps or levels | `.principle-list` | serif h4 titles, 14px body |
| Sequential how-to | `.workflow` + `.step` | Numbered circles in `.step-no` |
| Section bridge / insight | `.chapter-bridge` | 1–2 sentences, bold the key claim |
| Quick comparison table | `table` inside `.table-wrap` | Min-width 600px, horizontal scroll on mobile |
| Critical insight | `.callout` / `.info` / `.warn` / `.gold-callout` | Pick variant by semantic meaning |
| Hero proof-of-concept | `.hero-terminal` | Stagger lines with animation-delay |
