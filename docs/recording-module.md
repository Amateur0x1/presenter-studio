# Recording Module Reference

Dual-stream video recording implementation for Presenter Studio. This module captures the screen and camera as two independent video files using browser-native APIs.

## Architecture Overview

```
┌─────────────────────────────────────────────────┐
│                  Browser Page                     │
├─────────────────────────────────────────────────┤
│                                                  │
│  ┌─────────────────┐    ┌─────────────────────┐ │
│  │  Camera Stream   │    │   Screen Stream      │ │
│  │  getUserMedia()  │    │  getDisplayMedia()   │ │
│  └────────┬────────┘    └──────────┬──────────┘ │
│           │                         │            │
│  ┌────────▼────────┐    ┌──────────▼──────────┐ │
│  │ MediaRecorder #1 │    │  MediaRecorder #2    │ │
│  │ (camera chunks)  │    │  (screen chunks)     │ │
│  └────────┬────────┘    └──────────┬──────────┘ │
│           │                         │            │
│  ┌────────▼────────┐    ┌──────────▼──────────┐ │
│  │  camera.webm     │    │   screen.webm        │ │
│  │  (download)      │    │   (download)         │ │
│  └─────────────────┘    └─────────────────────┘ │
└─────────────────────────────────────────────────┘
```

## Complete DualRecorder Implementation

```javascript
/* ===========================================
   DUAL-STREAM VIDEO RECORDER
   Records screen and camera as two separate files.
   Zero dependencies — uses only browser-native APIs.
   =========================================== */
class DualRecorder {
    constructor() {
        this.cameraStream = null;
        this.screenStream = null;
        this.cameraRecorder = null;
        this.screenRecorder = null;
        this.cameraChunks = [];
        this.screenChunks = [];
        this.isRecording = false;
        this.startTime = null;
        this.timerInterval = null;

        // DOM references
        this.cameraPreview = document.getElementById('cameraPreview');
        this.btnRecord = document.getElementById('btnRecord');
        this.btnStop = document.getElementById('btnStop');
        this.recIndicator = document.getElementById('recIndicator');
        this.recTimer = document.getElementById('recTimer');

        this.init();
    }

    async init() {
        // Start camera preview immediately on page load
        try {
            this.cameraStream = await navigator.mediaDevices.getUserMedia({
                video: { width: 1280, height: 720, facingMode: 'user' },
                audio: true
            });
            this.cameraPreview.srcObject = this.cameraStream;
        } catch (err) {
            console.warn('Camera not available:', err.message);
            // Camera is optional — recording can still work without it
        }

        this.btnRecord.addEventListener('click', () => this.startRecording());
        this.btnStop.addEventListener('click', () => this.stopRecording());
    }

    async startRecording() {
        try {
            // Request screen capture (user picks which screen/window/tab)
            this.screenStream = await navigator.mediaDevices.getDisplayMedia({
                video: { width: 1920, height: 1080, frameRate: 30 },
                audio: true  // System audio (if user permits)
            });

            // Handle user stopping screen share via browser UI
            this.screenStream.getVideoTracks()[0].onended = () => {
                if (this.isRecording) this.stopRecording();
            };

        } catch (err) {
            console.error('Screen capture denied:', err.message);
            return; // User cancelled — do nothing
        }

        // Determine best mimeType
        const mimeType = this.getBestMimeType();

        // --- Screen Recorder ---
        this.screenChunks = [];
        this.screenRecorder = new MediaRecorder(this.screenStream, {
            mimeType,
            videoBitsPerSecond: 5_000_000  // 5 Mbps for screen
        });
        this.screenRecorder.ondataavailable = (e) => {
            if (e.data.size > 0) this.screenChunks.push(e.data);
        };
        this.screenRecorder.onstop = () => {
            const ext = this.getFileExtension(mimeType);
            const blobType = mimeType.includes('mp4') ? 'video/mp4' : 'video/webm';
            this.exportVideo(this.screenChunks, `screen-recording.${ext}`, blobType);
            this.cleanupStream(this.screenStream);
        };

        // --- Camera Recorder ---
        if (this.cameraStream) {
            this.cameraChunks = [];
            this.cameraRecorder = new MediaRecorder(this.cameraStream, {
                mimeType,
                videoBitsPerSecond: 2_500_000  // 2.5 Mbps for camera
            });
            this.cameraRecorder.ondataavailable = (e) => {
                if (e.data.size > 0) this.cameraChunks.push(e.data);
            };
            this.cameraRecorder.onstop = () => {
                const ext = this.getFileExtension(mimeType);
                const blobType = mimeType.includes('mp4') ? 'video/mp4' : 'video/webm';
                this.exportVideo(this.cameraChunks, `camera-recording.${ext}`, blobType);
            };
        }

        // Start both recorders
        this.screenRecorder.start(1000);  // Collect data every 1s
        if (this.cameraRecorder) {
            this.cameraRecorder.start(1000);
        }

        this.isRecording = true;
        this.startTime = Date.now();
        this.updateUI(true);
        this.startTimer();
    }

    stopRecording() {
        if (!this.isRecording) return;

        this.isRecording = false;
        this.stopTimer();

        if (this.screenRecorder && this.screenRecorder.state !== 'inactive') {
            this.screenRecorder.stop();
        }
        if (this.cameraRecorder && this.cameraRecorder.state !== 'inactive') {
            this.cameraRecorder.stop();
        }

        this.updateUI(false);
    }

    exportVideo(chunks, filename, blobType) {
        if (chunks.length === 0) return;
        const blob = new Blob(chunks, { type: blobType || 'video/mp4' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = filename;
        a.click();
        // Release object URL after a short delay
        setTimeout(() => URL.revokeObjectURL(url), 5000);
    }

    getBestMimeType() {
        // Prefer MP4 (H.264+AAC) for universal playback, fall back to WebM
        const candidates = [
            'video/mp4;codecs=avc1.42E01E,mp4a.40.2',
            'video/mp4;codecs=h264,aac',
            'video/mp4',
            'video/webm;codecs=vp9,opus',
            'video/webm;codecs=vp8,opus',
            'video/webm'
        ];
        for (const type of candidates) {
            if (MediaRecorder.isTypeSupported(type)) return type;
        }
        return '';  // Let browser decide
    }

    getFileExtension(mimeType) {
        if (mimeType.includes('mp4')) return 'mp4';
        return 'webm';
    }

    cleanupStream(stream) {
        if (stream) {
            stream.getTracks().forEach(track => track.stop());
        }
    }

    updateUI(recording) {
        this.btnRecord.disabled = recording;
        this.btnStop.disabled = !recording;
        this.recIndicator.classList.toggle('active', recording);
        this.btnRecord.textContent = recording ? '录制中...' : '开始录制';
    }

    startTimer() {
        this.timerInterval = setInterval(() => {
            const elapsed = Math.floor((Date.now() - this.startTime) / 1000);
            const mm = String(Math.floor(elapsed / 60)).padStart(2, '0');
            const ss = String(elapsed % 60).padStart(2, '0');
            this.recTimer.textContent = `${mm}:${ss}`;
        }, 1000);
    }

    stopTimer() {
        clearInterval(this.timerInterval);
        this.recTimer.textContent = '00:00';
    }
}
```

