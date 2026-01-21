<!--
 * File: ui/app/pages/TTSPage.vue
 * Description: Text-to-Speech interface using Gemini TTS models
 *
 * Maintainers: AI Assistant
-->

<template>
    <div class="tts-page">
        <div class="tts-container">
            <!-- Header -->
            <header class="page-header">
                <div class="header-left">
                    <router-link to="/" class="back-btn" :title="t('backToStatus')">
                        <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <path d="m15 18-6-6 6-6"/>
                        </svg>
                    </router-link>
                    <h1>{{ t('ttsTitle') }}</h1>
                </div>
                <div class="header-right">
                    <el-select v-model="selectedModel" :placeholder="t('selectModel')" class="model-select">
                        <el-option
                            v-for="model in ttsModels"
                            :key="model.id"
                            :label="model.name"
                            :value="model.id"
                        />
                    </el-select>
                </div>
            </header>

            <!-- Main Content -->
            <div class="main-content">
                <!-- Input Section -->
                <div class="input-section">
                    <div class="text-card">
                        <h3>{{ t('ttsInputTitle') }}</h3>
                        <textarea
                            v-model="inputText"
                            :placeholder="t('ttsInputPlaceholder')"
                            rows="8"
                            :disabled="isGenerating"
                            :maxlength="5000"
                        ></textarea>
                        <div class="char-count">{{ inputText.length }} / 5000</div>
                        
                        <div class="options-row">
                            <div class="option-group">
                                <label>{{ t('ttsVoice') }}</label>
                                <el-select v-model="selectedVoice" size="small" class="voice-select">
                                    <el-option
                                        v-for="voice in voices"
                                        :key="voice.id"
                                        :label="voice.name"
                                        :value="voice.id"
                                    />
                                </el-select>
                            </div>
                        </div>
                        
                        <div class="actions-row">
                            <button 
                                class="generate-btn" 
                                @click="generateSpeech"
                                :disabled="!inputText.trim() || isGenerating"
                            >
                                <svg v-if="!isGenerating" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                                    <polygon points="5 3 19 12 5 21 5 3"/>
                                </svg>
                                <span v-if="isGenerating" class="loading-spinner"></span>
                                <span>{{ isGenerating ? t('generating') : t('generateSpeech') }}</span>
                            </button>
                            <button class="clear-btn" @click="clearText" :disabled="isGenerating">
                                {{ t('clearText') }}
                            </button>
                        </div>
                    </div>
                    
                    <!-- Quick Phrases -->
                    <div class="phrases-card">
                        <h4>{{ t('quickPhrases') }}</h4>
                        <div class="phrases-list">
                            <button 
                                v-for="phrase in quickPhrases" 
                                :key="phrase"
                                class="phrase-btn"
                                @click="inputText = phrase"
                            >
                                {{ phrase }}
                            </button>
                        </div>
                    </div>
                </div>

                <!-- Results Section -->
                <div class="results-section">
                    <div v-if="audioList.length === 0 && !isGenerating" class="empty-state">
                        <div class="empty-icon">
                            <svg xmlns="http://www.w3.org/2000/svg" width="80" height="80" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1" stroke-linecap="round" stroke-linejoin="round">
                                <path d="M12 2a3 3 0 0 0-3 3v7a3 3 0 0 0 6 0V5a3 3 0 0 0-3-3Z"/>
                                <path d="M19 10v2a7 7 0 0 1-14 0v-2"/>
                                <line x1="12" x2="12" y1="19" y2="22"/>
                            </svg>
                        </div>
                        <h2>{{ t('ttsWelcome') }}</h2>
                        <p>{{ t('ttsWelcomeDesc') }}</p>
                    </div>

                    <div v-if="isGenerating" class="generating-state">
                        <div class="sound-wave">
                            <span></span>
                            <span></span>
                            <span></span>
                            <span></span>
                            <span></span>
                        </div>
                        <p>{{ t('ttsGenerating') }}</p>
                    </div>

                    <div v-if="audioList.length > 0" class="audio-list">
                        <div 
                            v-for="(audio, index) in audioList" 
                            :key="index" 
                            class="audio-card"
                        >
                            <div class="audio-info">
                                <div class="audio-icon">
                                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                                        <path d="M9 18V5l12-2v13"/>
                                        <circle cx="6" cy="18" r="3"/>
                                        <circle cx="18" cy="16" r="3"/>
                                    </svg>
                                </div>
                                <div class="audio-details">
                                    <div class="audio-text">{{ audio.text.substring(0, 100) }}{{ audio.text.length > 100 ? '...' : '' }}</div>
                                    <div class="audio-meta">
                                        <span>{{ audio.voice }}</span>
                                        <span>{{ formatTime(audio.timestamp) }}</span>
                                    </div>
                                </div>
                            </div>
                            <div class="audio-player">
                                <audio 
                                    :ref="el => setAudioRef(el, index)"
                                    :src="audio.url" 
                                    @play="onAudioPlay(index)"
                                    @pause="onAudioPause(index)"
                                    @ended="onAudioEnded(index)"
                                    @timeupdate="onTimeUpdate(index)"
                                    @loadedmetadata="onLoadedMetadata(index)"
                                    @error="onAudioError(index, $event)"
                                ></audio>
                                
                                <!-- 播放控制按钮 -->
                                <div class="player-controls">
                                    <button 
                                        class="play-btn" 
                                        @click="togglePlay(index)"
                                        :class="{ playing: playingIndex === index }"
                                        :title="playingIndex === index ? '暂停' : '播放'"
                                    >
                                        <svg v-if="playingIndex !== index" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                                            <polygon points="5 3 19 12 5 21 5 3"/>
                                        </svg>
                                        <svg v-else xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                                            <rect x="6" y="4" width="4" height="16"/>
                                            <rect x="14" y="4" width="4" height="16"/>
                                        </svg>
                                    </button>
                                    
                                    <!-- 时间显示 -->
                                    <div class="time-display">
                                        <span class="current-time">{{ formatDuration(audio.currentTime || 0) }}</span>
                                        <span class="separator">/</span>
                                        <span class="total-time">{{ formatDuration(audio.duration || 0) }}</span>
                                    </div>
                                </div>
                                
                                <!-- 进度条 -->
                                <div class="progress-container">
                                    <div 
                                        class="progress-bar" 
                                        @click="seekAudio(index, $event)"
                                        @mousedown="startDragging(index)"
                                    >
                                        <div 
                                            class="progress-fill" 
                                            :style="{ width: getProgress(index) + '%' }"
                                        >
                                            <div class="progress-handle"></div>
                                        </div>
                                    </div>
                                </div>
                                
                                <!-- 音量和其他控制 -->
                                <div class="extra-controls">
                                    <!-- 音量控制 -->
                                    <div class="volume-control">
                                        <button 
                                            class="volume-btn" 
                                            @click="toggleMute(index)"
                                            :title="audio.muted ? '取消静音' : '静音'"
                                        >
                                            <svg v-if="!audio.muted && (audio.volume || 1) > 0.5" xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                                                <polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"/>
                                                <path d="M15.54 8.46a5 5 0 0 1 0 7.07"/>
                                                <path d="M19.07 4.93a10 10 0 0 1 0 14.14"/>
                                            </svg>
                                            <svg v-else-if="!audio.muted && (audio.volume || 1) > 0" xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                                                <polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"/>
                                                <path d="M15.54 8.46a5 5 0 0 1 0 7.07"/>
                                            </svg>
                                            <svg v-else xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                                                <polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"/>
                                                <line x1="23" y1="9" x2="17" y2="15"/>
                                                <line x1="17" y1="9" x2="23" y2="15"/>
                                            </svg>
                                        </button>
                                        <div class="volume-slider-container">
                                            <input 
                                                type="range" 
                                                class="volume-slider" 
                                                min="0" 
                                                max="100" 
                                                :value="(audio.volume || 1) * 100"
                                                @input="changeVolume(index, $event)"
                                            />
                                        </div>
                                    </div>
                                    
                                    <!-- 播放速度 -->
                                    <div class="speed-control">
                                        <select 
                                            class="speed-select" 
                                            :value="audio.playbackRate || 1"
                                            @change="changeSpeed(index, $event)"
                                        >
                                            <option value="0.5">0.5x</option>
                                            <option value="0.75">0.75x</option>
                                            <option value="1">1x</option>
                                            <option value="1.25">1.25x</option>
                                            <option value="1.5">1.5x</option>
                                            <option value="2">2x</option>
                                        </select>
                                    </div>
                                    
                                    <!-- 下载按钮 -->
                                    <button class="download-btn" @click="downloadAudio(audio)" :title="t('download')">
                                        <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                                            <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/>
                                            <polyline points="7 10 12 15 17 10"/>
                                            <line x1="12" x2="12" y1="15" y2="3"/>
                                        </svg>
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Navigation -->
            <nav class="bottom-nav">
                <router-link to="/chat" class="nav-item">
                    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/></svg>
                    <span>{{ t('navChat') }}</span>
                </router-link>
                <router-link to="/image" class="nav-item">
                    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect width="18" height="18" x="3" y="3" rx="2" ry="2"/><circle cx="9" cy="9" r="2"/><path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/></svg>
                    <span>{{ t('navImage') }}</span>
                </router-link>
                <router-link to="/tts" class="nav-item active">
                    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 2a3 3 0 0 0-3 3v7a3 3 0 0 0 6 0V5a3 3 0 0 0-3-3Z"/><path d="M19 10v2a7 7 0 0 1-14 0v-2"/><line x1="12" x2="12" y1="19" y2="22"/></svg>
                    <span>{{ t('navTTS') }}</span>
                </router-link>
            </nav>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { ElMessage } from 'element-plus';
