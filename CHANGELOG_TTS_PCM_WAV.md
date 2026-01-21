# 🎵 TTS PCM to WAV 转换功能更新

## 📝 更新内容

### 新增功能
1. **PCM到WAV自动转换**
   - 自动检测Gemini TTS API返回的L16 PCM格式音频
   - 实时转换为标准WAV格式,支持浏览器直接播放
   - 支持从mimeType自动解析采样率

2. **完整的WAV文件头生成**
   - RIFF chunk descriptor
   - fmt sub-chunk (PCM格式配置)
   - data sub-chunk (音频数据)
   - 支持16-bit, 单声道, 可变采样率

3. **测试工具**
   - 创建独立的PCM到WAV转换测试页面
   - 音频数据分析功能
   - 统计信息展示

### 修改的文件

#### `ui/app/pages/TTSPage.vue`
- 添加 `pcmToWav()` 函数 - PCM到WAV转换核心逻辑
- 添加 `writeString()` 辅助函数 - 写入WAV文件头字符串
- 修改 `generateSpeech()` - 自动检测并转换PCM格式
- 音频URL生成改用 `URL.createObjectURL()` 以支持Blob对象

#### `test_pcm_to_wav.html` (新增)
- 独立的PCM到WAV转换测试工具
- 音频数据分析功能
- 可视化统计信息展示

#### 其他更新
- `ui/app/pages/ChatPage.vue` - 模型配置更新
- `ui/app/pages/ImagePage.vue` - 模型配置更新
- `ui/app/pages/StatusPage.vue` - 界面优化
- `ui/app/router/index.js` - 路由配置
- `ui/locales/zh.json` - 中文翻译
- `ui/locales/en.json` - 英文翻译

## 🔧 技术细节

### WAV文件格式
```
[RIFF Header - 12 bytes]
  - "RIFF" (4 bytes)
  - File size - 8 (4 bytes)
  - "WAVE" (4 bytes)

[fmt Chunk - 24 bytes]
  - "fmt " (4 bytes)
  - Chunk size: 16 (4 bytes)
  - Audio format: 1 (PCM) (2 bytes)
  - Channels: 1 (2 bytes)
  - Sample rate: 24000 (4 bytes)
  - Byte rate (4 bytes)
  - Block align (2 bytes)
  - Bits per sample: 16 (2 bytes)

[data Chunk - 8 bytes + audio data]
  - "data" (4 bytes)
  - Data size (4 bytes)
  - Audio data (variable)
```

### 转换流程
1. Base64解码PCM数据
2. 解析采样率 (从mimeType)
3. 计算WAV参数 (byte rate, block align)
4. 创建44字节WAV文件头
5. 合并头部和PCM数据
6. 生成Blob对象
7. 创建Object URL供<audio>标签使用

## 🎯 使用方法

### 正常使用
1. 在TTS页面输入文本
2. 点击"生成语音"
3. 系统自动检测PCM格式并转换为WAV
4. 音频自动加载到播放器
5. 可直接播放、调整速度、音量、下载

### 测试转换功能
```bash
# 在浏览器中打开测试页面
start test_pcm_to_wav.html
```

## 📊 兼容性

- ✅ Chrome/Edge (推荐)
- ✅ Firefox
- ✅ Safari
- ✅ 所有支持Web Audio API的现代浏览器

## 🐛 问题修复

- 修复了Gemini TTS返回PCM格式无法播放的问题
- 修复了音频下载格式不正确的问题
- 优化了音频加载和播放体验

## 📅 更新日期

2026-01-21

## 👨‍💻 开发者

AI Assistant