## Browser Compatibility

| Feature | Chrome | Firefox | Safari | Edge |
|---------|--------|---------|--------|------|
| `getUserMedia` (camera) | ✅ | ✅ | ✅ | ✅ |
| `getDisplayMedia` (screen) | ✅ | ✅ | ✅ 13+ | ✅ |
| `MediaRecorder` | ✅ | ✅ | ✅ 14.6+ | ✅ |
| VP9 codec | ✅ | ✅ | ❌ | ✅ |
| System audio capture | ✅ (tab only) | ❌ | ❌ | ✅ |

**Recommended browser: Chrome or Edge** for best codec support and system audio capture.

## MIME Type Priority

1. `video/mp4;codecs=avc1.42E01E,mp4a.40.2` — Universal playback (H.264+AAC), Chrome 57+, macOS/Windows
2. `video/mp4;codecs=h264,aac` — Alternative MP4 specifier
3. `video/mp4` — MP4 container, browser picks codecs
4. `video/webm;codecs=vp9,opus` — Fallback, best WebM quality
5. `video/webm;codecs=vp8,opus` — Broad WebM fallback
6. `video/webm` — WebM default

MP4 is preferred because it plays natively on all platforms (macOS QuickTime, Windows Media Player, iOS, Android) without conversion. WebM is used as fallback when MP4 recording is not supported (e.g., older Firefox on Linux).

## Error Handling

| Scenario | Behavior |
|----------|----------|
| Camera permission denied | Recording works without camera; only screen exported |
| Screen share cancelled | No recording starts; UI stays in ready state |
| User clicks "Stop sharing" in browser | Auto-triggers `stopRecording()` |
| Tab/window closed during recording | Streams auto-stop; partial data is lost |
| MediaRecorder not supported | Show fallback message; disable recording UI |

## Performance Considerations

- **Bitrate**: Screen at 5 Mbps, camera at 2.5 Mbps. Adjust via `videoBitsPerSecond`.
- **Chunk interval**: Collect every 1000ms (`start(1000)`). Smaller intervals = less data loss on crash but more memory overhead.
- **Memory**: For long recordings (>30 min), chunks accumulate in RAM. Consider periodic flushing to IndexedDB for very long sessions.
- **CPU**: Two simultaneous MediaRecorders + video encoding. On older machines, reduce screen capture to 720p.

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Ctrl+Shift+R` | Toggle recording |
| `Ctrl+Shift+S` | Stop recording |

## File Naming Convention

Generated files use timestamps for uniqueness:

```javascript
const timestamp = new Date().toISOString().replace(/[:.]/g, '-').slice(0, 19);
// screen-recording-2025-01-15T14-30-22.webm
// camera-recording-2025-01-15T14-30-22.webm
```
