<template>
  <view class="container">
    <!-- 对话内容区 -->
    <scroll-view class="chat-container" scroll-y="true" :scroll-with-animation="true" :scroll-top="scrollTop">
      <!-- 欢迎语 -->
      <view class="welcome-message">
        <text>你好，有什么我能帮你的吗</text>
      </view>
      
      <!-- 对话记录 -->
      <view class="message-list">
        <!-- 用户消息 -->
        <view class="message-item user-message" v-for="(item, index) in chatHistory" :key="index" v-if="item.role === 'user'">
          <view class="avatar user-avatar">
            <image src="/static/user-avatar.png" mode="widthFix"></image>
          </view>
          <view class="message-content user-content">
            <text>{{ item.content }}</text>
          </view>
        </view>
        
        <!-- AI消息 -->
        <view class="message-item ai-message" v-for="(item, index) in chatHistory" :key="index" v-if="item.role === 'assistant'">
          <view class="avatar ai-avatar">
            <image src="/static/ai-avatar.png" mode="widthFix"></image>
          </view>
          <view class="message-content ai-content">
            <text>{{ item.content }}</text>
          </view>
        </view>
        
        <!-- 加载状态 -->
        <view class="loading" v-if="isLoading">
          <view class="loading-dot"></view>
          <view class="loading-dot"></view>
          <view class="loading-dot"></view>
        </view>
      </view>
    </scroll-view>
    
    <!-- 输入区域 -->
    <view class="input-area">
      <view class="input-container">
        <textarea 
          class="input-box" 
          placeholder="请输入你的问题..." 
          v-model="inputText" 
          @input="autoResize"
          @confirm="sendMessage"
          maxlength="500"
        ></textarea>
        <button class="send-btn" @click="sendMessage" :disabled="!inputText.trim() || isLoading">
          <text class="send-icon">→</text>
        </button>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      chatHistory: [], // 对话历史记录
      inputText: '', // 输入框内容
      scrollTop: 0, // 滚动位置
      isLoading: false, // 加载状态
      textareaHeight: 40 // 输入框初始高度
    };
  },
  onLoad() {
    // 适配小程序导航栏
    uni.setNavigationBarTitle({
      title: 'AI知识库'
    });
    uni.setNavigationBarColor({
      frontColor: '#000000',
      backgroundColor: '#f5f7f5'
    });
  },
  methods: {
    // 自动调整输入框高度
    autoResize(e) {
      const textarea = e.target;
      // 限制最大高度，适配小程序
      if (textarea.scrollHeight > 120) {
        this.textareaHeight = 120;
      } else {
        this.textareaHeight = textarea.scrollHeight;
      }
    },
    
    // 发送消息
    async sendMessage() {
      const question = this.inputText.trim();
      if (!question || this.isLoading) return;
      
      // 添加用户消息到历史记录
      this.chatHistory.push({
        role: 'user',
        content: question
      });
      
      // 清空输入框
      this.inputText = '';
      this.textareaHeight = 40;
      
      // 滚动到底部
      this.scrollToBottom();
      
      // 显示加载状态
      this.isLoading = true;
      
      try {
        // 调用阿里云智能体API获取回答
        const answer = await this.callAliyunAI(question);
        
        // 添加AI回答到历史记录
        this.chatHistory.push({
          role: 'assistant',
          content: answer
        });
      } catch (error) {
        // 错误处理
        this.chatHistory.push({
          role: 'assistant',
          content: '抱歉，暂时无法获取回答，请稍后再试。'
        });
        console.error('API调用错误:', error);
      } finally {
        // 隐藏加载状态
        this.isLoading = false;
        // 滚动到底部
        this.scrollToBottom();
      }
    },
    
    // 滚动到最新消息
    scrollToBottom() {
      // 延迟执行确保DOM已更新，适配小程序滚动
      setTimeout(() => {
        this.scrollTop = 99999;
      }, 100);
    },
    
    // 调用阿里云智能体API
    async callAliyunAI(question) {
      // 阿里云智能体API调用示例（需替换为实际接口）
      try {
        const response = await uni.request({
          url: 'https://your-aliyun-api-endpoint',
          method: 'POST',
          data: {
            prompt: question,
            // 其他API所需参数
          },
          header: {
            'Content-Type': 'application/json',
            'Authorization': '你的API授权信息'
          }
        });
        
        if (response[1].statusCode === 200) {
          return response[1].data.result; // 根据实际返回结构调整
        } else {
          throw new Error('API请求失败');
        }
      } catch (error) {
        console.error('阿里云API调用失败:', error);
        throw error;
      }
    }
  }
};
</script>

