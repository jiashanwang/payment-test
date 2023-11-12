<template>
	<view>
		<view class="payMethodLogo">
			<image class="img" src="../../static/wx.png" mode="widthFix"></image>
		</view>
		<view class="title">微信安全支付</view>
		<view class="title orderno">订单号:{{orderNo}}</view>
		<view class="order-area">
			<view class="item">
				<view class="icon">
					<image class="img" src="/static/tiaoxingma.png"></image>
				</view>
				<view class="inputBox orderInput">
					<input class="input" v-model="orderNo" disabled />
				</view>
			</view>
			<view class="item">
				<view class="icon">
					<image class="img" src="/static/username.png"></image>
				</view>
				<view class="inputBox">
					<input class="input" v-model="userName" placeholder="请输入用户名称" />
				</view>
			</view>
			<view class="item">
				<view class="icon">
					<image class="img" src="/static/amount.png"></image>
				</view>
				<view class="inputBox">
					<input class="input" v-model="amount" @blur="changeAmount" type="digit" placeholder="请输入支付金额" />
				</view>
			</view>
			<view class="item payCodeBox" @tap="getPayCode">
				获取付款码
			</view>
		</view>
		<view class="qrcode titleDesc" v-if="url"> 微信扫描二维码支付 </view>	
		<view class="qrcode titleAmount" v-if="url"> 支付金额：{{amount}} 元</view>	
		
		<view class="qrcode">
			<canvas id="qrcode" canvas-id="qrcode" style="width: 200px;height: 200px;"></canvas>
		</view>
		
	</view>
</template>

<script>
	import UQRCode from 'uqrcodejs'; // npm install uqrcodejs
	export default {
		data() {
			return {
				orderNo: "", //订单号
				userName:"",// 用户姓名
				amount:"",// 用户支付金额
				token:"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkYXRhIjp7ImFwcF9pZCI6InRkeGo0bzZ3aDFmcW5wMiIsImFwcF9zZWNyZXQiOiIxMTRlMGFmNTIxMGZjNTljZGZmNmIxZTFiNjg5MmY3NiJ9fQ.x274LsJRM854i3FdREiB3yKb-lPfUzMPi8zXhQaOEZQ",
				payMethod:"wxpay", // 支付方式发wxpay alipay
				url:"",// 支付链接
			}
		},
		onLoad() {
			this.orderNo = this.randomNumber();
		},
		methods: {
			/**
			 * 获取支付二维码
			 */
			getPayCode(){	
				debugger;
				let payMethod = this.payMethod;
				let token = this.token;
				uni.request({
					// url: 'https://www.atwillpay.cn/paymentzpw/main/createOrder',
					url: 'http://192.168.10.101:4001/paymentzpw/main/createOrder',
					data: {
						outOrderNo: this.orderNo,
						amount: this.amount,
						notify_url: "https://www.atwillpay.com/paymentzpw/common/notifyToApp",// 测试回调通知
						payMethod: payMethod,
					},
					method: "POST",
					header: {
						'Authorization': 'Bearer ' + this.token
					},
					success: (res) => {
						debugger;
						let result = res.data;				
						if (result.code == 0) {
							// 成功
							if (result.data.paymentType == 1) {
				
							} else if (result.data.paymentType == 2) {
								//跳转到二维码支付页面
								if (result.data.code == 200){
									// 获取成功
									let urlData = JSON.parse(result.data.message);
									let url = urlData.code_url;
									let params = {
										outOrderNo: this.orderNo,
										amount: this.amount,
										url,
										userName: this.userName
									};
									this.url = url;
									this.getQrcodeImg(url);
									// let queryData = encodeURIComponent(JSON.stringify(params))
									// uni.navigateTo({
									// 	url: "/pages/qrcodePay/index?data=" + queryData
									// });
								}else{
									uni.showToast({
										title: "原生支付通道获取失败",
										icon: 'none'
									})
								}
							} else if (result.data.paymentType == 3) {
								let url = result.data.url_link;
								window.open(url)
							}
						} else {
							uni.showToast({
								title: "支付通道获取失败",
								icon: 'none'
							})
						}
				
					}
				});
			},
			// 获取支付二维码
			getQrcodeImg(url) {
				var qr = new UQRCode();
				// 设置二维码内容
				qr.data = url;
				// 设置二维码大小，必须与canvas设置的宽高一致
				qr.size = 200;
				// 调用制作二维码方法
				qr.make();
				// 获取canvas上下文
				var canvasContext = uni.createCanvasContext('qrcode', this); // 如果是组件，this必须传入
				// 设置uQRCode实例的canvas上下文
				qr.canvasContext = canvasContext;
				// 调用绘制方法将二维码图案绘制到canvas上
				qr.drawCanvas();
			},
			randomNumber() {
				const now = new Date()
				let month = now.getMonth() + 1
				let day = now.getDate()
				let hour = now.getHours()
				let minutes = now.getMinutes()
				let seconds = now.getSeconds()
				month = month < 10 ? "0" + month : month;
				day = day < 10 ? "0" + day : day;
				hour = hour < 10 ? "0" + hour : hour;
				minutes = minutes < 10 ? "0" + minutes : minutes;
				seconds = seconds < 10 ? "0" + seconds : seconds;
				let orderCode = now.getFullYear().toString() + month.toString() + day + hour + minutes + seconds + (Math
					.round(Math.random() * 1000000)).toString();
				return orderCode;
			},
		}
	}
</script>

<style lang="scss" scoped>
	.payMethodLogo {
		display: flex;
		justify-content: center;
		align-content: center;
		margin-top: 40rpx;
		.img {
			width: 250rpx;
		}
	}

	.title {
		text-align: center;
		margin: 25rpx;
	}
	.qrcode {
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.titleDesc{
		margin:20rpx;
		color:#409eff;
	}
	.titleAmount{
		margin:30rpx;
	}
	.order-area {
		border: 1rpx dashed #327ab7;
		margin: 40rpx 40rpx;
		.item {
			display: flex;
			justify-content: flex-start;
			margin: 20rpx;
			height: 60rpx;

			.icon {
				display: flex;
				justify-content: center;
				align-items: center;
				background-color: #eeeeee;
				width: 80rpx;
				height: 100%;
				border-right: 1rpx dashed #cccccc;
				border-top-left-radius: 10%;
				border-bottom-left-radius: 10%;

				.img {
					width: 44rpx;
					height: 44rpx;
				}
			}

			.inputBox {
				flex: auto;
				border-top-right-radius: 30%;
				border-bottom-right-radius: 30%;
				height: 100%;

				.input {
					// background-color: #eeeeee;
					border:1rpx solid #eeeeee;
					height: 100%;
					// width:100%;
					padding-left: 20rpx;
					font-size: 26rpx;
				}
			}
			.orderInput {
				.input{
					background-color: #eeeeee;
				}
			}
		}
		.payCodeBox{
			display: flex;
			justify-content: center;
			align-items: center;
			// border:1rpx solid #01c800;
			background-color: #01c800;
			border-radius: 10rpx;
			padding:10rpx;
			color:#ffffff;
			cursor: pointer;
			margin-top:50rpx;
		}
	
	}
</style>