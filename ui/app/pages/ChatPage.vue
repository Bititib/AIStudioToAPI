<!--
 * File: ui/app/pages/ChatPage.vue
 * Description: Chat interface for AI conversations with Gemini models
 *
 * Maintainers: AI Assistant
-->

<template>
    <div class="chat-page">
        <div class="chat-container">
            <!-- Header -->
            <header class="chat-header">
                <div class="header-left">
                    <router-link to="/" class="back-btn" :title="t('backToStatus')">
                        <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <path d="m15 18-6-6 6-6"/>
                        </svg>
                    </router-link>
                    <h1>{{ t('chatTitle') }}</h1>
                </div>
                <div class="header-right">
                    <el-select v-model="selectedModel" :placeholder="t('selectModel')" class="model-select">
                        <el-option
                            v-for="model in chatModels"
                            :key="model.id"
                            :label="model.name"
                            :value="model.id"
                        />
                    </el-select>
                    <button class="clear-btn" @click="clearChat" :title="t('clearChat')">
                        <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <path d="M3 6h18"/>
                            <path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6"/>
                            <path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"/>
                        </svg>
                    </button>
                </div>
            </header>

            <!-- Messages Area -->
            <div class="messages-container" ref="messagesContainer">
                <div v-if="messages.length === 0" class="empty-state">
                    <div class="empty-icon">
                        <svg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
                            <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>
                        </svg>
                    </div>
                    <h2>{{ t('chatWelcome') }}</h2>
                    <p>{{ t('chatWelcomeDesc') }}</p>
                </div>
                
                <div v-for="(msg, index) in messages" :key="index" :class="['message', msg.role]">
                    <div class="message-avatar">
                        <span v-if="msg.role === 'user'">👤</span>
                        <span v-else>🤖</span>
                    </div>
                    <div class="message-content">
                        <div class="message-text" v-html="formatMessage(msg.content)"></div>
                        <div class="message-time">{{ formatTime(msg.timestamp) }}</div>
                    </div>
                </div>
                
                <div v-if="isLoading" class="message assistant loading">
                    <div class="message-avatar">🤖</div>
                    <div class="message-content">
                        <div class="typing-indicator">
                            <span></span>
                            <span></span>
                            <span></span>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Input Area -->
            <div class="input-container">
                <div class="input-wrapper">
                    <textarea
                        v-model="inputMessage"
                        :placeholder="t('chatPlaceholder')"
                        @keydown.enter.exact="handleSend"
                        @keydown.enter.shift.exact.prevent="inputMessage += '\n'"
                        :disabled="isLoading"
                        ref="inputArea"
                        rows="1"
                    ></textarea>
                    <button 
                        class="send-btn" 
                        @click="handleSend" 
                        :disabled="!inputMessage.trim() || isLoading"
                        :title="t('send')"
                    >
                        <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <line x1="22" x2="11" y1="2" y2="13"/>
                            <polygon points="22 2 15 22 11 13 2 9 22 2"/>
                        </svg>
                    </button>
                </div>
                <div class="input-hint">{{ t('chatInputHint') }}</div>
            </div>
        </div>

        <!-- Navigation -->
        <nav class="bottom-nav">
            <router-link to="/chat" class="nav-item active">
                <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/></svg>
                <span>{{ t('navChat') }}</span>
            </router-link>
            <router-link to="/image" class="nav-item">
                <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect width="18" height="18" x="3" y="3" rx="2" ry="2"/><circle cx="9" cy="9" r="2"/><path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/></svg>
                <span>{{ t('navImage') }}</span>
            </router-link>
            <router-link to="/tts" class="nav-item">
                <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 2a3 3 0 0 0-3 3v7a3 3 0 0 0 6 0V5a3 3 0 0 0-3-3Z"/><path d="M19 10v2a7 7 0 0 1-14 0v-2"/><line x1="12" x2="12" y1="19" y2="22"/></svg>
                <span>{{ t('navTTS') }}</span>
            </router-link>
        </nav>
    </div>
</template>

<script setup>
import { ref, onMounted, nextTick, computed } from 'vue';
import { ElMessage } from 'element-plus';
import I18n from '../utils/i18n';

const t = (key) => I18n.t(key);

const messages = ref([]);
const inputMessage = ref('');
const isLoading = ref(false);
const messagesContainer = ref(null);
const inputArea = ref(null);
const selectedModel = ref('gemini-2.5-flash');

const chatModels = ref([
    { id: 'gemini-2.5-flash', name: 'Gemini 2.5 Flash' },
    { id: 'gemini-2.5-pro', name: 'Gemini 2.5 Pro' },
    { id: 'gemini-flash-latest', name: 'Gemini Flash Latest' },
    { id: 'gemini-pro-latest', name: 'Gemini Pro Latest' },
    { id: 'gemini-3-flash-preview', name: 'Gemini 3 Flash Preview' },
]);

const formatMessage = (content) => {
    // Simple markdown-like formatting
    return content
        .replace(/\n/g, '<br>')
        .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
        .replace(/\*(.*?)\*/g, '<em>$1</em>')
        .replace(/`(.*?)`/g, '<code>$1</code>');
};

const formatTime = (timestamp) => {
    return new Date(timestamp).toLocaleTimeString();
};

const scrollToBottom = () => {
    nextTick(() => {
        if (messagesContainer.value) {
            messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight;
        }
    });
};

