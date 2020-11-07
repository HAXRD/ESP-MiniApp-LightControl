<template>
	<view class="content">
		<view class="text-area">
			<text class="title">{{title}}</text>
			<button type="primary" class="btn-class" @tap="goSetting">开始</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: '欢迎使用\nESP32灯光控制小程序'
			}
		},
		onLoad() {

		},
		methods: {
			goSetting() {
				uni.reLaunch({
					url: '../setting/setting'
				})
			}
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		position: absolute;
		bottom: 50%;
		text-align: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
	
	.btn-class {
		margin-top: 30rpx;
	}
</style>
