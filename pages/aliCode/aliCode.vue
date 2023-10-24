<template>
	<view>
			
	</view>
</template>

<script>
	export default {
		data() {
			return {
			}
		},
		methods: {
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
			/**
			 * 获取支付宝支付链接
			 */
			getAliPayFormData(){
				uni.request({
					// url: 'https://www.atwillpay.cn/payment/common/getToken',
					url: 'http://10.32.203.162:4005/payback/miniprogram/getAliPayObjectTest',
					data: {
						// app_id: appid,
						// app_secret:appsecret,
					},
					method: "GET",
					success: (res) => {
						let result = res.data;
						this.generateCode(result);
						// if (result.code == 0) {
						// 	// 成功
						// 	this.token = result.data;
						// } else {
						// 	uni.showToast({
						// 		title: "token获取失败",
						// 		icon: 'none'
						// 	})
						// }
					}
				});
			},
		},
	
		onLoad(options) {
			this.getAliPayFormData();
		},
	}
</script>

<style>

</style>