import I18n from '../utils/i18n';

const t = (key) => I18n.t(key);

const inputText = ref('');
const isGenerating = ref(false);
const audioList = ref([]);
const selectedModel = ref('gemini-2.5-flash-preview-tts');
const selectedVoice = ref('Zephyr');
const playingIndex = ref(-1);
const audioRefs = ref({});
const isDragging = ref(false);
const draggingIndex = ref(-1);

const ttsModels = ref([
    { id: 'gemini-2.5-flash-preview-tts', name: 'Gemini 2.5 Flash TTS' },
    { id: 'gemini-2.5-pro-preview-tts', name: 'Gemini 2.5 Pro TTS' },
]);

const voices = ref([
    { id: 'Zephyr', name: 'Zephyr (温柔女声)' },
    { id: 'Puck', name: 'Puck (活泼男声)' },
    { id: 'Charon', name: 'Charon (低沉男声)' },
    { id: 'Kore', name: 'Kore (甜美女声)' },
    { id: 'Fenrir', name: 'Fenrir (成熟男声)' },
    { id: 'Aoede', name: 'Aoede (优雅女声)' },
]);

const quickPhrases = ref([
    '你好,欢迎使用 AI 语音合成系统!',
    'Hello, welcome to the AI text-to-speech system!',
    '今天天气真好,适合出去走走。',
    '科技改变生活,AI 创造未来。',
]);

