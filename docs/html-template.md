# HTML Presentation + Recording Template

Reference architecture for generating slide presentations with built-in video recording and text editing capabilities. Every presentation follows a fixed 16:9 stage model: slides are authored at 1920×1080 and the whole stage scales to fit the browser window.

## Base HTML Structure

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Presentation Title</title>

    <!-- Fonts: use Fontshare or Google Fonts — never system fonts -->
    <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=...">

    <style>
        /* ===========================================
           CSS CUSTOM PROPERTIES (THEME)
           =========================================== */
        :root {
            /* Colors */
            --bg-primary: #0a0f1c;
            --text-primary: #ffffff;
            --accent: #00ffcc;

            /* Typography — authored at 1920×1080 stage size */
            --font-display: 'Clash Display', sans-serif;
            --font-body: 'Satoshi', sans-serif;
            --title-size: 112px;
            --subtitle-size: 34px;
            --body-size: 28px;

            /* Spacing */
            --slide-padding: 72px;
            --content-gap: 32px;

            /* Animation */
            --ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
            --duration-normal: 0.6s;
        }

        /* ===========================================
           BASE STYLES
           =========================================== */
        * { margin: 0; padding: 0; box-sizing: border-box; }

        /* --- PASTE viewport-base.css CONTENTS HERE --- */

        /* ===========================================
           ANIMATIONS
           =========================================== */
        .reveal {
            opacity: 0;
            transform: translateY(30px);
            transition: opacity var(--duration-normal) var(--ease-out-expo),
                        transform var(--duration-normal) var(--ease-out-expo);
        }

        .slide.visible .reveal {
            opacity: 1;
            transform: translateY(0);
        }

        .reveal:nth-child(1) { transition-delay: 0.1s; }
        .reveal:nth-child(2) { transition-delay: 0.2s; }
        .reveal:nth-child(3) { transition-delay: 0.3s; }
        .reveal:nth-child(4) { transition-delay: 0.4s; }

        /* ===========================================
           EDITABLE TEXT BLOCKS
           =========================================== */
        .text-block {
            position: absolute;
            min-width: 200px;
            min-height: 40px;
            padding: 12px 16px;
            border: 2px solid transparent;
            border-radius: 6px;
            cursor: move;
            user-select: none;
            transition: border-color 0.2s, box-shadow 0.2s;
        }

        .text-block:hover {
            border-color: var(--accent, #00ffcc);
        }

        .text-block.editing {
            border-color: var(--accent, #00ffcc);
            box-shadow: 0 0 0 3px rgba(0, 255, 204, 0.2);
            cursor: text;
            user-select: text;
        }

        .text-block.dragging {
            opacity: 0.8;
            z-index: 100;
        }

        /* ===========================================
           RECORDING UI (overlays the viewport, not the stage)
           =========================================== */
        .recording-panel {
            position: fixed;
            top: 16px;
            right: 16px;
            z-index: 9999;
            display: flex;
            flex-direction: column;
            gap: 8px;
            align-items: flex-end;
        }

        .recording-controls {
            display: flex;
            gap: 8px;
            align-items: center;
            padding: 8px 16px;
            background: rgba(0, 0, 0, 0.85);
            backdrop-filter: blur(8px);
            border-radius: 24px;
            color: #fff;
            font-family: system-ui, sans-serif;
            font-size: 13px;
        }

        .recording-controls button {
            padding: 6px 14px;
            border: none;
            border-radius: 16px;
            font-size: 12px;
            font-weight: 600;
            cursor: pointer;
            transition: transform 0.15s, opacity 0.15s;
        }

        .recording-controls button:hover {
            transform: scale(1.05);
        }

        .btn-record {
            background: #ff4444;
            color: #fff;
        }

        .btn-stop {
            background: #666;
            color: #fff;
        }

        .camera-preview {
            width: 180px;
            height: 135px;
            border-radius: 12px;
            overflow: hidden;
            border: 2px solid rgba(255, 255, 255, 0.3);
            background: #000;
        }

        .camera-preview video {
            width: 100%;
            height: 100%;
            object-fit: cover;
            transform: scaleX(-1); /* Mirror for natural feel */
        }

        .recording-indicator {
            display: none;
            align-items: center;
            gap: 6px;
            color: #ff4444;
            font-weight: 600;
        }

        .recording-indicator.active {
            display: flex;
        }

        .recording-dot {
            width: 8px;
            height: 8px;
            background: #ff4444;
            border-radius: 50%;
            animation: pulse 1.2s infinite;
        }

        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.3; }
        }
    </style>
