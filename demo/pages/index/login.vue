<template>
  <!-- 添加过渡动画容器 -->
  <view class="transition-container" :class="{'fade-in': isVisible}">
    <!-- 原有内容保持不变 -->
  <view class="auth-container">
    <!-- 主容器 -->
    <view class="auth-card">
      <!-- 标题区域 -->
      <view class="auth-header">
        <text class="title">{{ 
          currentMode === 'login' ? '欢迎登录' : 
          currentMode === 'register' ? '手机注册' : 
          '忘记密码' 
        }}</text>
        <view class="title-underline"></view>
      </view>
      
      <!-- 表单区域 -->
      <form class="auth-form">
        <!-- 手机号输入框（所有模式） -->
        <view class="input-group" :class="{ error: phoneError }">
          <uni-icons type="phone" size="22" color="#0a7124"></uni-icons>
          <input 
            type="tel" 
            v-model="phone" 
            placeholder="请输入手机号" 
            class="input-field"
            @input="clearError('phone')"
            maxlength="11"
          />
        </view>
        <text class="error-text" v-if="phoneError">{{ phoneError }}</text>
        
        <!-- 验证码输入框（注册/忘记密码） -->
        <view v-if="currentMode !== 'login'" class="input-group verify-group" :class="{ error: codeError }">
          <uni-icons type="code" size="22" color="#2e7d32"></uni-icons>
          <input 
            type="text" 
            v-model="verifyCode" 
            placeholder="请输入验证码" 
            class="input-field"
            @input="clearError('code')"
            maxlength="6"
          />
          <button class="verify-btn" :disabled="codeDisabled" @click="sendCode">
            {{ codeText }}
          </button>
        </view>
        <text class="error-text" v-if="currentMode !== 'login' && codeError">{{ codeError }}</text>
        
        <!-- 密码输入框（所有模式） -->
        <view class="input-group" :class="{ error: passwordError }">
          <uni-icons type="locked" size="22" color="#2e7d32"></uni-icons>
          <input 
            type="password" 
            v-model="password" 
            :placeholder="
              currentMode === 'login' ? '请输入密码' : 
              currentMode === 'register' ? '设置登录密码（6-20位）' : 
              '请设置新密码（6-20位）'
            " 
            class="input-field"
            @input="clearError('password')"
          />
        </view>
        <text class="error-text" v-if="passwordError">{{ passwordError }}</text>
        
        <!-- 确认密码（注册/忘记密码） -->
        <view v-if="currentMode !== 'login'" class="input-group" :class="{ error: confirmError }">
          <uni-icons type="locked" size="22" color="#2e7d32"></uni-icons>
          <input 
            type="password" 
            v-model="confirmPassword" 
            :placeholder="currentMode === 'register' ? '确认密码' : '确认新密码'" 
            class="input-field"
            @input="clearError('confirm')"
          />
        </view>
        <text class="error-text" v-if="currentMode !== 'login' && confirmError">{{ confirmError }}</text>
        
        <!-- 提交按钮 -->
        <button class="submit-btn" @click="handleSubmit">
          {{ 
            currentMode === 'login' ? '登录' : 
            currentMode === 'register' ? '完成注册' : 
            '重置密码' 
          }}
        </button>
        
        <!-- 辅助链接 -->
        <view class="link-group">
          <text 
            class="link-item" 
            @click="switchMode(currentMode === 'login' ? 'register' : 'login')"
          >
            {{ currentMode === 'login' ? '没有账号？注册' : '已有账号？登录' }}
          </text>
          
          <text class="divider" v-if="currentMode !== 'forgot'">|</text>
          
          <text 
            class="link-item" 
            @click="switchMode('forgot')"
            v-if="currentMode !== 'forgot'"
          >
            忘记密码
          </text>
          
          <text 
            class="link-item" 
            @click="switchMode('login')"
            v-if="currentMode === 'forgot'"
          >
            返回登录
          </text>
        </view>
      </form>
    </view>
    
    <!-- 底部信息 -->
    <view class="footer">
      <text class="copyright">开启大先生小先生共学之旅</text>
    </view>
  </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      // 添加过渡控制变量
      isVisible: false,
      // 表单数据
      phone: '',         // 手机号
      password: '',      // 密码
      confirmPassword: '',// 确认密码
      verifyCode: '',    // 验证码
      
      // 错误信息
      phoneError: '',
      passwordError: '',
      confirmError: '',
      codeError: '',
      
      // 验证码状态
      codeText: '获取验证码',
      codeDisabled: false,
      codeCount: 60,
      smsCode: '',       // 存储服务器返回的验证码，实际开发中由后端生成
      
      // 当前模式：login/register/forgot
      currentMode: 'login'
    }
  },
  onShow() {
    // 页面显示时触发渐入动画
    setTimeout(() => {
      this.isVisible = true
    }, 100)
  },
  methods: {
    // 切换模式
    switchMode(mode) {
      this.currentMode = mode
      this.resetForm()
    },
    
    // 重置表单
    resetForm() {
      this.phone = ''
      this.password = ''
      this.confirmPassword = ''
      this.verifyCode = ''
      this.clearAllErrors()
    },
    
    // 清除错误
    clearError(type) {
      if (type === 'phone') this.phoneError = ''
      if (type === 'password') this.passwordError = ''
      if (type === 'confirm') this.confirmError = ''
      if (type === 'code') this.codeError = ''
    },
    
    // 清除所有错误
    clearAllErrors() {
      this.phoneError = ''
      this.passwordError = ''
      this.confirmError = ''
      this.codeError = ''
    },
    
    // 验证手机号格式
    isPhoneValid(phone) {
      const reg = /^1[3-9]\d{9}$/
      return reg.test(phone)
    },
    
    // 发送验证码
    sendCode() {
      // 验证手机号
      if (!this.phone) {
        this.phoneError = '请输入手机号'
        return
      }
      
      if (!this.isPhoneValid(this.phone)) {
        this.phoneError = '请输入正确的手机号'
        return
      }
      
      // 实际开发中调用后端接口发送验证码
      uni.showLoading({ title: '发送中...' })
      
      // 模拟发送验证码（实际开发中替换为阿里云短信服务调用）
      setTimeout(() => {
        uni.hideLoading()
        
        // 生成随机6位验证码
        this.smsCode = Math.floor(100000 + Math.random() * 900000).toString()
        
        // 开发环境下显示验证码（生产环境移除）
        uni.showToast({
          title: `验证码已发送，验证码：${this.smsCode}`,
          icon: 'none',
          duration: 3000
        })
        
        // 开始倒计时
        this.codeDisabled = true
        this.codeText = `${this.codeCount}s后重发`
        
        const timer = setInterval(() => {
          this.codeCount--
          this.codeText = `${this.codeCount}s后重发`
          
          if (this.codeCount <= 0) {
            clearInterval(timer)
            this.codeText = '获取验证码'
            this.codeDisabled = false
            this.codeCount = 60
          }
        }, 1000)
      }, 1000)
    },
    
    // 验证表单
    validateForm() {
      let isValid = true
      
      // 验证手机号
      if (!this.phone) {
        this.phoneError = '请输入手机号'
        isValid = false
      } else if (!this.isPhoneValid(this.phone)) {
        this.phoneError = '请输入正确的手机号'
        isValid = false
      }
      
      // 验证密码
      if (!this.password) {
        this.passwordError = '请输入密码'
        isValid = false
      } else if (this.password.length < 6 || this.password.length > 20) {
        this.passwordError = '密码长度必须为6-20位'
        isValid = false
      }
      
      // 验证确认密码（注册/忘记密码）
      if (this.currentMode !== 'login') {
        if (!this.confirmPassword) {
          this.confirmError = this.currentMode === 'register' ? '请确认密码' : '请确认新密码'
          isValid = false
        } else if (this.confirmPassword !== this.password) {
          this.confirmError = '两次密码输入不一致'
          isValid = false
        }
      }
      
      // 验证验证码（注册/忘记密码）
      if (this.currentMode !== 'login') {
        if (!this.verifyCode) {
          this.codeError = '请输入验证码'
          isValid = false
        } else if (this.verifyCode.length !== 6) {
          this.codeError = '验证码为6位数字'
          isValid = false
        } else if (this.verifyCode !== this.smsCode) { // 实际开发中由后端验证
          this.codeError = '验证码不正确'
          isValid = false
        }
      }
      
      return isValid
    },
    
    // 处理提交
    handleSubmit() {
      this.clearAllErrors()
      
      if (!this.validateForm()) {
        return
      }
      // 显示加载状态
      uni.showLoading({
        title: this.currentMode === 'login' ? '登录中...' : 
               this.currentMode === 'register' ? '注册中...' : 
               '重置中...'
      })
      // 模拟提交到后端
      setTimeout(() => {
        uni.hideLoading()
        
        if (this.currentMode === 'login') {
          // 实际开发中：调用登录接口，验证手机号和密码
          uni.showToast({ title: '登录成功', icon: 'success' })
          setTimeout(() => {
            uni.switchTab({ url: '/pages/index/index' })
          }, 1000)
        } else if (this.currentMode === 'register') {
          // 实际开发中：调用注册接口，检查手机号是否已注册
          uni.showToast({ title: '注册成功', icon: 'success' })
          setTimeout(() => {
            this.switchMode('login')
          }, 1000)
        } else {
          // 实际开发中：调用密码重置接口
          uni.showToast({ title: '密码已重置', icon: 'success' })
          setTimeout(() => {
            this.switchMode('login')
          }, 1000)
        }
      }, 1500)


// // 后端接口调用
//       try {
//         // 根据模式确定API端点
//         let apiEndpoint = '';
//         if (this.currentMode === 'login') {
//           apiEndpoint = '/auth/login';
//         } else if (this.currentMode === 'register') {
//           apiEndpoint = '/auth/register';
//         } else {
//           apiEndpoint = '/auth/reset-password';
//         }
//
//         // 构造请求数据
//         const requestData = {
//           phone: this.phone,
//           password: this.password
//         };
//
//         // 添加验证码和确认密码（非登录模式）
//         if (this.currentMode !== 'login') {
//           requestData.verifyCode = this.verifyCode;
//           requestData.confirmPassword = this.confirmPassword;
//         }
//
//         // 发起API请求
//         const response = await uni.request({
//           url: `${this.$env.API_BASE}${apiEndpoint}`,
//           method: 'POST',
//           header: { 'Content-Type': 'application/json' },
//           data: requestData
//         });
//
//         uni.hideLoading();
//
//         // 处理响应
//         if (response.statusCode === 200) {
//           // 存储token
//           uni.setStorageSync('token', response.data.token);
//
//           uni.showToast({
//             title: this.currentMode === 'login' ? '登录成功' :
//                 this.currentMode === 'register' ? '注册成功' :
//                     '密码已重置',
//             icon: 'success'
//           });
//
//           // 跳转逻辑
//           setTimeout(() => {
//             if (this.currentMode === 'login') {
//               uni.switchTab({ url: '/pages/index/index' });
//             } else {
//               this.switchMode('login');
//             }
//           }, 1000);
//         } else {
//           // 处理错误响应
//           const errorMsg = response.data?.message || '请求失败，请重试';
//           uni.showToast({ title: errorMsg, icon: 'none' });
//         }
//       } catch (error) {
//         uni.hideLoading();
//         uni.showToast({ title: '网络错误，请检查连接', icon: 'none' });
//         console.error('API请求错误:', error);
//       }
//     }
    }
  }
}
</script>

