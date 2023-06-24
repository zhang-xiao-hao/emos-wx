<template>
	<view class="page">
		<checkbox-group @change="selected">
			<block v-for="dept in list" :key="dept.id">
				<view class="list-title">{{dept.deptName}}（{{ dept.count }}人）</view>
				<view class="item" v-for="member in dept.members" :key="member.userId">
					<view class="key">{{member.name}}</view>
					<checkbox class="value" :value="member.userId" :checked="member.checked"></checkbox>
				</view>
			</block>
		</checkbox-group>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				list: [],
				members: [] //勾选的用户id
			};
		},
		onShow:function(){
			this.loadData(this)
		},
		onLoad:function(options){
			// meeting页面传来的数据
			if(options.hasOwnProperty('members')){
				let members = options.members
				this.members = members.split(',')
			}
		},
		methods:{
			loadData: function(ref){
				ref.ajax(ref.url.searchUserGroupByDept, "POST", {keyword: ref.keyword}, function(resp){
					let result = resp.data.result
					ref.list = result
					for (let dept of ref.list){
						for (let member of dept.members){
							// 勾选已经参会的员工
							if (ref.members.indexOf(member.userId + '') != -1){
								member.checked = true
							}else {
								member.checked = false
							}
						}
					}
				})
			},
			selected: function(e){
				let that = this
				that.members = e.detail.value // 当前所有选中的员工id
				let pages = getCurrentPages() // 当前页面栈
				let prevPage = pages[pages.length - 2] // 上一个页面
				// 向上一个页面传递数据
				prevPage.members = that.members
				prevPage.finishMembers = true // 标志是从members页面跳转到meeting的，而不是meeting_list
			}
		}
	}
</script>

<style lang="less">
	@import url("members.less");
</style>
