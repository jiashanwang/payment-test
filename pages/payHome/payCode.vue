<template>
	<view>
<!-- 		<view class="payMethodLogo">
			<image class="img" v-if="payMethod=='wxpay'" src="../../static/wxlogo.png" mode="widthFix"></image>
			<image class="img" v-if="payMethod=='alipay'" src="../../static/alilogo.png" mode="widthFix"></image>
		</view> -->
		<view class="company">
			<view class="name">财易付远程收款</view>
			<view class="method">
				<view class="type">支付方式</view>
			</view>
			<view class="payMethodList">
				<view :class="{'item':true,'active':selectedType=='wxpay'}" @tap="chanageMethodType('wxpay')">
					<image class="img" src="../../static/wxpay.png" mode="widthFix"></image>
				</view>
				<view  :class="{'item':true,'active':selectedType=='alipay'}" @tap="chanageMethodType('alipay')">
					<image class="img" src="../../static/ali.png" mode="widthFix"></image>
				</view>
			</view>
		</view>
		<!-- <view class="title" v-if="payMethod=='wxpay'">微信安全支付</view>
		<view class="title" v-if="payMethod=='alipay'">支付宝安全支付</view> -->
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
			<view class="item payCodeBox" v-if="payMethod=='wxpay'" @tap="getPayCode">
				获取付款码
			</view>
			<view class="item payCodeBox payAliBgColor" v-if="payMethod=='alipay'" @tap="getPayCode">
				立即支付
			</view>
		</view>
		<view class="qrcode titleDesc" v-if="url && payMethod=='wxpay'"> 微信扫描二维码打开小程序支付 </view>	
		<view class="qrcode titleDesc" v-if="url && payMethod=='alipay'"> 支付宝或浏览器扫描二维码支付 </view>	
		<view class="qrcode titleAmount" v-if="url"> 支付金额：<span class="amountBold">{{amount}}</span> 元</view>	
		
		<view class="qrcode qrcodeBox">
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
				token:"",
				payMethod:"wxpay", // 支付方式发wxpay alipay
				url:"",// 支付链接
				selectedType:"wxpay",
			}
		},
		onLoad() {
			this.orderNo = this.randomNumber();
			let query = this.$route.query;
			// this.payMethod = query.payMethod?query.payMethod:"alipay";
			if (query.appid && query.appsecret){
				let token = window.sessionStorage.getItem("token");
				if (token){
					this.token = token;
				}else {
					this.getToken(query.appid,query.appsecret);
				}
			}
		},
		methods: {
			changeAmount(event) {
				this.amount = Number(event.target.value.match(/^\d*(\.?\d{0,2})/g)[0]);
			},
			getToken(appid,appsecret){
				uni.request({
					url: 'http://1.14.43.168/paymentcmj/common/getToken',
					// url: 'http://127.0.0.1:4001/paymentcmj/common/getToken',
					data: {
						app_id: appid,
						app_secret:appsecret,
					},
					method: "POST",
					success: (res) => {
						let result = res.data;
						if (result.code == 0) {
							// 成功
							let token = result.data;
							this.token = token;
							window.sessionStorage.setItem("token",token);
						} else {
							uni.showToast({
								title: "token获取失败",
								icon: 'none'
							});
							window.sessionStorage.removeItem("token");
						}
					}
				});
			},
			/**
			 * 获取支付二维码
			 */
			getPayCode(){	
				// 支付金额不能为空
				if(!this.amount || Number(this.amount) == 0){
					uni.showToast({
					    title: '请输入支付金额!',
					    //将值设置为 success 或者直接不用写icon这个参数
					    icon: 'none',
					    //显示持续时间为 2秒
					    duration: 2000
					});
					return ;
				}
				uni.showLoading({
					title: '正在连接，请稍候...',
					mask: true,
				});
				let payMethod = this.payMethod;
				let token = this.token;
				uni.request({
					url: 'http://1.14.43.168/paymentcmj/main/createOrder',
					// url: 'http://127.0.0.1:4001/paymentcmj/main/createOrder',
					data: {
						outOrderNo: this.orderNo,
						amount: this.amount,
						notify_url: "",// 测试回调通知
						payMethod: payMethod,
						userName:this.userName
					},
					method: "POST",
					header: {
						'Authorization': 'Bearer ' + this.token
					},
					success: (res) => {
						uni.hideLoading();
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
									this.url = url;
									this.getQrcodeImg(url);
								}else{
									uni.showToast({
										title: "微信原生付款码获取失败",
										icon: 'none'
									})
								}
							} else if (result.data.paymentType == 3) {
								// 微信小程序支付
								let url = result.data.url_link;
								this.url = url;
								this.getQrcodeImg(url);
							}else if (result.data.paymentType == 5){
								// 支付宝H5支付
								let url = result.data.url;
								this.url = url;
								// 生成支付码的方式 beging
								// this.getQrcodeImg(url);
								// 生成支付码的方式 end
								//直接拉起支付的方式 begin
								this.generateCode(url);
								//直接拉起支付的方式 begin
							
							}
						} else {
							uni.showToast({
								title: "付款码获取失败",
								icon: 'none'
							})
						}
				
					},
					fail: (err) => {},
					complete: () => {
						uni.hideLoading();
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
			//直接拉起支付
			generateCode(url){
				// 方法1:
				 // const newPage = window.open(); 
				 // newPage.location.href = url
				 // 方法2: 推荐
			  const a = document.createElement('a');
				   const id = 'newpage'
				   a.setAttribute('href', url);
				   a.setAttribute('target', '_self');
				   a.setAttribute('id', id);
				   // 防止反复添加
				   if(!document.getElementById(id)) {
					   document.body.appendChild(a);
				   }
				   a.click();
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
			/**
			 * 支付方式的切换
			 */
			chanageMethodType(method){
				this.selectedType = method;
				this.payMethod = method;
			},
		}
	}
</script>

<style lang="scss" scoped>
	.payMethodList{
		width:60%;
		margin:0 auto;
		margin-top:30rpx;
		display: flex;
		justify-content: space-between;
		align-items: center;
		.img{
			width:70rpx;
		}
		.item{
			padding:20rpx;
			padding:16rpx 20rpx;
			margin-right:30rpx;
			background-color: #ffffff;
			border-radius: 10rpx;
		}
		.active{
			border:1rpx solid #327ab7;
			background-color: #f8f8f8;
		}
	}
	.method{
		margin:20rpx 20rpx 10rpx 20rpx;
		width:60%;
		color:#7f7f7f;
		font-size:30rpx;
	}
	.payMethodLogo {
		display: flex;
		justify-content: center;
		align-content: center;
		margin-top: 40rpx;
		.img {
			width: 250rpx;
		}
	}
	.company{
		// width:80%;
		margin:0 30rpx 30rpx 30rpx;
		// background-color: #f8f8f8;
		background-color: #ffffff;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-content: center;
		padding:20rpx 20rpx 50rpx 20rpx;
		border-bottom:1rpx dashed #cccccc;
		.name{
			font-family: Arial, "Microsoft YaHei";	
			font-size:40rpx;
			color:#585858;
			font-weight: bold;
			text-align: center;
		}
	}
	.amountBold{
		color:#ff0000;
		font-size:40rpx;
		display: inline-block;
		margin-right:14rpx;
	}
	.title {
		text-align: center;
		margin: 25rpx;
	}
	.orderno{
		font-size:32rpx;
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
		// border: 1rpx dashed #327ab7;
		margin: 40rpx 40rpx;
		.item {
			display: flex;
			justify-content: flex-start;
			margin: 20rpx;
			height: 88rpx;

			.icon {
				display: flex;
				justify-content: center;
				align-items: center;
				background-color: #eeeeee;
				width: 110rpx;
				height: 104%;
				border-right: 1rpx dashed #cccccc;
				border-top-left-radius: 10%;
				border-bottom-left-radius: 10%;

				.img {
					width: 54rpx;
					height: 54rpx;
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
					font-size: 36rpx;
				}
			}
			.orderInput {
				.input{
					background-color: #eeeeee;
					font-size:36rpx;
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
		.payAliBgColor{
			background-color: #00a9f1;
		}
		.qrcodeBox{
			padding-bottom:80rpx;
		}
	}
</style>