const handleSend = async (e) => {
    if (e && e.shiftKey) return;
    if (e) e.preventDefault();
    
    const content = inputMessage.value.trim();
    if (!content || isLoading.value) return;
    
    // Add user message
    messages.value.push({
        role: 'user',
        content: content,
        timestamp: new Date().toISOString()
    });
    
    inputMessage.value = '';
    isLoading.value = true;
    scrollToBottom();
    
    try {
        const response = await fetch('/v1/chat/completions', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': 'Bearer 123456'
            },
            body: JSON.stringify({
                model: selectedModel.value,
                messages: messages.value.map(m => ({
                    role: m.role,
                    content: m.content
                })),
                stream: false
            })
        });
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);
        }
        
        const data = await response.json();
        const assistantMessage = data.choices?.[0]?.message?.content || t('chatNoResponse');
        
        messages.value.push({
            role: 'assistant',
            content: assistantMessage,
            timestamp: new Date().toISOString()
        });
    } catch (error) {
        ElMessage.error(t('chatError') + ': ' + error.message);
        messages.value.push({
            role: 'assistant',
            content: t('chatErrorMessage') + ': ' + error.message,
            timestamp: new Date().toISOString()
        });
    } finally {
        isLoading.value = false;
        scrollToBottom();
    }
};

const clearChat = () => {
    messages.value = [];
};

onMounted(() => {
    inputArea.value?.focus();
});
</script>

<style lang="less" scoped>
@import '../styles/variables.less';

.chat-page {
    min-height: 100vh;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    display: flex;
    flex-direction: column;
}

.chat-container {
    flex: 1;
    display: flex;
    flex-direction: column;
    max-width: 900px;
    margin: 0 auto;
    width: 100%;
    background: rgba(255, 255, 255, 0.95);
    box-shadow: 0 0 40px rgba(0, 0, 0, 0.1);
}

.chat-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 16px 20px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
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
        display: flex;
        align-items: center;
        gap: 12px;
        
        .model-select {
            width: 200px;
            
            :deep(.el-input__wrapper) {
                background: rgba(255, 255, 255, 0.2);
                border: none;
                box-shadow: none;
                
                .el-input__inner {
                    color: white;
                    
                    &::placeholder {
                        color: rgba(255, 255, 255, 0.7);
                    }
                }
            }
        }
        
        .clear-btn {
            display: flex;
            align-items: center;
            justify-content: center;
            width: 36px;
            height: 36px;
            border-radius: 50%;
            background: rgba(255, 255, 255, 0.2);
            border: none;
            color: white;
            cursor: pointer;
            transition: background 0.2s;
            
            &:hover {
                background: rgba(255, 255, 255, 0.3);
            }
        }
    }
}

.messages-container {
    flex: 1;
    overflow-y: auto;
    padding: 20px;
    display: flex;
    flex-direction: column;
    gap: 16px;
}

.empty-state {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    color: #666;
    text-align: center;
    
    .empty-icon {
        color: #667eea;
        margin-bottom: 20px;
        opacity: 0.5;
    }
    
    h2 {
        margin: 0 0 10px;
        font-size: 1.5em;
        color: #333;
    }
    
    p {
        margin: 0;
        color: #888;
    }
}

.message {
    display: flex;
    gap: 12px;
    max-width: 80%;
    
    &.user {
        align-self: flex-end;
        flex-direction: row-reverse;
        
        .message-content {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border-radius: 18px 18px 4px 18px;
        }
        
        .message-time {
            text-align: right;
            color: rgba(255, 255, 255, 0.7);
        }
    }
    
    &.assistant {
        align-self: flex-start;
        
        .message-content {
            background: #f0f2f5;
            color: #333;
            border-radius: 18px 18px 18px 4px;
        }
    }
    
    .message-avatar {
        width: 36px;
        height: 36px;
        border-radius: 50%;
        background: #e0e0e0;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 1.2em;
        flex-shrink: 0;
    }
    
    .message-content {
        padding: 12px 16px;
        
        .message-text {
            line-height: 1.5;
            word-break: break-word;
            
            code {
                background: rgba(0, 0, 0, 0.1);
                padding: 2px 6px;
                border-radius: 4px;
                font-family: monospace;
            }
        }
        
        .message-time {
            font-size: 0.75em;
            margin-top: 6px;
            opacity: 0.7;
        }
    }
}

.typing-indicator {
    display: flex;
    gap: 4px;
    padding: 8px 0;
    
    span {
        width: 8px;
        height: 8px;
        background: #888;
        border-radius: 50%;
        animation: typing 1.4s ease-in-out infinite;
        
        &:nth-child(2) { animation-delay: 0.2s; }
        &:nth-child(3) { animation-delay: 0.4s; }
    }
}

@keyframes typing {
    0%, 60%, 100% { transform: translateY(0); }
    30% { transform: translateY(-6px); }
}

.input-container {
    padding: 16px 20px;
    background: white;
    border-top: 1px solid #eee;
    
    .input-wrapper {
        display: flex;
        gap: 12px;
        align-items: flex-end;
        background: #f5f7fa;
        border-radius: 24px;
        padding: 8px 8px 8px 20px;
        
        textarea {
            flex: 1;
            border: none;
            background: transparent;
            resize: none;
            font-size: 1em;
            line-height: 1.5;
            max-height: 120px;
            outline: none;
            font-family: inherit;
            
            &::placeholder {
                color: #999;
            }
        }
        
        .send-btn {
            width: 44px;
            height: 44px;
            border-radius: 50%;
            border: none;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: transform 0.2s, opacity 0.2s;
            
            &:hover:not(:disabled) {
                transform: scale(1.05);
            }
            
            &:disabled {
                opacity: 0.5;
                cursor: not-allowed;
            }
        }
    }
    
    .input-hint {
        text-align: center;
        font-size: 0.75em;
        color: #999;
        margin-top: 8px;
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
            color: #667eea;
        }
        
        &.active {
            font-weight: 600;
        }
    }
}

@media (max-width: 768px) {
    .chat-header {
        .header-right {
            .model-select {
                width: 140px;
            }
        }
    }
    
    .message {
        max-width: 90%;
    }
}
</style>
