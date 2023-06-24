<template>
	<view class="message">
		<view class="header">
			<view class="desc">{{sendTime}}</view>
			<view class="opt" @tap="deleteMsg()">删除</view>
		</view>
		<view class="content">{{msg}}</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				id: null,
				readFlag: null,
				refId: null,
				sendTime: '',
				msg: ''				
			};
		},
		onShow:function(){
			let that = this
			uni.setNavigationBarTitle({
				title:"系统通知"
			})
			that.ajax(that.url.searchMessageById, "POST", {"id": that.id}, function(resp){
				let result = resp.data.result
				that.sendTime = result.sendTime
				that.msg = result.msg
			})
		},
		// 通过:to="" 跳转过来的传参（message_list），可以通过onLoad:function(params){}获得
		onLoad:function(options){
			let that = this
			that.id = options.id
			// 传递的是字符串，转换一下
			that.readFlag = options.readFlag == "true" ? true : false
			that.refId = options.refId
			// 如果未读
			if (!that.readFlag){
				that.ajax(that.url.updateUnreadMessage, "POST", {"id": that.refId}, function(resp){
					console.log("消息更新为已读")
				})
			}
		},
		methods: {
			deleteMsg: function(){
				let that=this
				uni.showModal({
					title:"提示信息",
					content:"是否要删除这条消息？",
					success:function(resp){
						if(resp.confirm){
							that.ajax(that.url.deleteMessageRefById,"POST",{"id":that.refId},function(resp){
								uni.showToast({
									icon:"success",
									title:"删除成功",
									// 1秒后回退到上一个页面
									complete:function(){
										setTimeout(function(){
											uni.navigateBack({
												delta:1
											})
										},1000)
									}
								})
							})
						}
					}
				})
			}
		}
	}
</script>

<style lang="less">
	@import url("message.less");
</style>
