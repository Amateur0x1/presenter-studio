---
name: presenter-studio
description: 生成可录制视频的 HTML 演示文稿。当用户需要做演示、录课、录制讲解视频，或者需要一个带录屏+录像功能的幻灯片时使用。
---

# Presenter Studio

基于 `template.html` 模板生成带视频录制能力的 HTML 演示文稿，单文件、零依赖、浏览器直接打开即用。

## 这个 Skill 做什么

把用户提供的内容（标题、大纲、文字）填入模板，生成一份完整的 HTML 演示文稿。生成的文件自带：

- 标签页录制（只录当前页面内容，不录浏览器 UI）
- 摄像头录制（后台录，不显示预览）
- 麦克风收音（音频存入摄像头文件）
- 双文件导出到用户选的文件夹（slides + camera）
- 可编辑文本块（双击编辑、拖拽移动）
- 键盘快捷键控制录制（Space 暂停/继续，Esc 停止）

## 工作流程

### Step 1: 收集信息

引导用户提供以下内容（一次性问完）：

1. **标题** — 演示文稿的主标题
2. **内容** — 每张幻灯片的要点（大纲、笔记、或完整文本都行）
3. **风格偏好**（可选） — 暗色/亮色、配色倾向、字体风格。不提供则默认暗色主题。
4. **用途**（可选） — 教学/演示录制/会议分享/视频创作。影响信息密度。

如果用户只给了主题没给具体内容，帮他生成合理的大纲内容。

### Step 2: 生成演示文稿

1. 读取模板文件：[template.html](template.html)
2. 基于模板结构，将用户内容填入幻灯片区域
3. 根据风格偏好调整 `:root` 中的 CSS 变量（配色、字体等）
4. 输出为一个完整的 `.html` 文件

**生成规则：**

- 保持模板中所有 JS 逻辑不变（SlidePresentation、TextBlockEditor、DualRecorder）
- 保持录制工具栏、键盘快捷键等交互代码不变
- 只修改：`<title>`、`:root` CSS 变量、`<main class="deck-stage">` 内的幻灯片内容
- 每张幻灯片是一个 `<section class="slide" data-slide="N">`，第一张加 `active` class
- 文本用 `.text-block` 包裹，用 `style="top: Ypx; left: Xpx;"` 定位（基于 1920×1080 坐标）
- 可参考 [docs/STYLE_PRESETS.md](docs/STYLE_PRESETS.md) 获取配色方案灵感

### Step 3: 交付

1. 保存生成的 HTML 文件
2. 启动本地服务器：`python3 -m http.server 8080`（录制功能需要 localhost 安全上下文）
3. 告诉用户：
   - 用浏览器打开 `http://localhost:8080/文件名.html`
   - 方向键翻页
   - 顶部工具栏点"开始"录制，录制中工具栏自动隐藏
   - Space 暂停/继续，Esc 停止并导出
   - 停止后选一个文件夹，会保存 slides 和 camera 两个视频文件

## 支撑文件

| 文件 | 用途 |
|------|------|
| [template.html](template.html) | 空白模板，包含完整的录制/编辑/导出能力 |
| [docs/STYLE_PRESETS.md](docs/STYLE_PRESETS.md) | 6 套配色预设供参考 |
| [docs/recording-module.md](docs/recording-module.md) | DualRecorder 实现参考 |
| [docs/html-template.md](docs/html-template.md) | HTML 结构参考 |
| [docs/viewport-base.css](docs/viewport-base.css) | 固定 16:9 舞台 CSS |
| [examples/demo.html](examples/demo.html) | 带示例内容的完整演示 |

## 注意事项

- 必须通过 localhost 访问（不能 file:// 打开），否则录制 API 不可用
- 推荐 Chrome/Edge 浏览器
- 视频格式优先 MP4，不支持则 WebM
- 码率：slides 8Mbps、camera 4Mbps
