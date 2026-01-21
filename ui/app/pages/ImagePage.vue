<!--
 * File: ui/app/pages/ImagePage.vue
 * Description: Image generation interface using Gemini image models
 *
 * Maintainers: AI Assistant
-->

<template>
    <div class="image-page">
        <div class="image-container">
            <!-- Header -->
            <header class="page-header">
                <div class="header-left">
                    <router-link to="/" class="back-btn" :title="t('backToStatus')">
                        <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <path d="m15 18-6-6 6-6"/>
                        </svg>
                    </router-link>
                    <h1>{{ t('imageTitle') }}</h1>
                </div>
                <div class="header-right">
                    <el-select v-model="selectedModel" :placeholder="t('selectModel')" class="model-select">
                        <el-option
                            v-for="model in imageModels"
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
                    <div class="prompt-card">
                        <h3>{{ t('imagePromptTitle') }}</h3>
                        <textarea
                            v-model="prompt"
                            :placeholder="t('imagePromptPlaceholder')"
                            rows="4"
                            :disabled="isGenerating"
                        ></textarea>
                        <div class="prompt-actions">
                            <button 
                                class="generate-btn" 
                                @click="generateImage"
                                :disabled="!prompt.trim() || isGenerating"
                            >
                                <svg v-if="!isGenerating" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                                    <path d="m21.44 11.05-9.19 9.19a6 6 0 0 1-8.49-8.49l8.57-8.57A4 4 0 1 1 18 8.84l-8.59 8.57a2 2 0 0 1-2.83-2.83l8.49-8.48"/>
                                </svg>
                                <span v-if="isGenerating" class="loading-spinner"></span>
                                <span>{{ isGenerating ? t('generating') : t('generate') }}</span>
                            </button>
                            <button class="clear-btn" @click="clearAll" :disabled="isGenerating">
                                {{ t('clearAll') }}
                            </button>
                        </div>
                    </div>
                    
                    <!-- Prompt Examples -->
                    <div class="examples-card">
                        <h4>{{ t('promptExamples') }}</h4>
                        <div class="examples-list">
                            <button 
                                v-for="example in promptExamples" 
                                :key="example"
                                class="example-btn"
                                @click="prompt = example"
                            >
                                {{ example }}
                            </button>
                        </div>
                    </div>
                </div>

                <!-- Results Section -->
                <div class="results-section">
                    <div v-if="generatedImages.length === 0 && !isGenerating" class="empty-state">
                        <div class="empty-icon">
                            <svg xmlns="http://www.w3.org/2000/svg" width="80" height="80" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1" stroke-linecap="round" stroke-linejoin="round">
                                <rect width="18" height="18" x="3" y="3" rx="2" ry="2"/>
                                <circle cx="9" cy="9" r="2"/>
                                <path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/>
                            </svg>
                        </div>
                        <h2>{{ t('imageWelcome') }}</h2>
                        <p>{{ t('imageWelcomeDesc') }}</p>
                    </div>

                    <div v-if="isGenerating" class="generating-state">
                        <div class="generating-animation">
                            <div class="pulse-ring"></div>
                            <div class="pulse-ring delay-1"></div>
                            <div class="pulse-ring delay-2"></div>
                            <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                                <rect width="18" height="18" x="3" y="3" rx="2" ry="2"/>
                                <circle cx="9" cy="9" r="2"/>
                                <path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/>
                            </svg>
                        </div>
                        <p>{{ t('imageGenerating') }}</p>
                    </div>

                    <div v-if="generatedImages.length > 0" class="images-grid">
                        <div 
                            v-for="(img, index) in generatedImages" 
                            :key="index" 
                            class="image-card"
                            @click="openLightbox(index)"
                        >
                            <img :src="img.url" :alt="img.prompt" />
                            <div class="image-overlay">
                                <span class="prompt-preview">{{ img.prompt }}</span>
                                <div class="image-actions">
                                    <button @click.stop="downloadImage(img)" :title="t('download')">
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
                <router-link to="/image" class="nav-item active">
                    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect width="18" height="18" x="3" y="3" rx="2" ry="2"/><circle cx="9" cy="9" r="2"/><path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/></svg>
                    <span>{{ t('navImage') }}</span>
                </router-link>
                <router-link to="/tts" class="nav-item">
                    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 2a3 3 0 0 0-3 3v7a3 3 0 0 0 6 0V5a3 3 0 0 0-3-3Z"/><path d="M19 10v2a7 7 0 0 1-14 0v-2"/><line x1="12" x2="12" y1="19" y2="22"/></svg>
                    <span>{{ t('navTTS') }}</span>
                </router-link>
            </nav>
        </div>

        <!-- Lightbox -->
        <div v-if="lightboxVisible" class="lightbox" @click="closeLightbox">
            <div class="lightbox-content" @click.stop>
                <button class="lightbox-close" @click="closeLightbox">×</button>
                <img :src="generatedImages[lightboxIndex]?.url" :alt="generatedImages[lightboxIndex]?.prompt" />
                <div class="lightbox-info">
                    <p>{{ generatedImages[lightboxIndex]?.prompt }}</p>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import { ElMessage } from 'element-plus';