const setAudioRef = (el, index) => {
    if (el) {
        audioRefs.value[index] = el;
    }
};

const formatTime = (timestamp) => {
    return new Date(timestamp).toLocaleTimeString();
};

// 格式化时长 (秒 -> MM:SS)
const formatDuration = (seconds) => {
    if (!seconds || isNaN(seconds)) return '00:00';
    const mins = Math.floor(seconds / 60);
    const secs = Math.floor(seconds % 60);
    return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
};

// PCM转WAV格式
const pcmToWav = (base64PcmData, sampleRate = 24000) => {
    try {
        // 解码Base64
        const binaryString = atob(base64PcmData);
        const len = binaryString.length;
        const bytes = new Uint8Array(len);
        for (let i = 0; i < len; i++) {
            bytes[i] = binaryString.charCodeAt(i);
        }
        
        // PCM参数
        const numChannels = 1; // 单声道
        const bitsPerSample = 16; // 16位
        const byteRate = sampleRate * numChannels * bitsPerSample / 8;
        const blockAlign = numChannels * bitsPerSample / 8;
        const dataSize = bytes.length;
        
        // 创建WAV文件头 (44字节)
        const wavHeader = new ArrayBuffer(44);
        const view = new DataView(wavHeader);
        
        // RIFF chunk descriptor
        writeString(view, 0, 'RIFF');
        view.setUint32(4, 36 + dataSize, true); // 文件大小 - 8
        writeString(view, 8, 'WAVE');
        
        // fmt sub-chunk
        writeString(view, 12, 'fmt ');
        view.setUint32(16, 16, true); // fmt chunk size
        view.setUint16(20, 1, true); // audio format (1 = PCM)
        view.setUint16(22, numChannels, true); // number of channels
        view.setUint32(24, sampleRate, true); // sample rate
        view.setUint32(28, byteRate, true); // byte rate
        view.setUint16(32, blockAlign, true); // block align
        view.setUint16(34, bitsPerSample, true); // bits per sample
        
        // data sub-chunk
        writeString(view, 36, 'data');
        view.setUint32(40, dataSize, true); // data size
        
        // 合并头部和数据
        const wavData = new Uint8Array(44 + dataSize);
        wavData.set(new Uint8Array(wavHeader), 0);
        wavData.set(bytes, 44);
        
        // 创建Blob
        return new Blob([wavData], { type: 'audio/wav' });
    } catch (error) {
        console.error('PCM to WAV conversion error:', error);
        throw error;
    }
};

