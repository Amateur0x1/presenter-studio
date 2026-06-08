# Style Presets Reference

Curated visual styles for Presenter Studio. Each preset provides colors, fonts, and signature elements. **Abstract shapes only — no illustrations.**

**Viewport CSS:** For mandatory base styles, see [viewport-base.css](viewport-base.css). Include in every presentation.

---

## Dark Themes

### 1. Midnight Studio

**Vibe:** Confident, professional, cinematic

**Typography:**
- Display: `Outfit` (700/800)
- Body: `Inter` (400/500)

**Colors:**
```css
:root {
    --bg-primary: #0f0f13;
    --bg-secondary: #1a1a22;
    --text-primary: #f0f0f2;
    --text-secondary: #8a8a9a;
    --accent: #6366f1;
    --accent-glow: rgba(99, 102, 241, 0.25);
    --slide-bg: #0f0f13;
    --stage-bg: #000;
}
```

**Signature Elements:**
- Deep black background with subtle indigo glow
- Thin accent lines as dividers
- Large, cinematic typography with generous tracking
- Recording panel blends naturally into dark theme

---

### 2. Neon Broadcast

**Vibe:** Bold, tech-forward, live-streaming aesthetic

**Typography:**
- Display: `Space Grotesk` (700)
- Body: `DM Sans` (400/500)

**Colors:**
```css
:root {
    --bg-primary: #0a0e1a;
    --text-primary: #ffffff;
    --text-secondary: #6b7394;
    --accent: #00ffaa;
    --accent-secondary: #ff3366;
    --slide-bg: #0a0e1a;
    --stage-bg: #000;
}
```

**Signature Elements:**
- Neon green accent for "live" feeling
- Grid pattern overlay (subtle)
- Glowing recording indicator matches aesthetic
- Scanline texture on hover states

---

### 3. Dark Editorial

**Vibe:** Quiet authority, literary, scholarly

**Typography:**
- Display: `Cormorant` (600/700)
- Body: `Source Sans 3` (400/500)

**Colors:**
```css
:root {
    --bg-primary: #1c1917;
    --text-primary: #e7e5e4;
    --text-secondary: #a8a29e;
    --accent: #d4a574;
    --accent-secondary: #78716c;
    --slide-bg: #1c1917;
    --stage-bg: #0c0a09;
}
```

**Signature Elements:**
- Warm stone tones on deep charcoal
- Serif display for gravitas
- Horizontal rules as section breaks
- Understated, confident spacing

---

## Light Themes

### 4. Paper Canvas

**Vibe:** Clean, creative, warm, approachable

**Typography:**
- Display: `Syne` (700/800)
- Body: `Work Sans` (400/500)

**Colors:**
```css
:root {
    --bg-primary: #faf8f5;
    --text-primary: #1a1a1a;
    --text-secondary: #666666;
    --accent: #e85d04;
    --accent-secondary: #264653;
    --slide-bg: #faf8f5;
    --stage-bg: #e8e4df;
}
```

**Signature Elements:**
- Warm cream paper background
- Bold orange accent for energy
- Rounded elements and soft shadows
- Text blocks feel like sticky notes on a wall

---

### 5. Swiss Grid

**Vibe:** Precise, minimal, Bauhaus-inspired

**Typography:**
- Display: `Archivo` (800/900)
- Body: `Archivo` (400/500)

**Colors:**
```css
:root {
    --bg-primary: #ffffff;
    --text-primary: #000000;
    --text-secondary: #555555;
    --accent: #ff3300;
    --accent-secondary: #0033ff;
    --slide-bg: #ffffff;
    --stage-bg: #f0f0f0;
}
```

**Signature Elements:**
- Pure black and white with red accent
- Visible grid structure
- Asymmetric layouts
- Sharp geometric decorations

---

### 6. Soft Morning

**Vibe:** Gentle, calm, wellness-inspired

**Typography:**
- Display: `Plus Jakarta Sans` (700)
- Body: `Plus Jakarta Sans` (400/500)

**Colors:**
```css
:root {
    --bg-primary: #f8f6f3;
    --text-primary: #2d2d2d;
    --text-secondary: #7a7a7a;
    --accent: #7c9eb2;
    --accent-secondary: #c4a882;
    --slide-bg: #f8f6f3;
    --stage-bg: #eae6e1;
}
```

**Signature Elements:**
- Muted blue-grey and warm sand accents
- Rounded corners everywhere
- Generous whitespace
- Soft, ambient gradient backgrounds

---

## Recording-Optimized Notes

When choosing styles for recording-heavy presentations, consider:

1. **Dark themes** work better on camera — presenter's face is more visible against dark backgrounds
2. **High contrast** text ensures readability even at lower recording resolution (720p)
3. **Avoid busy patterns** — they create encoding artifacts in compressed video
4. **Large fonts** (96px+) remain legible in screen recordings viewed on mobile
5. **Accent colors** for the recording indicator should contrast with the theme's primary palette

---

## Font Pairing Quick Reference

| Preset | Display Font | Body Font | Source |
|--------|--------------|-----------|--------|
| Midnight Studio | Outfit | Inter | Google |
| Neon Broadcast | Space Grotesk | DM Sans | Google |
| Dark Editorial | Cormorant | Source Sans 3 | Google |
| Paper Canvas | Syne | Work Sans | Google |
| Swiss Grid | Archivo | Archivo | Google |
| Soft Morning | Plus Jakarta Sans | Plus Jakarta Sans | Google |

---

## CSS Gotchas

### Negating CSS Functions

**WRONG — silently ignored by browsers:**
```css
right: -clamp(28px, 3.5vw, 44px);
```

**CORRECT — wrap in `calc()`:**
```css
right: calc(-1 * clamp(28px, 3.5vw, 44px));
```

### Recording Panel Z-Index

The recording panel must sit above everything:
```css
.recording-panel { z-index: 9999; }
.deck-viewport { z-index: 1; }
```

Never apply `transform` on the recording panel's parent — it creates a new stacking context and breaks `position: fixed`.