import I18n from '../utils/i18n';

const t = (key) => I18n.t(key);

const prompt = ref('');
const isGenerating = ref(false);
const generatedImages = ref([]);
const selectedModel = ref('gemini-2.5-flash-image');
const lightboxVisible = ref(false);
const lightboxIndex = ref(0);

const imageModels = ref([
    { id: 'gemini-2.5-flash-image', name: 'Gemini 2.5 Flash Image' },
    { id: 'gemini-2.5-flash', name: 'Gemini 2.5 Flash (支持图文生成)' },
]);

const promptExamples = ref([
    '一只戴着太空头盔的柴犬在月球上散步',
    '赛博朋克风格的未来城市夜景',
    '水彩风格的樱花树下的日本庭院',
    '可爱的卡通猫咪喝咖啡',
]);

const generateImage = async () => {
    if (!prompt.value.trim() || isGenerating.value) return;
    
    isGenerating.value = true;
    
    try {
        const response = await fetch('/v1/chat/completions', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': 'Bearer 123456'
            },
            body: JSON.stringify({
                model: selectedModel.value,
                messages: [
                    {
                        role: 'user',
                        content: prompt.value
                    }
                ],
                stream: false
            })
        });
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);
        }
        
        const data = await response.json();
        const content = data.choices?.[0]?.message?.content;
        
        // Check if response contains base64 image
        if (content && content.includes('data:image')) {
            // Extract base64 image from markdown or raw content
            const imgMatch = content.match(/data:image\/[^;]+;base64,[A-Za-z0-9+/=]+/);
            if (imgMatch) {
                generatedImages.value.unshift({
                    url: imgMatch[0],
                    prompt: prompt.value,
                    timestamp: new Date().toISOString()
                });
                ElMessage.success(t('imageGenSuccess'));
            } else {
                ElMessage.warning(t('imageNoImageInResponse'));
            }
        } else {
            ElMessage.warning(t('imageNoImageInResponse'));
        }
    } catch (error) {
        ElMessage.error(t('imageGenError') + ': ' + error.message);
    } finally {
        isGenerating.value = false;
    }
};

const clearAll = () => {
    prompt.value = '';
    generatedImages.value = [];
};

const downloadImage = (img) => {
    const link = document.createElement('a');
    link.href = img.url;
    link.download = `generated-image-${Date.now()}.png`;
    link.click();
};

const openLightbox = (index) => {
    lightboxIndex.value = index;
    lightboxVisible.value = true;
};

const closeLightbox = () => {
    lightboxVisible.value = false;
};
</script>

<style lang="less" scoped>
@import '../styles/variables.less';

.image-page {
    min-height: 100vh;
    background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
    display: flex;
    flex-direction: column;
}

