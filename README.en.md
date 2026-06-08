# Presenter Studio

A video-recordable HTML presentation tool. Single file, zero dependencies, runs directly in the browser.

## Features

- **Tab Recording** — Captures only the current page content, excluding browser chrome and toolbars
- **Camera Recording** — Records your face in the background with no visible preview on screen
- **Microphone Capture** — Your voice is recorded into the camera video file
- **Dual File Export** — On stop, pick a folder and get `slides-xxx.mp4` + `camera-xxx.mp4` saved together
- **Editable Text** — Double-click any text block to edit, drag to reposition
- **Clean Capture** — Recording toolbar completely disappears during recording so it's never in the video
- **High Clarity** — 8 Mbps for slides, 4 Mbps for camera, sharp text guaranteed

## Quick Start

```bash
# 1. Navigate to the directory
cd ~/self/presenter-studio

# 2. Start a local server (recording APIs require localhost secure context)
python3 -m http.server 8080

# 3. Open in browser
open http://localhost:8080/examples/demo.html
```

> ⚠️ You cannot open the file directly via `file://` — recording will not work.

## Controls

| Action | Method |
|--------|--------|
| Navigate slides | ← → arrow keys, or touch swipe |
| Start recording | Click "● 开始" in the top toolbar |
| Pause / Resume | Press `Space` |
| Stop recording | Press `Esc` |
| Edit text | Double-click a text block |
| Move text | Drag a text block |
| Exit editing | Press `Esc` or click outside |

## Recording Flow

1. Click "Start" → browser requests microphone, tab capture, and camera permissions in sequence
2. Once granted, the toolbar hides automatically and recording begins
3. Present normally — all navigation and edits are captured in the slides video
4. Press `Space` to pause, press again to resume
5. Press `Esc` to stop → folder picker appears → both video files are saved to the chosen folder

## Output Files

| File | Contents |
|------|----------|
| `slides-[timestamp].mp4` | Tab video + tab system audio |
| `camera-[timestamp].mp4` | Camera video + microphone audio |

Format priority is MP4 (H.264+AAC), falling back to WebM if unsupported.

## File Structure

```
presenter-studio/
├── README.md                ← Chinese documentation
├── README.en.md             ← You are here
├── SKILL.md                 ← AI Skill definition for automated generation
├── template.html            ← Blank template — fill in content to create a new deck
│
├── docs/                    ← Reference documentation
│   ├── STYLE_PRESETS.md     ← 6 color scheme presets
│   ├── html-template.md     ← HTML structure reference
│   ├── recording-module.md  ← DualRecorder implementation reference
│   └── viewport-base.css   ← Fixed 16:9 stage CSS
│
└── examples/                ← Examples
    └── demo.html            ← Full demo with example slides (try it out)
```

## Customization

Edit the `:root` CSS variables in the HTML file to quickly re-skin:

```css
:root {
    --bg-primary: #0f0f13;      /* Slide background */
    --text-primary: #f0f0f2;    /* Primary text color */
    --accent: #6366f1;          /* Accent color */
    --font-display: 'Outfit';   /* Heading font */
    --font-body: 'Inter';       /* Body font */
    --title-size: 96px;         /* Title font size */
}
```

## Creating a New Presentation

Copy `template.html` and add slides inside `<main class="deck-stage">`:

```html
<section class="slide active" data-slide="1">
    <div class="text-block" style="top: 300px; left: 140px;" contenteditable="false">
        <h1>Your Title</h1>
        <p style="margin-top: 20px;">Your content here</p>
    </div>
</section>

<section class="slide" data-slide="2">
    <div class="text-block" style="top: 200px; left: 140px;" contenteditable="false">
        <h2>Second Slide</h2>
        <p style="margin-top: 16px;">More content...</p>
    </div>
</section>
```

Coordinates are based on a 1920×1080 canvas. The page automatically scales to fit the browser window.

## Browser Compatibility

| Browser | Recording | Folder Export |
|---------|-----------|---------------|
| Chrome | ✅ | ✅ |
| Edge | ✅ | ✅ |
| Firefox | ✅ (WebM) | ❌ (falls back to download) |
| Safari | ❌ | ❌ |

Chrome or Edge recommended for the full experience.