// 辅助函数:写入字符串到DataView
const writeString = (view, offset, string) => {
    for (let i = 0; i < string.length; i++) {
        view.setUint8(offset + i, string.charCodeAt(i));
    }
};


// 生成语音
const generateSpeech = async () => {
    if (!inputText.value.trim() || isGenerating.value) return;
    
    isGenerating.value = true;
    
    try {
        const response = await fetch(`/v1beta/models/${selectedModel.value}:generateContent`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': 'Bearer 123456'
            },
            body: JSON.stringify({
                contents: [
                    {
                        parts: [
                            { text: inputText.value }
                        ]
                    }
                ],
                generationConfig: {
                    responseModalities: ["AUDIO"],
                    speechConfig: {
                        voiceConfig: {
                            prebuiltVoiceConfig: {
                                voiceName: selectedVoice.value
                            }
                        }
                    }
                }
            })
        });
        
        if (!response.ok) {
            const errorText = await response.text();
            throw new Error(`HTTP ${response.status}: ${errorText}`);
        }
        
        const data = await response.json();
        console.log('TTS Response:', data);
        
        // Extract audio from response
        const audioPart = data.candidates?.[0]?.content?.parts?.find(p => p.inlineData?.mimeType?.startsWith('audio/'));
        
        if (audioPart) {
            let audioUrl;
            
            // 检查是否是PCM格式,需要转换为WAV
            if (audioPart.inlineData.mimeType.includes('L16') || audioPart.inlineData.mimeType.includes('pcm')) {
                console.log('Converting PCM to WAV...');
                
                // 解析采样率
                const sampleRateMatch = audioPart.inlineData.mimeType.match(/rate=(\d+)/);
                const sampleRate = sampleRateMatch ? parseInt(sampleRateMatch[1]) : 24000;
                
                // 转换PCM为WAV
                const wavBlob = pcmToWav(audioPart.inlineData.data, sampleRate);
                audioUrl = URL.createObjectURL(wavBlob);
                
                console.log('PCM converted to WAV successfully');
            } else {
                // 其他格式直接使用
                audioUrl = `data:${audioPart.inlineData.mimeType};base64,${audioPart.inlineData.data}`;
            }
            
            // 添加音频到列表,包含状态信息
            audioList.value.unshift({
                url: audioUrl,
                text: inputText.value,
                voice: selectedVoice.value,
                timestamp: new Date().toISOString(),
                currentTime: 0,
                duration: 0,
                volume: 1,
                muted: false,
                playbackRate: 1
            });
            
            ElMessage.success(t('ttsGenSuccess') || '语音生成成功!');
            
            // 清空输入框
            // inputText.value = '';
        } else {
            console.error('No audio in response:', data);
            ElMessage.warning(t('ttsNoAudioInResponse') || '响应中没有音频数据');
        }
    } catch (error) {
        console.error('TTS Generation Error:', error);
        ElMessage.error((t('ttsGenError') || '生成失败') + ': ' + error.message);
    } finally {
        isGenerating.value = false;
    }
};

const clearText = () => {
    inputText.value = '';
};

// 播放/暂停切换
const togglePlay = (index) => {
    const audio = audioRefs.value[index];
    if (!audio) {
        console.error('Audio element not found for index:', index);
        return;
    }
    
    // 暂停其他正在播放的音频
    if (playingIndex.value !== -1 && playingIndex.value !== index) {
        const prevAudio = audioRefs.value[playingIndex.value];
        if (prevAudio) {
            prevAudio.pause();
        }
    }
    
    if (audio.paused) {
        audio.play().catch(err => {
            console.error('Play error:', err);
            ElMessage.error('播放失败: ' + err.message);
        });
    } else {
        audio.pause();
    }
};

