<template>
	<view>
		<image src="../../static/logo-2.png" mode="widthFix" class="logo"></image>
		<view class="register-container">
			<input type="text" placeholder="输入邀请码" class="register-code" maxlength="6" v-model="registerCode" />
			<view class="register-desc">
				<button class="register-btn" open-type="getUserInfo" @tap="register()">注册</button>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				registerCode:""
			};
		},
		methods:{
			register:function(){
				let that = this
				// 前端验证
				if(that.registerCode == null || that.registerCode.length == 0){
					uni.showToast({
						icon:"none",
						title:"邀请码不能为空"
					})
					return
				}else if(/^[0-9]{6}$/.test(that.registerCode) == false){
					uni.showToast({
						icon:"none",
						title:"邀请码必须是6位数字"
					})
					return
				}
				// login
				uni.login({
					provider:"weixin",
					success:function(resp){
						// 临时授权凭证，用于给后端换取 openid
						let code = resp.code
						// 获取用户信息
						uni.getUserInfo({
							provider:"weixin",
							success:function(resp) {
								// 微信用户名
								let nickName = resp.userInfo.nickName
								// 微信用户头像地址
								let avatarUrl = resp.userInfo.avatarUrl
								let data={
									code: code,
									nickname: nickName,
									photo: avatarUrl,
									registerCode: that.registerCode
								}
								that.ajax(that.url.register, "POST", data, function(resp){
									let permission = resp.data.permission
									uni.setStorageSync("permission", permission)
									console.log(permission)
									// 跳转index页面
									uni.switchTab({
										url: "../index/index"
									})
								})
							}
						})
					}
				})
			}
			
		}
	}
</script>

<style lang="less">
	@import url("register.less");
</style>
