<template>
	<view>
		<image src="../../static/logo-1.png" class="logo" mode="widthFix"></image>
		<view class="logo-title">EMOS企业在线办公系统</view>
		<view class="logo-subtitle">Ver 2050.6</view>
		<button class="login-btn" open-type="getUserInfo" @tap="login()">登录</button>
		<view class="register-container">
			没有账号？
			<text class="register" @tap="toRegister()">立即注册</text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				
			};
		},
		methods:{
			toRegister:function(){
				uni.navigateTo({
					url:"../register/register"
				})
			},
			login:function(){
				let that = this
				uni.login({
					provider:"weixin",
					success:function(resp){
						// 临时授权凭证
						let code = resp.code;
						that.ajax(that.url.login, "POST", {"code":code}, function(resp){
							let permission = resp.data.permission
							uni.setStorageSync("permission", permission)
							// 跳转index页面(index为一个导航栏,用switchTab跳转)
							uni.switchTab({
								url: "../index/index"
							})
						})
					},
					fail:function(e){
						uni.showToast({
							icon:"none",
							title:"执行异常"
						})
					}
				})
			}
		}
	}
</script>

<style lang="less">
	@import url("login.less");
</style>