// 音频播放事件
const onAudioPlay = (index) => {
    playingIndex.value = index;
};

const onAudioPause = (index) => {
    if (playingIndex.value === index) {
        playingIndex.value = -1;
    }
};

const onAudioEnded = (index) => {
    if (playingIndex.value === index) {
        playingIndex.value = -1;
    }
    // 重置进度
    if (audioList.value[index]) {
        audioList.value[index].currentTime = 0;
    }
};

// 时间更新
const onTimeUpdate = (index) => {
    const audio = audioRefs.value[index];
    if (audio && audioList.value[index]) {
        audioList.value[index].currentTime = audio.currentTime;
    }
};

// 元数据加载完成
const onLoadedMetadata = (index) => {
    const audio = audioRefs.value[index];
    if (audio && audioList.value[index]) {
        audioList.value[index].duration = audio.duration;
        console.log(`Audio ${index} loaded: ${audio.duration}s`);
    }
};

// 音频错误处理
const onAudioError = (index, event) => {
    console.error('Audio error:', event);
    const audio = audioRefs.value[index];
    if (audio && audio.error) {
        console.error('Audio error details:', {
            code: audio.error.code,
            message: audio.error.message
        });
        ElMessage.error(`音频加载失败 (错误码: ${audio.error.code})`);
    }
};

// 获取播放进度百分比
const getProgress = (index) => {
    const audio = audioList.value[index];
    if (!audio || !audio.duration) return 0;
    return (audio.currentTime / audio.duration) * 100;
};

// 拖动进度条
const seekAudio = (index, event) => {
    const audio = audioRefs.value[index];
    if (!audio || !audioList.value[index]?.duration) return;
    
    const progressBar = event.currentTarget;
    const rect = progressBar.getBoundingClientRect();
    const percent = (event.clientX - rect.left) / rect.width;
    const newTime = percent * audioList.value[index].duration;
    
    audio.currentTime = newTime;
    audioList.value[index].currentTime = newTime;
};

// 开始拖动
const startDragging = (index) => {
    isDragging.value = true;
    draggingIndex.value = index;
    
    const handleMouseMove = (e) => {
        if (isDragging.value && draggingIndex.value === index) {
            const progressBar = e.target.closest('.progress-bar');
            if (progressBar) {
                seekAudio(index, e);
            }
        }
    };
    
    const handleMouseUp = () => {
        isDragging.value = false;
        draggingIndex.value = -1;
        document.removeEventListener('mousemove', handleMouseMove);
        document.removeEventListener('mouseup', handleMouseUp);
    };
    
    document.addEventListener('mousemove', handleMouseMove);
    document.addEventListener('mouseup', handleMouseUp);
};

// 音量控制
const changeVolume = (index, event) => {
    const audio = audioRefs.value[index];
    const volume = event.target.value / 100;
    
    if (audio) {
        audio.volume = volume;
        if (audioList.value[index]) {
            audioList.value[index].volume = volume;
            if (volume > 0) {
                audioList.value[index].muted = false;
                audio.muted = false;
            }
        }
    }
};

// 静音切换
const toggleMute = (index) => {
    const audio = audioRefs.value[index];
    if (audio && audioList.value[index]) {
        audio.muted = !audio.muted;
        audioList.value[index].muted = audio.muted;
    }
};

// 播放速度控制
const changeSpeed = (index, event) => {
    const audio = audioRefs.value[index];
    const speed = parseFloat(event.target.value);
    
    if (audio) {
        audio.playbackRate = speed;
        if (audioList.value[index]) {
            audioList.value[index].playbackRate = speed;
        }
    }
};

// 下载音频
const downloadAudio = (audio) => {
    try {
        const link = document.createElement('a');
        link.href = audio.url;
        link.download = `tts-${audio.voice}-${Date.now()}.wav`;
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        ElMessage.success('开始下载音频');
    } catch (error) {
        console.error('Download error:', error);
        ElMessage.error('下载失败: ' + error.message);
    }
};

