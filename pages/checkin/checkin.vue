<template>
	<view>
		<camera device-position="front" flash="off" class="camera" @error="error()" v-if="showCamera"></camera>
		<image mode="widthFix" class="image" :src="photoPath" v-if="showImage"></image>
		<view class="operate-container">
			<button type="primary" class="btn" @tap="clickBtn()" :disabled="!canCheckin">
				{{btnText}}
			</button>
			<button type="warn" class="btn" @tap="afresh()" :disabled="!canCheckin">
				重拍
			</button>
		</view>
		<view class="notice-container">
			<text class="notice">注意事项</text>
			<text class="desc">拍照签到的时候，必须要拍摄自己的正面照片，侧面照片会导致无法识别。另外，拍照的时候不要戴墨镜或者帽子，避免影响拍照签到的准确度。</text>
		</view>
	</view>
</template>

<script>
	// 引入腾讯位置信息服务sdk
	var QQMapWX = require("../../lib/qqmap-wx-jssdk.min.js")
	var qqmapsdk;
	export default {		
		data() {
			return {
				canCheckin: true,
				photoPath: "",
				btnText: "拍照",
				showCamera: true,
				showImage: false
			};
		},
		onLoad:function(){
			// 初始化sdk
			qqmapsdk = new QQMapWX({
				key: "LK6BZ-ICTWG-FAMQB-QHVFI-2KD2S-4TBVQ"
			})
		},
		// 判断当前时间是否可以考勤
		onShow:function(){
			let that = this
			that.ajax(that.url.validCanCheckin, "GET", null, function(resp){
				let msg = resp.data.msg
				if(msg!="可以考勤"){
					that.canCheckin = false
					setTimeout(function(){
						uni.showToast({
							title: msg,
							icon:"none"
						})
					}, 1000)
				}
			})
		},
		methods:{
			clickBtn:function(){
				let that = this
				if (that.btnText == "拍照"){
					// 获得相机上下文
					let ctx = uni.createCameraContext()
					ctx.takePhoto({
						quality:"high",
						success:function(resp){
							// 获取相片路径
							that.photoPath = resp.tempImagePath
							that.showCamera = false
							that.showImage = true
							that.btnText = "签到"
						}
					})
				}else{
					// 签到
					uni.showLoading({
						title:"签到中，请稍后"
					})
					setTimeout(function(){
						uni.hideLoading()
					}, 20000)
					// 获取位置信息
					uni.getLocation({
						type:"wgs84",
						success:function(resp){
							let latitude = resp.latitude
							let longitude = resp.longitude
							// 通过腾讯位置信息服务sdk把位置坐标转化为地址
							qqmapsdk.reverseGeocoder({
								location:{
									latitude:latitude,
									longitude:longitude
								},
								success:function(resp){
									// console.log(resp.result)
									let address = resp.result.address
									let addressComponent = resp.result.address_component
									let nation = addressComponent.nation
									let province = addressComponent.province
									let city = addressComponent.city
									let district = addressComponent.district
									// 签到
									uni.uploadFile({
										url: that.url.checkin,
										filePath: that.photoPath,
										name: "photo",
										header: {
											token: uni.getStorageSync("token")
										},
										formData: {
											address: address,
											country: nation,
											province: province,
											city: city,
											district: district
										},
										success:function(resp){
											if(resp.statusCode == 500 && resp.data == "人脸模型不存在"){
												uni.hideLoading()
												uni.showModal({
													title: "提示信息",
													content: "EMOS系统中不存在你的人脸识别模型，是否用当前这张照片作为人脸识别模型？",
													success:function(resp){
														if(resp.confirm){
															uni.uploadFile({
																url: that.url.createFaceModel,
																filePath: that.photoPath,
																name: "photo",
																header: {
																	token: uni.getStorageSync("token")
																},
																success:function(resp){
																	if(resp.statusCode == 500){
																		uni.showToast({
																			title: resp.data,
																			icon: 'none'
																		})
																	}else if(resp.statusCode == 200){
																		uni.showToast({
																			title: "人脸建模成功",
																			icon: 'none'
																		})
																	}	
																}
															})
														}
													}
												})
											}else if(resp.statusCode == 200){
												let data = JSON.parse(resp.data)
												let code = data.code
												let msg = data.msg
												if(code == 200){
													uni.hideLoading()
													uni.showToast({
														title: msg,
														complete:function(){
															uni.navigateTo({
																url: "../checkin_result/checkin_result"
															})
														}
													})
												}
											}else if(resp.statusCode == 500){
												uni.showToast({
													title: resp.data,
													icon: 'none'
												})
											}
											
										},
										fail:function(e){
											console.log(e)
										}
									})
								}
							})
						}
					})
					
				}
			},
			afresh:function(){
				this.showCamera = true
				this.showImage = false
				this.btnText = "拍照"
			}
		}
	}
</script>

<style lang="less">
	@import url("checkin.less");
</style>