<style scoped>
/* 添加过渡动画样式 */
.transition-container {
  opacity: 0;
  transition: opacity 0.8s ease-in-out;
}
.fade-in {
  opacity: 1;
}
/* 登录注册页面样式 */
.auth-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background-color: #f1f8e9; /* 浅绿色背景 */
  padding: 0 30rpx;
}
.auth-card {
  width: 100%;
  max-width: 500rpx;
  background-color: #ffffff;
  border-radius: 16rpx;
  padding: 60rpx 40rpx;
  box-shadow: 0 4rpx 20rpx rgba(46, 125, 50, 0.1);
}

.auth-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 50rpx;
}

.title {
  font-size: 36rpx;
  color: #1b5e20;
  font-weight: 600;
}

.title-underline {
  width: 60rpx;
  height: 4rpx;
  background-color: #4caf50;
  margin-top: 15rpx;
  border-radius: 2rpx;
}
/* 输入框*/
.auth-form {
  display: flex;
  flex-direction: column;
  gap: 20rpx;
  margin-top: 30rpx;
}

.input-group {
  display: flex;
  align-items: center;
  padding: 0 25rpx;
  height: 90rpx;
  border: 2rpx solid #c8e6c9;
  border-radius: 10rpx;
  background-color: #f1f8e9;
  transition: all 0.3s;
  margin: 20rpx;
}