// 清理
onUnmounted(() => {
    // 停止所有音频
    Object.values(audioRefs.value).forEach(audio => {
        if (audio) {
            audio.pause();
        }
    });
});
</script>

<style lang="less" scoped>
@import '../styles/variables.less';

.tts-page {
    min-height: 100vh;
    background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
    display: flex;
    flex-direction: column;
}

.tts-container {
    flex: 1;
    display: flex;
    flex-direction: column;
    max-width: 1200px;
    margin: 0 auto;
    width: 100%;
    background: rgba(255, 255, 255, 0.95);
    box-shadow: 0 0 40px rgba(0, 0, 0, 0.1);
}

.page-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 16px 20px;
    background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
    color: white;
    
    .header-left {
        display: flex;
        align-items: center;
        gap: 12px;
        
        .back-btn {
            display: flex;
            align-items: center;
            justify-content: center;
            width: 36px;
            height: 36px;
            border-radius: 50%;
            background: rgba(255, 255, 255, 0.2);
            color: white;
            transition: background 0.2s;
            
            &:hover {
                background: rgba(255, 255, 255, 0.3);
            }
        }
        
        h1 {
            margin: 0;
            font-size: 1.3em;
            font-weight: 600;
        }
    }
    
    .header-right {
        .model-select {
            width: 220px;
            
            :deep(.el-input__wrapper) {
                background: rgba(255, 255, 255, 0.2);
                border: none;
                box-shadow: none;
                
                .el-input__inner {
                    color: white;
                }
            }
        }
    }
}

.main-content {
    flex: 1;
    display: grid;
    grid-template-columns: 400px 1fr;
    gap: 0;
    overflow: hidden;
    
    @media (max-width: 900px) {
        grid-template-columns: 1fr;
    }
}

.input-section {
    padding: 20px;
    background: #f8f9fa;
    border-right: 1px solid #eee;
    overflow-y: auto;
    
    .text-card {
        background: white;
        border-radius: 12px;
        padding: 20px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
        margin-bottom: 20px;
        
        h3 {
            margin: 0 0 12px;
            font-size: 1em;
            color: #333;
        }
        
        textarea {
            width: 100%;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 12px;
            font-size: 0.95em;
            resize: vertical;
            min-height: 150px;
            font-family: inherit;
            
            &:focus {
                outline: none;
                border-color: #4facfe;
            }
        }
        
        .char-count {
            text-align: right;
            font-size: 0.8em;
            color: #999;
            margin-top: 4px;
        }
        
        .options-row {
            display: flex;
            gap: 16px;
            margin-top: 16px;
            
            .option-group {
                flex: 1;
                
                label {
                    display: block;
                    font-size: 0.85em;
                    color: #666;
                    margin-bottom: 6px;
                }
                
                .voice-select {
                    width: 100%;
                }
            }
        }
        
        .actions-row {
            display: flex;
            gap: 10px;
            margin-top: 20px;
            
            .generate-btn {
                flex: 1;
                display: flex;
                align-items: center;
                justify-content: center;
                gap: 8px;
                padding: 14px 20px;
                background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
                color: white;
                border: none;
                border-radius: 8px;
                font-size: 1em;
                font-weight: 500;
                cursor: pointer;
                transition: transform 0.2s, box-shadow 0.2s;
                
                &:hover:not(:disabled) {
                    transform: translateY(-2px);
                    box-shadow: 0 4px 12px rgba(79, 172, 254, 0.4);
                }
                
                &:disabled {
                    opacity: 0.6;
                    cursor: not-allowed;
                }
                
                .loading-spinner {
                    width: 18px;
                    height: 18px;
                    border: 2px solid rgba(255, 255, 255, 0.3);
                    border-top-color: white;
                    border-radius: 50%;
                    animation: spin 0.8s linear infinite;
                }
            }
            
            .clear-btn {
                padding: 14px 20px;
                background: #f0f2f5;
                color: #666;
                border: none;
                border-radius: 8px;
                cursor: pointer;
                transition: background 0.2s;
                
                &:hover:not(:disabled) {
                    background: #e0e2e5;
                }
                
                &:disabled {
                    opacity: 0.5;
                }
            }
        }
    }
    
    .phrases-card {
        background: white;
        border-radius: 12px;
        padding: 16px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
        
        h4 {
            margin: 0 0 12px;
            font-size: 0.9em;
            color: #666;
        }
        
        .phrases-list {
            display: flex;
            flex-direction: column;
            gap: 8px;
            
            .phrase-btn {
                text-align: left;
                padding: 10px 12px;
                background: #f8f9fa;
                border: 1px solid #eee;
                border-radius: 8px;
                font-size: 0.85em;
                color: #555;
                cursor: pointer;
                transition: all 0.2s;
                
                &:hover {
                    background: #4facfe;
                    color: white;
                    border-color: #4facfe;
                }
            }
        }
    }
}

