<template>
	<view class="content">
		<view class="title-content">
			<image class="logo" src="/static/caiyifu.jpg"></image>
			<view class="title">财易付 - 测试支付</view>
		</view>
		<view class="order-area">
			<view class="item">
				<view class="icon">
					<image class="img" src="/static/tiaoxingma.png"></image>
				</view>
				<view class="inputBox">
					<input class="input" v-model="orderNo" />
				</view>

			</view>
			<view class="item">
				<view class="icon">
					<image class="img" src="/static/shangpin.png"></image>
				</view>
				<view class="inputBox">
					<input class="input" v-model="goodsName" />
				</view>

			</view>
			<view class="item">
				<view class="icon">
					<image class="img" src="/static/renminbi.png"></image>
				</view>
				<view class="inputBox">
					<input class="input" v-model="amount" @blur="changeAmount" type="digit" placeholder="请输入支付金额" />
				</view>
			</view>
			<view class="item tips">
				Tips: 可自定义支付金额
			</view>
		<!-- 	<view class="item pay-list">
				<view class="pay-type-name">支付形式:</view>
				<radio-group @change="radioChange" class="payTypeListBox">
					<label v-for="(item, index) in payTypeList" :key="item.value" class="payTypeList">
						<view>
							<radio :value="item.value" :disabled="item.disabled" :checked="index === current" />
						</view>
						<view class="pay-item">{{item.name}}</view>
					</label>
				</radio-group>
			</view> -->
			<div class="item payBox payment">
				<view class="payItem" @tap="payClick('wxpay')">
					<image class="wxlogo" src="/static/wxpay.png"></image>
					<text>微信支付</text>
				</view>
				<view class="payItem" @tap="payClick('alipay')">
					<image class="alilogo" src="/static/ali.png"></image>
					<text>支付宝支付</text>
				</view>
			</div>
		
			<view class="footer">
				<text>财易付 © 2023 All Rights Reserved.</text>
			</view>
		</view>
		<view id="myQrcode">

		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				orderNo: "",
				goodsName: "支付测试",
				amount: 1,
				payType: "2", // 支付交互形式：1. 拉起H5支付 2. 生成二维码支付（NATIVE支付） 3. 拉起小程序支付 4. 拉起公众号支付 5. APP支付
				payTypeList: [
					{
						value: "1",
						name: 'H5',
						disabled: true,
					},
					{
						value: "2",
						name: '原生支付',
						checked: 'true',
						disabled: false,
					},
					{
						value: "3",
						name: '小程序',
						disabled: false,
					},
					{
						value: "4",
						name: '公众号',
						disabled: true,
					},
				],
				current: 1, // 单选框下标
			}
		},
		onLoad() {
			this.orderNo = this.randomNumber();
			let query = this.$route.query;
			if (!query.appid || !query.appsecret){
				this.token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkYXRhIjp7ImFwcF9pZCI6InRkeGo0bzZ3aDFmcW5wMiIsImFwcF9zZWNyZXQiOiIxMTRlMGFmNTIxMGZjNTljZGZmNmIxZTFiNjg5MmY3NiJ9fQ.x274LsJRM854i3FdREiB3yKb-lPfUzMPi8zXhQaOEZQ";
			}else {
				this.getToken(query.appid,query.appsecret);
			}
			
		},
		methods: {
			radioChange(event) {
				this.payType = event.detail.value;
				for (let i = 0; i < this.payTypeList.length; i++) {
					if (this.payTypeList[i].value == event.detail.value) {
						this.current = event.detail.value;
						break;
					}
				}
			},
			changeAmount(event) {
				this.amount = Number(event.target.value.match(/^\d*(\.?\d{0,2})/g)[0]);
			},
			getToken(appid,appsecret){
				uni.request({
					url: 'https://www.atwillpay.cn/paymentzpw/common/getToken',
					// url: 'http://192.168.10.101:4001/paymentzpw/common/getToken',
					data: {
						app_id: appid,
						app_secret:appsecret,
					},
					method: "POST",
					success: (res) => {
						let result = res.data;
						if (result.code == 0) {
							// 成功
							this.token = result.data;
						} else {
							uni.showToast({
								title: "token获取失败",
								icon: 'none'
							})
						}
					}
				});
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
			payClick(payMethod) {
				if (payMethod == 2){
					// 支付宝支付
					let params = {
						outOrderNo: this.orderNo,
						amount: this.amount,
						notifyUrl:"https://www.atwillpay.com/paymentzpw/common/notifyToApp",
						goodsName: this.goodsName
					}
					let queryData = encodeURIComponent(JSON.stringify(params))
					uni.navigateTo({
						url: "/pages/aliCode/aliCode?data=" + queryData
					});
					return;
				}
				let token = this.token;
				uni.request({
					url: 'https://www.atwillpay.cn/paymentzpw/main/createOrder',
					// url: 'http://192.168.10.101:4001/paymentzpw/main/createOrder',
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
										goodsName: this.goodsName
									}
									let queryData = encodeURIComponent(JSON.stringify(params))
									uni.navigateTo({
										url: "/pages/qrcodePay/index?data=" + queryData
									});
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
			}
		}
	}
</script>

<style lang="scss" scoped>
	.tips{
		color:#409eff;
		font-size:24rpx;
	}
	.title-content {
		padding-top: 60rpx;
		display: flex;
		justify-content: center;
		align-items: center;

		.logo {
			width: 50rpx;
			height: 50rpx;
			margin-right: 20rpx;
			border-radius: 50%;
		}
	}

	.order-area {
		border: 1rpx solid #327ab7;
		margin: 40rpx 20rpx;

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
					background-color: #eeeeee;
					height: 100%;
					// width:100%;
					padding-left: 20rpx;
					font-size: 26rpx;
				}
			}
		}
		.payBox{
			display: flex;
			justify-content: space-around;
			margin-bottom:30rpx;
			.payItem{
				margin:30rpx;
			}
		}
		.payment {
			margin-top: 40rpx;
			margin-bottom: 40rpx;
			border: 1rpx solid #eeeeee;
			padding: 12rpx 0rpx;
			display: flex;
			justify-content: center;
			align-items: center;

			.wxlogo,.alilogo {
				width: 36rpx;
				height: 36rpx;
				margin-right: 15rpx;
				position: relative;
				top:10rpx;
			}
		}

		.footer {
			display: flex;
			justify-content: center;
			align-items: center;
			background-color: #eeeeee;
			padding: 16rpx 0rpx;
			font-size: 24rpx;
		}
	}

	.pay-list {
		display: flex;
		flex-direction: column;
		justify-content: flex-start;
		padding-bottom: 60rpx;

		.pay-type-name {
			margin: 10rpx 0 20rpx 0;
		}
	}

	.payTypeListBox {
		display: flex;
		justify-content: flex-start;
		align-items: center;
		padding-left: 20rpx;

		.payTypeList {
			display: flex;
			justify-content: flex-start;
			align-items: center;

			.pay-item {
				margin-right: 20rpx;
			}
		}
	}
</style>