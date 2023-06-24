<template>
	<view>
		<trtc-room id="trtc-room" :config="trtcConfig"></trtc-room>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				trtcConfig:{
					sdkAppID: this.tencent.trtc.appid,
					userID: '', // 用户唯一标识符
					userSig: '', // 用户签名
					template: 'grid' // 画面排版模式
				},
				id:null,
				roomId:null // 视频会议室id
			};
		},
		onLoad:function(options){
			let that=this
			that.id=options.id
			that.roomId=options.roomId
			// 进入全屏模式
			uni.setKeepScreenOn({
				keepScreenOn:true
			})
			// 获取页面的trtc-room组件
			let trtcRoomContext=that.selectComponent("#trtc-room")
			// 获取trtc-room组件的各种事件上下文对象
			let EVENT=trtcRoomContext.EVENT
			if(trtcRoomContext){
				trtcRoomContext.on(EVENT.LOCAL_JOIN, event => {
				    // 进房成功后发布本地音频流和视频流
				    trtcRoomContext.publishLocalVideo();
				    trtcRoomContext.publishLocalAudio();
				});
				// 监听远端用户的视频流的变更事件
				trtcRoomContext.on(EVENT.REMOTE_VIDEO_ADD, event => {
				    // 订阅（即播放）远端用户的视频流
				    let userID = event.data.userID;
				    let streamType = event.data.streamType; // 'main' or 'aux'
				    trtcRoomContext.subscribeRemoteVideo({ userID: userID, streamType: streamType });
				});
				// 监听远端用户的音频流的变更事件
				trtcRoomContext.on(EVENT.REMOTE_AUDIO_ADD, event => {
				    // 订阅（即播放）远端用户的音频流
				    let userID = event.data.userID;
				    trtcRoomContext.subscribeRemoteAudio({ userID: userID });
				});
			}
			that.trtcRoomContext=trtcRoomContext
		},
		onShow:function(){
			let that=this
			uni.setKeepScreenOn({
				keepScreenOn:true
			})
			// 获取用户签名
			that.ajax(that.url.genUserSig,"GET",null,function(resp){
				that.trtcConfig.userID=resp.data.email
				that.trtcConfig.userSig=resp.data.userSig
				// 完成之后进入视频会议室
				that.$nextTick(function(){
					that.trtcRoomContext.enterRoom({roomID:that.roomId}).catch(res=>{
						console.error('进入会议室失败:', res)
					})
				})
			})
		},
		methods: {
			
		}
	}
</script>

<style lang="less">
@import url("video_meeting.less");
</style>