</head>
<body>
    <!-- Recording UI Panel -->
    <div class="recording-panel">
        <div class="camera-preview">
            <video id="cameraPreview" autoplay muted playsinline></video>
        </div>
        <div class="recording-controls">
            <span class="recording-indicator" id="recIndicator">
                <span class="recording-dot"></span>
                <span id="recTimer">00:00</span>
            </span>
            <button class="btn-record" id="btnRecord">开始录制</button>
            <button class="btn-stop" id="btnStop" disabled>停止</button>
        </div>
    </div>

    <!-- Presentation -->
    <div class="deck-viewport">
        <main class="deck-stage" id="deckStage">
            <section class="slide title-slide active">
                <div class="text-block" style="top: 300px; left: 200px;"
                     contenteditable="false">
                    <h1>Presentation Title</h1>
                </div>
                <div class="text-block" style="top: 500px; left: 200px;"
                     contenteditable="false">
                    <p>Subtitle or author</p>
                </div>
            </section>

            <section class="slide">
                <div class="text-block" style="top: 100px; left: 120px;"
                     contenteditable="false">
                    <h2>Slide Title</h2>
                </div>
                <div class="text-block" style="top: 250px; left: 120px;"
                     contenteditable="false">
                    <p>Content goes here...</p>
                </div>
            </section>
        </main>
    </div>

    <script>
        /* ===========================================
           SLIDE PRESENTATION CONTROLLER
           =========================================== */
        class SlidePresentation { /* ... */ }

        /* ===========================================
           EDITABLE TEXT BLOCK CONTROLLER
           =========================================== */
        class TextBlockEditor { /* ... */ }

        /* ===========================================
           DUAL-STREAM VIDEO RECORDER
           =========================================== */
        class DualRecorder { /* ... */ }

        // Initialize
        new SlidePresentation();
        new TextBlockEditor();
        new DualRecorder();
    </script>
</body>
</html>
```

## Required JavaScript Modules

### 1. SlidePresentation — Slide Navigation

- Keyboard navigation (arrows, space, page up/down)
- Touch/swipe support
- Stage scaling (1920×1080 → viewport)
- Progress indicator outside the stage

### 2. TextBlockEditor — Editable Text Blocks

- Double-click to enter edit mode (`contenteditable="true"`)
- Click outside or Escape to exit edit mode
- Drag to reposition (mousedown on border area)
- Resize handle (bottom-right corner)
- Supports multiple text blocks per slide
- Text styling toolbar (optional): bold, italic, font size, color

### 3. DualRecorder — Screen + Camera Recording

- **Camera stream**: `getUserMedia({ video: true, audio: true })`
- **Screen stream**: `getDisplayMedia({ video: true, audio: true })`
- Two independent `MediaRecorder` instances
- Timer display during recording
- On stop: generate two `.webm` downloads (screen + camera)
- Camera preview always visible in corner

## Recording Implementation Notes

```javascript
// Key: two separate MediaRecorder instances
const cameraRecorder = new MediaRecorder(cameraStream, { mimeType: 'video/webm;codecs=vp9' });
const screenRecorder = new MediaRecorder(screenStream, { mimeType: 'video/webm;codecs=vp9' });

// Each collects its own chunks
const cameraChunks = [];
const screenChunks = [];

cameraRecorder.ondataavailable = (e) => cameraChunks.push(e.data);
screenRecorder.ondataavailable = (e) => screenChunks.push(e.data);

// On stop: create two blobs and trigger two downloads
cameraRecorder.onstop = () => {
    const blob = new Blob(cameraChunks, { type: 'video/webm' });
    downloadBlob(blob, 'camera-recording.webm');
};
screenRecorder.onstop = () => {
    const blob = new Blob(screenChunks, { type: 'video/webm' });
    downloadBlob(blob, 'screen-recording.webm');
};
```

## Text Block Implementation Notes

```javascript
class TextBlockEditor {
    constructor() {
        this.blocks = document.querySelectorAll('.text-block');
        this.activeBlock = null;
        this.isDragging = false;
        this.setupBlocks();
    }

    setupBlocks() {
        this.blocks.forEach(block => {
            // Double-click to edit
            block.addEventListener('dblclick', () => this.enterEdit(block));

            // Drag to move
            block.addEventListener('mousedown', (e) => {
                if (block.getAttribute('contenteditable') === 'true') return;
                this.startDrag(block, e);
            });
        });

        // Click outside to exit edit
        document.addEventListener('click', (e) => {
            if (this.activeBlock && !this.activeBlock.contains(e.target)) {
                this.exitEdit();
            }
        });
    }

    enterEdit(block) {
        this.exitEdit();
        block.setAttribute('contenteditable', 'true');
        block.classList.add('editing');
        block.focus();
        this.activeBlock = block;
    }

    exitEdit() {
        if (this.activeBlock) {
            this.activeBlock.setAttribute('contenteditable', 'false');
            this.activeBlock.classList.remove('editing');
            this.activeBlock = null;
        }
    }

    startDrag(block, e) {
        this.isDragging = true;
        block.classList.add('dragging');
        const rect = block.getBoundingClientRect();
        const offsetX = e.clientX - rect.left;
        const offsetY = e.clientY - rect.top;

        const onMove = (e) => {
            // Convert screen coords back to stage coords
            // (account for stage scale factor)
            const stage = document.getElementById('deckStage');
            const stageRect = stage.getBoundingClientRect();
            const scale = stageRect.width / 1920;
            const x = (e.clientX - stageRect.left - offsetX * scale) / scale;
            const y = (e.clientY - stageRect.top - offsetY * scale) / scale;
            block.style.left = `${x}px`;
            block.style.top = `${y}px`;
        };

        const onUp = () => {
            this.isDragging = false;
            block.classList.remove('dragging');
            document.removeEventListener('mousemove', onMove);
            document.removeEventListener('mouseup', onUp);
        };

        document.addEventListener('mousemove', onMove);
        document.addEventListener('mouseup', onUp);
    }
}
```

## Code Quality

- Every section needs clear `/* === SECTION NAME === */` comments
- Semantic HTML (`<section>`, `<main>`)
- Keyboard navigation works fully
- ARIA labels for recording controls
- `prefers-reduced-motion` support
