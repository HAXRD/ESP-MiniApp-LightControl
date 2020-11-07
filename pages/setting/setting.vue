<template>
	<view>
		<view class="conn-state-box">
			<image src="../../static/bluetooth-enabled.png" v-if="btnConn" mode="widthFix"></image>
			<image src="../../static/bluetooth-disabled.png" v-if="!btnConn" mode="widthFix"></image>
		</view>
		<view class="info-box" v-if="!btnConn">
			<text>请打开手机蓝牙</text>
			<text>并点击链接按钮链接蓝牙设备</text>
		</view>
		<view class="sett-box" v-if="btnConn">
			<view class="red-set" @tap="control(1)">
				<text>红灯</text>
			</view>
			<view class="green-set" @tap="control(2)">
				<text>绿灯</text>
			</view>
			<view class="blue-set" @tap="control(3)">
				<text>蓝灯</text>
			</view>
		</view>
		<view class="conn-btn-box" v-if="!btnConn">
			<button type='primary' :disabled="!btnOpen" @tap="connectBLE">{{btnOpen?'链接蓝牙':'请开启蓝牙'}}</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				btnOpen: false,
				btnConn: false,
				deviceId: '?',
				serviceId: '?',
				characteristicId: '?'
			};
		},
		onShow() {
			var that = this
			// 开启蓝牙适配器，如果返回失败则说明手机蓝牙没打开，提示用户打开手机蓝牙
			uni.openBluetoothAdapter({
				success(res) {
					that.btnOpen = true
					that.connectBLE()
					console.log(res)
				},
				fail(res) {
					// 提示用户开启手机蓝牙并监听蓝牙开启状态
					uni.showToast({
						title: '请开启手机蓝牙',
						duration: 1500,
						icon: none,
						mask: true
					})
					that.listenAdapterState()
				}
			})
		},
		onHide() {
			var that = this
			// 在页面隐藏的时候关闭链接销毁adapter
			uni.closeBLEConnection({
				deviceId: that.deviceId,
				success(res) {
					uni.closeBluetoothAdapter({
						success(res) {
							console.warn('页面隐藏，销毁蓝牙成功')
						}
					})
				}
			})
		},
		methods: {
			control(type) {
				var that = this
				const buffer = new ArrayBuffer(1)
				const dataView = new DataView(buffer)
				dataView.setUint8(0, parseInt(type).toString(16))
				uni.writeBLECharacteristicValue({
					deviceId: that.deviceId,
					serviceId: that.serviceId,
					characteristicId: that.characteristicId,
					value: buffer,
					success(res) {
						console.log('数据发送成功，内容为: ', buffer)
					}
				})
			},
			listenAdapterState() {
				var that = this
				uni.onBluetoothAdapterStateChange(function(res){
					// 蓝牙开启
					if (res.available) {
						that.btnOpen = true
					} else {
						that.btnOpen = false
					}
				})
			},
			connectBLE() {
				var that = this
				uni.startBluetoothDevicesDiscovery({
					services: ['service-UUID'], // 此处输入ESP32硬件代码中的service-UUID
					success(res) {
						// 发现蓝牙后回调
						uni.onBluetoothDeviceFound(function(res){
							that.deviceId = res.devices[0].deviceId
							console.log('发现设备', res)
							console.error(res[0])
							// 创建蓝牙链接
							uni.createBLEConnection({
								deviceId: that.deviceId,
								success(res) {
									console.log('链接蓝牙成功')
									that.btnConn = true
									// 监听蓝牙状态
									uni.onBLEConnectionStateChange(function(res){
										that.btnConn = res.connected
										console.log('蓝牙链接发生改变: ', res.connected)
									})
									// 蓝牙链接成功后搜索一下服务，做兼容性处理
									uni.getBLEDeviceServices({
										// 这里的 deviceId 需要已经通过 createBLEConnection 与对应设备建立链接
										deviceId: that.deviceId,
										success(res) {
											uni.getBLEDeviceCharacteristics({
												deviceId: that.deviceId,
												// 这里的 serviceId 需要在 getBLEDeviceServices 接口中获取
												serviceId: that.serviceId,
												success(res) {
													console.log('获取到所有特征', res.characteristics)
												}
											})
										}
									})
								}
							})
							
						})
						
					}
				})
			}
		}
	}
</script>

<style lang="scss">
	.conn-state-box {
		width: 100%;
		display: flex;
		justify-content: flex-end;
		padding-top: 20rpx;
		image {
			width: 40rpx;
			height: 40rpx;
			margin-right: 60rpx;
		}
	}
	
	.info-box {
		width: 100%;
		height: 200rpx;
		position: absolute;
		top: 40%;
		left: 50%;
		transform: translate(-50%, -50%);
		display: flex;
		flex-direction: column;
		justify-content: center;
		background-color: #F0AD4E;
		color: #FFFFFF;
		text-align: center;
		text {
			margin-top: 15rpx;
		}
	}
	
	.sett-box {
		width: 90%;
		position: absolute;
		top: 40%;
		left: 50%;
		transform: translate(-50%, -50%);
		height: 200rpx;
		display: flex;
		justify-content: space-between;
		text {
			color: #FFFFFF;
			font: bold;
		}
		.red-set {
			background-color: #FF0000;
			width: 200rpx;
			border-radius: 100rpx;
			display: flex;
			justify-content: center;
			text-align: center;
			align-items: center;
		}
		.green-set {
			background-color: #32CD32;
			width: 200rpx;
			border-radius: 100rpx;
			display: flex;
			justify-content: center;
			text-align: center;
			align-items: center;
		}
		.blue-set {
			background-color: #4169E1;
			width: 200rpx;
			border-radius: 100rpx;
			display: flex;
			justify-content: center;
			text-align: center;
			align-items: center;
		}
		
	}
	
	.conn-btn-box {
		width: 100%;
		position: absolute;
		bottom: 10%;
		display: flex;
		justify-content: center;
	}
</style>
