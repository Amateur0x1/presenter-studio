# Presenter Studio

可录制视频的 HTML 演示文稿工具。单文件、零依赖、浏览器打开即用。

## 功能

- **标签页录制** — 只录当前页面内容，不录浏览器标签栏和工具栏
- **摄像头录制** — 后台录制人脸，不在页面上显示预览
- **麦克风收音** — 你的讲解声音会录入摄像头视频文件
- **双文件导出** — 停止录制后选择文件夹，自动保存 `slides-xxx.mp4` + `camera-xxx.mp4`
- **可编辑文本** — 双击任意文本块编辑内容，拖拽调整位置
- **录制时 UI 隐藏** — 工具栏在录制过程中完全消失，不会被录进视频
- **高清录制** — slides 8Mbps、camera 4Mbps，文字清晰锐利

## 快速开始

```bash
# 1. 进入目录
cd ~/self/presenter-studio

# 2. 启动本地服务器（录制 API 需要 localhost 安全上下文）
python3 -m http.server 8080

# 3. 浏览器打开
open http://localhost:8080/examples/demo.html
```

> ⚠️ 不能直接用 `file://` 打开，否则录制功能不可用。

## 操作方式

| 操作 | 方式 |
|------|------|
| 翻页 | ← → 方向键，或触屏滑动 |
| 开始录制 | 点击顶部工具栏「● 开始」 |
| 暂停/继续 | 按 `Space` 键 |
| 停止录制 | 按 `Esc` 键 |
| 编辑文本 | 双击文本块 |
| 移动文本 | 拖拽文本块 |
| 退出编辑 | 按 `Esc` 或点击空白处 |

## 录制流程

1. 点击「开始」→ 浏览器依次请求麦克风、标签页捕获、摄像头权限
2. 授权后工具栏自动隐藏，开始录制
3. 正常翻页讲解，所有操作都会被录入 slides 视频
4. 按 `Space` 可暂停，再按继续
5. 按 `Esc` 停止 → 弹出文件夹选择器 → 两个视频文件保存到该文件夹

## 导出文件

| 文件 | 内容 |
|------|------|
| `slides-[时间戳].mp4` | 标签页画面 + 标签页系统音频 |
| `camera-[时间戳].mp4` | 摄像头画面 + 麦克风声音 |

格式优先 MP4（H.264+AAC），浏览器不支持时降级为 WebM。

## 文件说明

```
presenter-studio/
├── README.md                ← 你在看的这个
├── README.en.md             ← English version
├── SKILL.md                 ← AI Skill 定义，指导 AI 如何使用此工具
├── template.html            ← 空白模板，填入内容即可生成新演示
│
├── docs/                    ← 参考文档
│   ├── STYLE_PRESETS.md     ← 6 套配色预设
│   ├── html-template.md     ← HTML 结构参考
│   ├── recording-module.md  ← DualRecorder 实现参考
│   └── viewport-base.css   ← 固定 16:9 舞台 CSS
│
└── examples/                ← 示例
    └── demo.html            ← 带内容的完整演示（可直接体验）
```

## 自定义

修改 HTML 文件中 `:root` 的 CSS 变量即可快速换肤：

```css
:root {
    --bg-primary: #0f0f13;      /* 幻灯片背景 */
    --text-primary: #f0f0f2;    /* 主文字颜色 */
    --accent: #6366f1;          /* 强调色 */
    --font-display: 'Outfit';   /* 标题字体 */
    --font-body: 'Inter';       /* 正文字体 */
    --title-size: 96px;         /* 标题字号 */
}
```

## 创建新演示

最简单的方式：复制 `template.html`，在 `<main class="deck-stage">` 里添加幻灯片：

```html
<section class="slide active" data-slide="1">
    <div class="text-block" style="top: 300px; left: 140px;" contenteditable="false">
        <h1>你的标题</h1>
        <p style="margin-top: 20px;">你的内容</p>
    </div>
</section>

<section class="slide" data-slide="2">
    <div class="text-block" style="top: 200px; left: 140px;" contenteditable="false">
        <h2>第二页</h2>
        <p style="margin-top: 16px;">更多内容...</p>
    </div>
</section>
```

坐标基于 1920×1080，页面会自动缩放适配浏览器窗口。

## 浏览器兼容

| 浏览器 | 录制 | 文件夹导出 |
|--------|------|-----------|
| Chrome | ✅ | ✅ |
| Edge | ✅ | ✅ |
| Firefox | ✅（WebM） | ❌（降级为下载） |
| Safari | ❌ | ❌ |

推荐使用 Chrome 或 Edge 以获得完整体验。
