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
			<view class="item payment" @tap="payClick">
				<image class="wxlogo" src="/static/wxpay.png"></image>
				<text>微信支付</text>
			</view>
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
				orderNo:"",
				goodsName:"支付测试",
				amount:1,
				payType:2,// 支付交互形式：1. 拉起H5支付 2. 生成二维码支付（NATIVE支付） 3. 拉起小程序支付 4. 拉起公众号支付 5. APP支付
			}
		},
		onLoad() {
			this.orderNo = this.randomNumber();
		},
		methods: {
			changeAmount(val){
				debugger;
				let amount = Number(val.target.value);
				if (amount < 0.1){
					uni.showToast({
					  title: '金额不能小于0.1元',
					})
				}
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
			    let orderCode = now.getFullYear().toString() + month.toString() + day + hour + minutes + seconds + (Math.round(Math.random() * 1000000)).toString();
			    return orderCode;
			},
			payClick(){
				debugger;
				uni.request({
				    url: 'https://www.atwillpay.cn/payment/main/createOrder',
				    data: {
				        outOrderNo:this.orderNo,
						amount:this.amount,
						notify_url:"https://www.xuezhangstore.com",
						payType:this.payType,
						goodsName:this.goodsName
				    },
					method:"POST",
				    header: {
				        'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhcHBfaWQiOiJ0ZDMxbGl4ZW16ajZxb2IiLCJhcHBfc2VjcmV0IjoiODI5YTA5ODhiNWQ4ZDc0MjBlNmQ1NGRmN2MzZmNlNTgifQ.EiXMN1XdqXdQnsJRKhQwQSL0ZeAlNfn6NQ5XBl40zlg' //自定义请求头信息
				    },
				    success: (res) => {
				        console.log(res.data);
						debugger;
						let result = res.data;
						if (this.payType == 1){
							
						}else if (this.payType == 2){
							//跳转到二维码支付页面
							let url = result.data;
							let params = {
								outOrderNo:this.orderNo,
								amount:this.amount,
								url,
								goodsName:this.goodsName
							}
							debugger;
							let queryData = encodeURIComponent(JSON.stringify(params))
							uni.navigateTo({
								url: "/pages/qrcodePay/index?data=" + queryData
							});
						}else if (this.payType == 3 && result.code == 0){
							let url = result.data;
							window.open(url)
						}
				    }
				});
			}
		}
	}
</script>

<style lang="scss" scoped>
	.title-content{
		padding-top:60rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		.logo{
			width:50rpx;
			height: 50rpx;
			margin-right:20rpx;
			border-radius: 50%;
		}
	}
	.order-area{
		border:1rpx solid #327ab7;
		margin:40rpx 20rpx;
		.item{
			display:flex;
			justify-content: flex-start;
			margin:20rpx;
			height:60rpx;
			.icon{
				display: flex;
				justify-content: center;
				align-items: center;
				background-color: #eeeeee;
				width:80rpx;
				height:100%;
				border-right:1rpx dashed #cccccc;		
				border-top-left-radius: 10%;
				border-bottom-left-radius: 10%;
				.img{
					width:44rpx;
					height:44rpx;
				}
			}
			.inputBox{
				flex:auto;
				border-top-right-radius: 30%;
				border-bottom-right-radius: 30%;
				height:100%;
				.input{
					background-color: #eeeeee;
					height: 100%;
					// width:100%;
					padding-left:20rpx;
					font-size:26rpx;
				}
			}
		}
		.payment{
			margin-top:40rpx;
			margin-bottom:40rpx;
			border:1rpx solid #eeeeee;
			padding:12rpx 0rpx;
			display: flex;
			justify-content: center;
			align-items: center;
			.wxlogo{
				width:36rpx;
				height:36rpx;
				margin-right:15rpx;
			}
		}
		.footer{
			display: flex;
			justify-content: center;
			align-items: center;
			background-color: #eeeeee;
			padding:16rpx 0rpx;
			font-size:24rpx;
		}
	}
</style>
