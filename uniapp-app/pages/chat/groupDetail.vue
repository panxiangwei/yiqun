<template>
	<view class="content">
		<u-navbar
			:is-back="true"
			:title="'聊天信息(' + members.length + ')'"
			:background="{ background: '#F6F7F8' }"
			title-color="#404133"
			:border-bottom="false"
			z-index="1001"
		></u-navbar>
		<view style="background-color: #FFFFFF;padding-left: 30rpx;">
			<u-grid :col="6" :border="false">
				<u-grid-item v-for="(item, index) in members" :index="index" :key="item.id" v-if="index<=20" @tap="linkToCard(item.id)">
					<u-avatar :src="$url + item.avatar" mode="square"></u-avatar>
					<view class="grid-text">{{ item.groupNickName||item.nickName }}</view>
				</u-grid-item>
				<u-grid-item @click="showAddGroupUser">
					<view style="padding-bottom: 40rpx;">
						<u-icon name="plus" size="40"></u-icon>
					</view>
				</u-grid-item>
			</u-grid>
			<view @tap="linkToMore" v-if="members.length>20" style="text-align: center;color: #404133;padding-bottom: 10rpx;">查看更多成员</view>
		</view>
		<view style="height: 10rpx;"></view>
		<u-cell-group>
			<u-cell-item title="群名称" @click="showUpdate(group.id, group.groupName, 1)" :value="group.groupName" :title-style="{ marginLeft: '10rpx' }"></u-cell-item>
		</u-cell-group>
		<u-cell-group>
			<u-cell-item title="群二维码" :title-style="{ marginLeft: '10rpx' }"><u-avatar :src="src1" size="40"></u-avatar></u-cell-item>
		</u-cell-group>
		<u-cell-group>
			<u-cell-item title="群公告" @click="showUpdate(group.id, context, 2)" :value="context ? context : '暂无公告'" :title-style="{ marginLeft: '10rpx' }"></u-cell-item>
		</u-cell-group>
		<view style="height: 10rpx;"></view>
		<u-cell-group>
			<u-cell-item
				title="我在本群的昵称"
				@click="showUpdate(group.id, mine.groupNickName || _user_info.nickName, 3)"
				:value="mine.groupNickName || _user_info.nickName"
				:title-style="{ marginLeft: '10rpx' }"
			></u-cell-item>
		</u-cell-group>
		<u-cell-group>
			<u-cell-item title="显示成员昵称" :title-style="{ marginLeft: '10rpx' }" :arrow="false">
				<u-switch active-color="rgb(25, 190, 107)" v-model="isShowNickName" v-on:change="swichShowNickName"></u-switch>
			</u-cell-item>
		</u-cell-group>
		<view style="height: 10rpx;"></view>
		<u-cell-group><u-cell-item title="查看聊天内容" @click="showSearch" :title-style="{ marginLeft: '10rpx' }"></u-cell-item></u-cell-group>
		<u-cell-group><u-cell-item title="设置当前聊天背景" :title-style="{ marginLeft: '10rpx' }" @click="chooseImg"></u-cell-item></u-cell-group>
		<view style="height: 10rpx;"></view>
		<u-cell-group>
			<u-cell-item :title-style="{ marginLeft: '10rpx' }" @click="clearGroupMsg" :arrow="false">
				<view style="text-align: center; color: red;">清空聊天记录</view>
			</u-cell-item>
		</u-cell-group>
		<u-cell-group>
			<u-cell-item :title-style="{ marginLeft: '10rpx' }" @click="removeGroupUser" :arrow="false"><view style="text-align: center; color: red;">删除并退出</view></u-cell-item>
		</u-cell-group>
	</view>
</template>

<script>
export default {
	data() {
		return {
			src1: require('@/static/qrcode.png'),
			chatId: '',
			chatName: '',
			members: [],
			group: {
				groupName:''
			},
			context: '',
			mine: '',
			isShowNickName: false
		};
	},
	methods: {
		linkToMore(){
			this.$u.route({
				url: 'pages/chat/moreMem'
			})
		},
		linkToCard(id){
			this.$u.route({
				url: 'pages/businessCard/businessCard',
				params:{ id: id, source: 1}
			})
		},
		showSearch() {
			this.$u.route({
				url:"pages/search/search",
				params: {searchType: 4, chatId: this.chatObj.chatId}
			})
		},
		showAddGroupUser(){
			//管理员也可以修改
			if (this._user_info.userName != this.group.operUser) {
				uni.showModal({
					title: '无权限修改',
					showCancel: false
				});
				return;
			}
			this.$u.route({
				url: 'pages/chat/addGroupUser'
			});
		},
		removeGroupUser() {
			this.$socket.removeGroupUser([this._user_info.id], this.chatObj.chatId, res => {
				if (res.success) {
					this.$u.route({
						url: 'pages/home/home',
						type: 'switchTab'
					});
				}
			});
		},
		showUpdate(groupId, context, type) {
			if (this._user_info.userName != this.group.operUser && 3 != type) {
				uni.showModal({
					title: '无权限修改',
					showCancel: false
				});
				return;
			}
			this.$u.route({
				url: 'pages/chat/updateGroupInfo',
				params: {  groupId,  context,  type }
			});
		},
		clearGroupMsg() {
			this.$socket.clearGroupMsg(this._user_info.id, this.chatObj.chatId, res => {
				console.log(res);
				if (res.success) {
					uni.showModal({
						title: '成功',
						showCancel: false
					});
				} else {
					uni.showModal({
						title: '失败',
						showCancel: false
					});
				}
			});
		},
		chooseImg() {
			this.$u.route({
				url: 'components/u-avatar-cropper/u-avatar-cropper',
				params: {
					destWidth: uni.upx2px(750),
					rectWidth: uni.upx2px(500),
					fileType: 'jpg'
				}
			});
		},
		swichShowNickName(status) {
			this.isShowNickName = status;
			let storeKey = 'isShowNickName' + this._user_info.id + '_' + this.chatObj.chatId;
			uni.setStorageSync(storeKey, this.isShowNickName);
		},
		getShowNickName() {
			let storeKey = 'isShowNickName' + this._user_info.id + '_' + this.chatObj.chatId;
			this.isShowNickName = uni.getStorageSync(storeKey)? uni.getStorageSync(storeKey): false;
		},
		queryGroupUser() {
			this.$socket.queryMembers(this.chatObj.chatId, this._user_info.id, res => {
				if (res.success) {
					this.members = res.members;
					this.$u.vuex('_membersNoneIndex',res.members)
					this.mine = res.groupUser;
					this.group = res.group;
					this.getShowNickName();
				} else {
					uni.showModal({
						title: '获取数据失败',
						showCancel: false
					});
				}
			});
			this.$socket.queryNotice(this._user_info.id, this.chatObj.chatId, res => {
				if (res.success) {
					this.context = res.context;
				} else {
					uni.showModal({
						title: '获取数据失败',
						showCancel: false
					});
				}
			});
		}
	},
	onLoad(option) {
	},
	onShow() {
	},
	onReady() {
		this.queryGroupUser();
	}
};
</script>

<style>
	.content .grid-text {
		text-align: center;
		width: 80rpx;
		height:40rpx;
		overflow: hidden;
	}
</style>
