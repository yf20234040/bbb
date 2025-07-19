<template>
	<view class="content">
		<image class="logo" src="/static/logo.png"></image>
		<view class="text-area">
			<text class="title">{{title}}</text>
		</view>
    <!-- 添加绿色加载长条 -->
    <view class="progress-bar">
      <view class="progress-fill" :style="{ width: progressWidth }"></view>
    </view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: '你好',
        progressWidth: '0%' // 加载长条初始宽度为0
			}
		},
		onLoad() {
      // 启动进度条动画
      this.startProgressAnimation();

      // 添加3秒定时器，跳转到登录页
      setTimeout(() => {
        uni.redirectTo({
          url: '/pages/index/login'
        })
      }, 3000)
		},
		methods: {
      startProgressAnimation() {
        let width = 0;
        const interval = setInterval(() => {
          width += 1; // 每30ms增加1%
          this.progressWidth = `${width}%`;

          if (width >= 100) {
            clearInterval(interval);
          }
        }, 30); // 3000ms / 100 = 30ms
      }
		}
	}
</script>

<style>
/* 添加进度条样式 */
.progress-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 6rpx;
  background-color: #e0e0e0;
  z-index: 1000;
}

.progress-fill {
  height: 100%;
  background-color: #2e7d32; /* 绿色 */
  transition: width 0.03s linear; /* 平滑过渡效果 */
}

/* 添加内容样式 */
.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 600rpx;
		width: 600rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>