.results-section {
    flex: 1;
    padding: 20px;
    overflow-y: auto;
    
    .empty-state, .generating-state {
        height: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        color: #888;
        
        .empty-icon {
            color: #4facfe;
            opacity: 0.3;
            margin-bottom: 20px;
        }
        
        h2 {
            margin: 0 0 10px;
            color: #333;
        }
        
        p {
            margin: 0;
        }
    }
    
    .generating-state {
        .sound-wave {
            display: flex;
            align-items: center;
            gap: 4px;
            height: 60px;
            margin-bottom: 20px;
            
            span {
                width: 6px;
                background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
                border-radius: 3px;
                animation: wave 1s ease-in-out infinite;
                
                &:nth-child(1) { height: 20px; animation-delay: 0s; }
                &:nth-child(2) { height: 35px; animation-delay: 0.1s; }
                &:nth-child(3) { height: 50px; animation-delay: 0.2s; }
                &:nth-child(4) { height: 35px; animation-delay: 0.3s; }
                &:nth-child(5) { height: 20px; animation-delay: 0.4s; }
            }
        }
    }
    
    .audio-list {
        display: flex;
        flex-direction: column;
        gap: 16px;
        
        .audio-card {
            display: flex;
            align-items: center;
            justify-content: space-between;
            background: white;
            border-radius: 12px;
            padding: 16px 20px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
            transition: box-shadow 0.2s;
            
            &:hover {
                box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
            }
            
            .audio-info {
                display: flex;
                align-items: center;
                gap: 16px;
                flex: 1;
                min-width: 0;
                
                .audio-icon {
                    width: 48px;
                    height: 48px;
                    border-radius: 50%;
                    background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    color: white;
                    flex-shrink: 0;
                }
                
                .audio-details {
                    flex: 1;
                    min-width: 0;
                    
                    .audio-text {
                        font-size: 0.95em;
                        color: #333;
                        white-space: nowrap;
                        overflow: hidden;
                        text-overflow: ellipsis;
                    }
                    
                    .audio-meta {
                        display: flex;
                        gap: 12px;
                        margin-top: 4px;
                        font-size: 0.8em;
                        color: #888;
                    }
                }
            }
            
            .audio-player {
                display: flex;
                flex-direction: column;
                gap: 12px;
                padding: 16px;
                background: #f8f9fa;
                border-radius: 12px;
                margin-top: 12px;
                
                audio {
                    display: none;
                }
                
                .player-controls {
                    display: flex;
                    align-items: center;
                    gap: 12px;
                    
                    .play-btn {
                        width: 44px;
                        height: 44px;
                        border-radius: 50%;
                        border: none;
                        background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
                        color: white;
                        cursor: pointer;
                        display: flex;
                        align-items: center;
                        justify-content: center;
                        transition: transform 0.2s, box-shadow 0.2s;
                        flex-shrink: 0;
                        
                        &:hover {
                            transform: scale(1.05);
                            box-shadow: 0 4px 12px rgba(79, 172, 254, 0.4);
                        }
                        
                        &.playing {
                            animation: pulse-play 1.5s ease-in-out infinite;
                        }
                    }
                    
                    .time-display {
                        display: flex;
                        align-items: center;
                        gap: 6px;
                        font-size: 0.85em;
                        color: #666;
                        font-family: 'Courier New', monospace;
                        
                        .separator {
                            color: #999;
                        }
                    }
                }
                
                .progress-container {
                    width: 100%;
                    
                    .progress-bar {
                        width: 100%;
                        height: 6px;
                        background: #ddd;
                        border-radius: 3px;
                        cursor: pointer;
                        position: relative;
                        overflow: hidden;
                        
                        &:hover {
                            .progress-handle {
                                opacity: 1;
                            }
                        }
                        
                        .progress-fill {
                            height: 100%;
                            background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
                            border-radius: 3px;
                            position: relative;
                            transition: width 0.1s linear;
                            
                            .progress-handle {
                                position: absolute;
                                right: -6px;
                                top: 50%;
                                transform: translateY(-50%);
                                width: 14px;
                                height: 14px;
                                background: white;
                                border: 2px solid #4facfe;
                                border-radius: 50%;
                                opacity: 0;
                                transition: opacity 0.2s;
                                box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
                            }
                        }
                    }
                }
                
                .extra-controls {
                    display: flex;
                    align-items: center;
                    justify-content: space-between;
                    gap: 12px;
                    
                    .volume-control {
                        display: flex;
                        align-items: center;
                        gap: 8px;
                        flex: 1;
                        max-width: 200px;
                        
                        .volume-btn {
                            width: 32px;
                            height: 32px;
                            border-radius: 50%;
                            border: 1px solid #ddd;
                            background: white;
                            color: #666;
                            cursor: pointer;
                            display: flex;
                            align-items: center;
                            justify-content: center;
                            transition: all 0.2s;
                            flex-shrink: 0;
                            
                            &:hover {
                                background: #f5f7fa;
                                border-color: #4facfe;
                                color: #4facfe;
                            }
                        }
                        
                        .volume-slider-container {
                            flex: 1;
                            
                            .volume-slider {
                                width: 100%;
                                height: 4px;
                                -webkit-appearance: none;
                                appearance: none;
                                background: #ddd;
                                border-radius: 2px;
                                outline: none;
                                cursor: pointer;
                                
                                &::-webkit-slider-thumb {
                                    -webkit-appearance: none;
                                    appearance: none;
                                    width: 12px;
                                    height: 12px;
                                    background: #4facfe;
                                    border-radius: 50%;
                                    cursor: pointer;
                                    transition: transform 0.2s;
                                    
                                    &:hover {
                                        transform: scale(1.2);
                                    }
                                }
                                
                                &::-moz-range-thumb {
                                    width: 12px;
                                    height: 12px;
                                    background: #4facfe;
                                    border-radius: 50%;
                                    cursor: pointer;
                                    border: none;
                                    transition: transform 0.2s;
                                    
                                    &:hover {
                                        transform: scale(1.2);
                                    }
                                }
                            }
                        }
                    }
                    
                    .speed-control {
                        .speed-select {
                            padding: 6px 10px;
                            border: 1px solid #ddd;
                            border-radius: 6px;
                            background: white;
                            color: #666;
                            font-size: 0.85em;
                            cursor: pointer;
                            transition: all 0.2s;
                            
                            &:hover {
                                border-color: #4facfe;
                            }
                            
                            &:focus {
                                outline: none;
                                border-color: #4facfe;
                            }
                        }
                    }
                    
                    .download-btn {
                        width: 36px;
                        height: 36px;
                        border-radius: 50%;
                        border: 1px solid #ddd;
                        background: white;
                        color: #666;
                        cursor: pointer;
                        display: flex;
                        align-items: center;
                        justify-content: center;
                        transition: all 0.2s;
                        flex-shrink: 0;
                        
                        &:hover {
                            background: #f5f7fa;
                            border-color: #4facfe;
                            color: #4facfe;
                        }
                    }
                }
            }
        }
    }
}

.bottom-nav {
    display: flex;
    justify-content: center;
    gap: 40px;
    padding: 12px 20px;
    background: white;
    border-top: 1px solid #eee;
    
    .nav-item {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 4px;
        color: #888;
        text-decoration: none;
        font-size: 0.8em;
        transition: color 0.2s;
        
        &:hover, &.active {
            color: #4facfe;
        }
        
        &.active {
            font-weight: 600;
        }
    }
}

@keyframes spin {
    to { transform: rotate(360deg); }
}

@keyframes wave {
    0%, 100% { transform: scaleY(1); }
    50% { transform: scaleY(1.5); }
}

@keyframes pulse-play {
    0%, 100% { box-shadow: 0 0 0 0 rgba(79, 172, 254, 0.4); }
    50% { box-shadow: 0 0 0 12px rgba(79, 172, 254, 0); }
}
</style>