.image-container {
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
    background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
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
            width: 250px;
            
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
    grid-template-columns: 350px 1fr;
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
    
    .prompt-card {
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
            min-height: 100px;
            font-family: inherit;
            
            &:focus {
                outline: none;
                border-color: #f093fb;
            }
        }
        
        .prompt-actions {
            display: flex;
            gap: 10px;
            margin-top: 16px;
            
            .generate-btn {
                flex: 1;
                display: flex;
                align-items: center;
                justify-content: center;
                gap: 8px;
                padding: 12px 20px;
                background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
                color: white;
                border: none;
                border-radius: 8px;
                font-size: 1em;
                font-weight: 500;
                cursor: pointer;
                transition: transform 0.2s, box-shadow 0.2s;
                
                &:hover:not(:disabled) {
                    transform: translateY(-2px);
                    box-shadow: 0 4px 12px rgba(240, 147, 251, 0.4);
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
                padding: 12px 16px;
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
    
    .examples-card {
        background: white;
        border-radius: 12px;
        padding: 16px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
        
        h4 {
            margin: 0 0 12px;
            font-size: 0.9em;
            color: #666;
        }
        
        .examples-list {
            display: flex;
            flex-direction: column;
            gap: 8px;
            
            .example-btn {
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
                    background: #f093fb;
                    color: white;
                    border-color: #f093fb;
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
            color: #f093fb;
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
        .generating-animation {
            position: relative;
            width: 100px;
            height: 100px;
            display: flex;
            align-items: center;
            justify-content: center;
            margin-bottom: 20px;
            
            .pulse-ring {
                position: absolute;
                width: 100%;
                height: 100%;
                border: 2px solid #f093fb;
                border-radius: 50%;
                animation: pulse 2s ease-out infinite;
                
                &.delay-1 { animation-delay: 0.5s; }
                &.delay-2 { animation-delay: 1s; }
            }
            
            svg {
                color: #f093fb;
            }
        }
    }
    
    .images-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
        gap: 16px;
        
        .image-card {
            position: relative;
            aspect-ratio: 1;
            border-radius: 12px;
            overflow: hidden;
            cursor: pointer;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
            transition: transform 0.2s, box-shadow 0.2s;
            
            &:hover {
                transform: translateY(-4px);
                box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
                
                .image-overlay {
                    opacity: 1;
                }
            }
            
            img {
                width: 100%;
                height: 100%;
                object-fit: cover;
            }
            
            .image-overlay {
                position: absolute;
                inset: 0;
                background: linear-gradient(transparent 50%, rgba(0, 0, 0, 0.7) 100%);
                opacity: 0;
                transition: opacity 0.2s;
                display: flex;
                flex-direction: column;
                justify-content: flex-end;
                padding: 16px;
                
                .prompt-preview {
                    color: white;
                    font-size: 0.85em;
                    line-height: 1.4;
                    display: -webkit-box;
                    -webkit-line-clamp: 2;
                    -webkit-box-orient: vertical;
                    overflow: hidden;
                }
                
                .image-actions {
                    display: flex;
                    gap: 8px;
                    margin-top: 10px;
                    
                    button {
                        width: 36px;
                        height: 36px;
                        border-radius: 50%;
                        border: none;
                        background: rgba(255, 255, 255, 0.2);
                        color: white;
                        cursor: pointer;
                        display: flex;
                        align-items: center;
                        justify-content: center;
                        transition: background 0.2s;
                        
                        &:hover {
                            background: rgba(255, 255, 255, 0.4);
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
            color: #f093fb;
        }
        
        &.active {
            font-weight: 600;
        }
    }
}

.lightbox {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.9);
    z-index: 1000;
    display: flex;
    align-items: center;
    justify-content: center;
    
    .lightbox-content {
        position: relative;
        max-width: 90vw;
        max-height: 90vh;
        
        .lightbox-close {
            position: absolute;
            top: -40px;
            right: 0;
            width: 36px;
            height: 36px;
            border: none;
            background: rgba(255, 255, 255, 0.2);
            color: white;
            font-size: 24px;
            border-radius: 50%;
            cursor: pointer;
            
            &:hover {
                background: rgba(255, 255, 255, 0.3);
            }
        }
        
        img {
            max-width: 100%;
            max-height: 80vh;
            border-radius: 8px;
        }
        
        .lightbox-info {
            margin-top: 16px;
            color: white;
            text-align: center;
            
            p {
                margin: 0;
                opacity: 0.8;
            }
        }
    }
}

@keyframes spin {
    to { transform: rotate(360deg); }
}

@keyframes pulse {
    0% {
        transform: scale(1);
        opacity: 1;
    }
    100% {
        transform: scale(1.5);
        opacity: 0;
    }
}
</style>
