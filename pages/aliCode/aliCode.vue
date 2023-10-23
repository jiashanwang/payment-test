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
				let divForm = document.getElementsByTagName('divform')
				
				if (divForm.length) {
				
				    document.body.removeChild(divForm[0])
				
				}
				
				const div = document.createElement('divform')
				
				// div.innerHTML = res.data.alipayOrderStr // 放入支付宝的表单数据
				div.innerHTML = url  //付宝的表单数据
				
				document.body.appendChild(div)
				
				document.forms[0].setAttribute('target', '_blank') // 新开窗口跳转
				
				document.forms[0].submit()
			},
			/**
			 * 获取支付宝FormData
			 */
			getAliPayFormData(){
				uni.request({
					// url: 'https://www.atwillpay.cn/payment/common/getToken',
					url: 'http://10.32.203.162:4005/payback/miniprogram/getDCAliPayObject',
					data: {
						// app_id: appid,
						// app_secret:appsecret,
					},
					method: "GET",
					success: (res) => {
						debugger;
						let result = res.data;
						let obj = JSON.parse(result)
						console.log(result)
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
