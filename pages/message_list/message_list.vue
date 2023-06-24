<template>
	<view class="page">
		<uni-list>
			<uni-list-chat v-for="one in list" :key="one.id" :title="one.senderName"
				:note="one.msg" :avatar="one.senderPhoto" badgePositon="left"
				:badgeText="one.readFlag?'':'dot'" link="navigateTo"
				:to="'../message/message?id=' + one.id + '&readFlag=' + one.readFlag + '&refId=' + one.refId">
				<view class="chat-custom-right">
					<text class="chat-custom-text">{{one.sendTime}}</text>
				</view>
			</uni-list-chat>
		</uni-list>
	</view>
</template>

<script>
	// 引入uni列表控件
	import uniList from '@/components/uni-list/uni-list.vue';
	import uniListItem from '@/components/uni-list-item/uni-list-item.vue';
	export default {
		components:{
			uniList,
			uniListItem
		},
		data() {
			return {
				page:1,
				length:20,
				list:[],
				isLastPage:false
			}
		},
		onShow:function(){
			this.page = 1
			this.isLastPage = false
			this.list = []
			uni.pageScrollTo({
				scrollTop:"0"
			})
			this.loadMessageList(this)
		},
		onReachBottom:function(){
			if (this.isLastPage){
				return
			}
			this.page = this.page + 1
			this.loadMessageList(this)
		},
		methods:{
			// 分页查询消息
			loadMessageList: function(ref){
				let data = {
					page: ref.page,
					length: ref.length
				}
				ref.ajax(ref.url.searchMessageByPage, "POST", data, function(resp){
					let result = resp.data.result
					if (result == null || result.length == 0){
						ref.isLastPage = true
						ref.page = ref.page - 1
						uni.showToast({
							icon:"none",
							title:"已经到底了"
						})
					}else{
						// 如果跳转到了其它消息页面，清空消息list
						if(ref.page == 1){
							ref.list = []
						}
						ref.list = ref.list.concat(result)
						if(ref.page > 1){
							uni.showToast({
								icon:"none",
								title:"加载了" + result.length + "条消息"
							})
						}
					}
				})
			}			
		}
	}
</script>

<style lang="less">
	@import url("message_list.less");
</style>