.input-group.error {
  border-color: #e53935;
  background-color: #ffebee;
}

.input-group:focus-within {
  border-color: #4caf50;
  box-shadow: 0 0 0 5rpx rgba(76, 175, 80, 0.1);
}

.verify-group {
  position: relative;
  padding-right: 150rpx;
}

.verify-btn {
  position: absolute;
  right: 5rpx;
  top: 50%;
  transform: translateY(-50%);
  width: 140rpx;
  height: 70rpx;
  line-height: 70rpx;
  background-color: #e8f5e9;
  color: #2e7d32;
  border: none;
  border-radius: 8rpx;
  font-size: 24rpx;
  padding: 0;
  margin: 0;
}

.verify-btn:disabled {
  background-color: #e0e0e0;
  color: #9e9e9e;
}

.input-field {
  flex: 1;
  height: 100%;
  font-size: 28rpx;
  color: #212121;
  padding-left: 20rpx;
  background: transparent;
}

.error-text {
  font-size: 24rpx;
  color: #e53935;
  height: 28rpx;
  line-height: 28rpx;
  padding-left: 5rpx;
  margin-top: -10rpx;
  margin-bottom: 10rpx;
}

.submit-btn {
  height: 96rpx;
  background-color: #2e7d32;
  color: #ffffff;
  font-size: 32rpx;
  font-weight: 500;
  border-radius: 10rpx;
  margin-top: 30rpx;
  border: none;
  transition: background-color 0.3s;
}

.submit-btn:hover {
  background-color: #1b5e20;
}

.link-group {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 40rpx;
  gap: 20rpx;
  font-size: 26rpx;
}

.link-item {
  color: #2e7d32;
  text-decoration: none;
}

.link-item:active {
  color: #1b5e20;
}

.divider {
  color: #c8e6c9;
  font-size: 28rpx;
}

.footer {
  margin-top: 60rpx;
}

.copyright {
  font-size: 24rpx;
  color: #66bb6a;
}
</style>