<style scoped>
/* 基础样式 */
.container {
  display: flex;
  flex-direction: column;
  height: 100vh; /* 小程序全屏高度 */
  background-color: #f5f7f5; /* 浅灰绿背景 */
}

/* 对话内容区样式 */
.chat-container {
  flex: 1;
  padding: 20rpx 15rpx;
  box-sizing: border-box;
}

/* 欢迎语样式 - 高级绿色调 */
.welcome-message {
  background-color: #eaf7f0; /* 淡绿色背景 */
  color: #333;
  padding: 24rpx 30rpx;
  border-radius: 20rpx;
  margin: 20rpx auto 40rpx; /* 居中显示 */
  font-size: 30rpx;
  text-align: center;
  max-width: 80%;
  box-shadow: 0 2rpx 8rpx rgba(76, 175, 80, 0.1);
}

.message-list {
  display: flex;
  flex-direction: column;
  gap: 30rpx;
}

.message-item {
  display: flex;
  align-items: flex-start;
  gap: 20rpx;
  max-width: 100%;
}

/* 头像样式 */
.avatar {
  width: 80rpx;
  height: 80rpx;
  border-radius: 16rpx;
  overflow: hidden;
  flex-shrink: 0;
  background-color: #f0f5f0;
}

.avatar image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* 消息内容样式 */
.message-content {
  max-width: 75%;
  padding: 20rpx 30rpx;
  border-radius: 20rpx;
  font-size: 30rpx;
  line-height: 1.6;
  word-break: break-all; /* 适配小程序文本换行 */
}

/* 用户消息样式 */
.user-message {
  flex-direction: row-reverse;
}

.user-content {
  background-color: #4caf50; /* 主色调-高级绿色 */
  color: white;
  border-top-right-radius: 8rpx;
}

/* AI消息样式 */
.ai-content {
  background-color: white;
  color: #333;
  border-top-left-radius: 8rpx;
  box-shadow: 0 2rpx 8rpx rgba(0,0,0,0.05);
}

/* 加载动画 */
.loading {
  display: flex;
  gap: 12rpx;
  padding: 20rpx 30rpx;
  align-items: center;
}

.loading-dot {
  width: 16rpx;
  height: 16rpx;
  border-radius: 50%;
  background-color: #4caf50; /* 绿色加载点 */
  animation: loading 1.4s infinite ease-in-out both;
}

.loading-dot:nth-child(1) {
  animation-delay: -0.32s;
}

.loading-dot:nth-child(2) {
  animation-delay: -0.16s;
}

@keyframes loading {
  0%, 80%, 100% {
    transform: scale(0.6);
  }
  40% {
    transform: scale(1);
  }
}

/* 输入区域样式 - 小程序适配 */
.input-area {
  padding: 20rpx 15rpx;
  background-color: white;
  border-top: 1px solid #eee;
}

.input-container {
  display: flex;
  gap: 20rpx;
  align-items: flex-end;
}

.input-box {
  flex: 1;
  min-height: 80rpx;
  max-height: 240rpx;
  padding: 20rpx 30rpx;
  background-color: #f5f7f5;
  border-radius: 40rpx;
  font-size: 30rpx;
  line-height: 1.4;
  resize: none;
  outline: none;
  /* 去除小程序默认样式 */
  border: none;
}

.send-btn {
  width: 80rpx;
  height: 80rpx;
  border-radius: 50%;
  background-color: #4caf50; /* 绿色发送按钮 */
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40rpx;
  padding: 0;
  opacity: 0.95;
  /* 去除小程序按钮默认样式 */
  border: none;
  line-height: normal;
}

.send-btn:disabled {
  background-color: #b2d8b7; /* 禁用状态浅绿色 */
  opacity: 0.7;
}

.send-icon {
  transform: rotate(45deg);
  display: block;
}
</style>