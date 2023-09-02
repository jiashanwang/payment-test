<template>
	<view class="content">
		<view class="title-content">
			<image class="logo" src="/static/wxpay.png"></image>
			<view class="title">微信扫码支付</view>
		</view>
		<view class="line-operate"></view>
		<view class="mobile-tips">提示：二维码会风控，请复制下方链接支付</view>
		<view class="amount">
			<h4 class="icon">￥</h4><span class="price">{{data.amount.toFixed(2)}}</span>
		</view>
		<view class="qrcode">
			<canvas id="qrcode" canvas-id="qrcode" style="width: 200px;height: 200px;"></canvas>
		</view>
		<view class="wxpayDesc">
			操作流程：复制链接→打开微信搜索自己微信名→打开聊天对话框→粘贴链接→发送→点击发送出来的蓝色链接→进入付款页面→完成付款
		</view>
		<view class="operate">
			<button size="mini" type="primary">保存相册</button>
				<button size="mini" type="primary">复制链接</button>
		</view>
		<view class="goods" v-if="!isShow">
			<view class="goods-item goods-name">
				<p>购买商品名称</p>
				<p>{{data.goodsName}}</p>
			</view>
			<view class="goods-item goods-name">
				<p>商户订单号</p>
				<p>{{data.outOrderNo}}</p>
			</view>
			<view class="goods-item goods-name">
				<p>订单创建时间</p>
				<p>2023-09-02 12:12:12</p>
			</view>
		</view>
		<view class="showOrHide">
			<image class="img" @click="showOrHideEvent" v-show="!isShow" src="/static/top.png"></image>
			<image class="img" @click="showOrHideEvent" v-show="isShow" src="/static/bottom.png"></image>
		</view>
		<view class="saoyisao">
			<image class="img" src="/static/saoyisao.png"></image>
			<view class="saomadesc">
				请使用微信扫一扫<br>
				扫描二维码完成付款
			</view>
		</view>
		<view class="remark">
			<p class="desc">手机用户可保存上方二维码到手机中</p>
			<p class="desc">在微信扫一扫中选择“相册”即可</p>
		</view>
	</view>
</template>

<script>
	import UQRCode from 'uqrcodejs'; // npm install uqrcodejs
	export default {
		data() {
			return {
				data:{},
				isShow:true
			};
		},
		onReady() {
			this.getQrcodeImg(this.data.url)
		},
		onLoad(options){
			if (options.data){
				console.log("options.data = ",JSON.parse(decodeURIComponent(options.data)))
				this.data = JSON.parse(decodeURIComponent(options.data));
			};
		},
		methods:{
			showOrHideEvent(){
				this.isShow = !this.isShow;
			},
			// 获取支付二维码
			getQrcodeImg(url){
				var qr = new UQRCode();
				// 设置二维码内容
				qr.data = "https://uqrcode.cn/doc";
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
		}
	}
</script>

<style lang="scss">
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
	.line-operate{
		margin-top:30rpx;
		height: 40rpx;
		background-color: #f7f7f7;
	}
	.mobile-tips{
		color:#ff0000;
		margin:40rpx auto;
		text-align:center;
		font-size:26rpx;
	}
	.amount{
		margin-bottom:50rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		font-size:60rpx;
		.icon{
			margin-right:10rpx;
		}
		.price{
			font-size:50rpx;
		}
	}
	.qrcode{
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.wxpayDesc{
		margin:50rpx 20rpx;
		border:1rpx dashed #06ae56;
		padding:30rpx;
		color:#ff0000;
		font-size:26rpx;
	}
	.operate{
		display: flex;
		justify-content: space-around;
		align-items: center;
		margin-bottom:30rpx;
	}
	.goods{
		margin-top:40rpx;
		padding:30rpx;
		.goods-item{
			font-size:26rpx;
			margin:20rpx 0;
			display: flex;
			justify-content: space-between;
			align-items: center;
		}
	}
	.showOrHide{
		border-top:1rpx dashed #cccccc;
		display: flex;
		justify-content: center;
		align-items: center;
		margin-bottom:20rpx;
		.img{
			margin-top:30rpx;
			width:60rpx;
			height: 60rpx;
			cursor: pointer;
		}
	}
	// .fade-enter-active,
	// .fade-leave-active {
	//   transition: opacity .5s ease;
	// }
	
	// .fade-enter,
	// .fade-leave-to {
	//   transition: 0 .5s ease;
	// }
	.saoyisao{
		display: flex;
		justify-content: center;
		align-items: center;
		.img{
			width:100rpx;
			height:100rpx;
			margin-right:30rpx;
		}
		.saomadesc{
			line-height: 55rpx;
			font-size:28rpx;
		}
	}
	.remark{
		margin-top:30rpx;
		padding:30rpx;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		background-color: #f7f7f7;
		font-size:26rpx;
		.desc{
			margin:10rpx;
		}
	}
</style>
