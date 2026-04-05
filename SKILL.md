---
name: video-learning
description: 视频内容学习技能。从视频链接中提取音频、转写文字、分析内容、截图学习，最终输出文字+图片+总结的完整学习报告。
**前置依赖（需自行安装）** - `yt-dlp` + `ffmpeg` - 音频提取
- `whisper`（Python包）- 语音转文字
- `browser`工具 - 截图学习（OpenClaw内置）

当用户给一个视频链接要"学习"、"分析"、"提取内容"时自动触发。
---

# Video Learning

学习视频内容的完整流程。

## 前置依赖（必须自行安装）

| 依赖 | 安装命令 | 用途 |
|------|---------|------|
| yt-dlp | `pip install yt-dlp` | 从视频提取音频 |
| ffmpeg | [下载安装](https://ffmpeg.org/download.html) | 音频格式转换 |
| whisper | `pip install openai-whisper` | 语音转文字 |
| browser | OpenClaw 内置 | 截图学习（无需安装） |

**安装顺序建议**：先装ffmpeg，再装yt-dlp，最后装 whisper。

## 标准流程

| 步骤 | 操作 | 工具 | 输出 |
|------|------|------|------|
| 1 | 提取音频 | yt-dlp + ffmpeg | audio.wav |
| 2 | 转文字 | whisper | audio_text.txt |
| 3 | 文字学习 | AI分析 | 关键信息提取 |
| 4 | 截图学习 | browser | 关键时间点截图 |
| 5 | 综合反馈 | 合并输出 | 文字+截图+总结 |

## 步骤1：提取音频
```bash
yt-dlp -x --audio-format wav -o "audio.wav" "<video_url>"
```

或使用 ffmpeg 转换：
```bash
ffmpeg -i "<video_file>" audio.wav
```

## 步骤2：转文字

使用 whisper 转写音频为文字：

```bash
whisper audio.wav --model medium --language Chinese --output_format txt
```

输出文件：`audio_text.txt`

## 步骤3：文字学习

读取 audio_text.txt，通过 AI 分析提取：
- 核心主题
- 关键概念
- 需要截图验证的重要时间点（带上时间参数）

## 步骤4：截图学习

### 跳转前检查登录弹窗

**关键要求**：打开视频页面后，先执行 snapshot 检查是否有登录弹窗。
```
browser(action=snapshot)
```

- **如有登录弹窗**：点击"关闭"按钮后再继续
- **无弹窗**：直接继续

### 截图学习而非截图验证

通过截图来**学习**视频中的内容（界面、代码、图表等），不仅仅是简单截图。

### 操作方式

1. 根据步骤3确定的关键时间点，使用 browser 跳转到对应时间：
   ```
   browser(action=navigate, targetUrl="<video_url>&t=XXs")
   ```
2. 等待页面加载完成后执行 snapshot
3. 确认无登录弹窗后截图：
   ```
   browser(action=screenshot)
   ```

### 逐帧学习（关键部分）

对于特别重要的内容，可以使用键盘方向键逐帧控制：
- `ArrowRight`：前进一帧
- `ArrowLeft`：后退一帧
```
browser(action=act, kind=press, key="ArrowRight")
```

结合截图 until 找到最关键的那一帧进行学习。

## 步骤5：综合反馈

将分析结果整理为结构化输出：
- 核心收获（文字+截图）
- 关键时间点截图列表
- 完整总结

## 步骤6：自动清理（完成后必须执行）

**清理所有临时文件**：
| 文件类型 | 清理方式 |
|---------|---------|
| 音频文件 | `Remove-Item audio.wav -Force` |
| 转录文字 | `Remove-Item audio_text.txt -Force` |
| 学习截图 | `Remove-Item screenshot_*.png -Force` |
| 其他临时文件 | `Remove-Item temp_* -Force` |

**触发时机**：完成步骤5综合反馈后**立刻执行**，不等待用户确认。

**操作规范**：
1. 反馈输出后立即执行清理
2. 保留最终学习报告（文字+截图）到指定目录
3. 清理前确认文件存在再删除
4. 如果某个文件不存在，跳过该文件继续清理其他的

## 快速参考

```
视频URL → 提取音频 → whisper转文字 → 文字学习 → 截图学习(先检查登录弹窗+逐帧学习) → 综合输出 → 自动清理
```

**前置依赖**：yt-dlp + ffmpeg + whisper（browser由OpenClaw提供）
**逐帧学习**：关键部分用 ArrowRight/ArrowLeft 精确定位后截图
**清理规则**：反馈完成后立即删除所有临时文件，不保留转录音频、文字和中间截图。
