<template>
	<view class="container">
		<cu-custom bgColor="bg-cyan" :isBack="true">
			<block slot="backText"></block>
			<block slot="content">入库审核详情页</block>
		</cu-custom>

		<view class="bg-top bg-cyan">
			<view class="top-box shadow">
				<view class="qh-pic cu-avatar xl"
					style="background-image:url(https://zhoukaiwen.com/img/kevinLogo.png)"></view>
				<view class="qh-title text-bold text-blue text-xl">
					<text>{{documentNumber}}</text>
				</view>

				<view class="cu-list menu">
					<view class="cu-item" style="padding: 0;">
						<view class="content">
							<text class="cuIcon-timefill text-blue  margin-right-xs"></text>
							<text class="text-black">出库时间</text>
						</view>
						<view class="action">
							<view class="cu-tag round bg-blue light">{{inWarehouseTime}}</view>
						</view>
					</view>

					<view class="cu-item" style="padding: 0;">
						<view class="content">
							<text class="cuIcon-myfill text-red"></text>
							<text class="text-black">入库类型</text>
						</view>
						<view class="action">
							<text class="text-grey text-sm">{{inWarehouseTypeName}}</text>
						</view>
					</view>

					<view class="bxBox" style="border-top: 1rpx solid #eee;">
						<text class="bxImg cuIcon-selectionfill text-pink"></text>
						<view class="title text-black">备注</view>
					</view>

					<view class='padding-bottom-sm padding-left-sm bg-white'>
						{{!!remark?remark:''}}
					</view>

				</view>

			</view>

			<!-- 物品列表 -->
			<view class="center-box shadow">
				<view class="cu-list menu">
					<view class="cu-bar bg-white margin-top-xs u-border-bottom">
						<view class="action sub-title">
							<text class="text-xl text-bold text-blue text-shadow">物品列表</text>
							<text class="text-ABC text-blue">LIST</text>
						</view>
					</view>
					<view style="min-height: 34vh">
						<zb-table :columns="column" :stripe="true" :border="true" :data="dataList"></zb-table>
					</view>

				</view>
			</view>

			<!-- 加入按钮 -->
			<view class="padding-lr flex margin-top-sm" style="justify-content: space-between;">
				<button class="cu-btn bg-blue margin-top-sm lg" style="width: 46%;" @click="rejectApprove()">拒绝申请</button>
				<button class="cu-btn bg-green margin-top-sm lg" style="width: 46%;"  @click="receiptApprove()">同意申请</button>
			</view>
		</view>
	</view>
</template>

<script>
	import request from '@/utils/request.js';
	export default {
		data() {
			return {
				receiptId: '',
				documentNumber: '',
				inWarehouseTime: '',
				inWarehouseTypeName: '',
				remark:"",
				warehouseName:"",
				column: [
					{
						name: 'goodsName',
						fixed: true,
						label: '物品名称',
						width: 80,
						align: "center"
					},
					{
						name: 'categoryName',
						label: '物品类别',
						align: "center"
					},{
						name: 'goodsPrice',
						label: '物品单价',
						align: "center"
					},{
						name: 'goodsNumber',
						label: '入库数量',
						align: "center"
					}
				],
				dataList: [],
			}
		},
		onLoad(option) {
			let params = JSON.parse(option.params)
			this.documentNumber = params.documentNumber;
			this.receiptId = params.receiptId;
			this.inWarehouseTypeName = params.inWarehouseTypeName;
			this._getReceiptDetail();
		},
		onShareAppMessage() {
			return {
				title: '分享'
			}
		},
		methods: {
			_getReceiptDetail() {
				var that = this;
					let opts = {
						url: 'warehouse/app/inReceiptApproveDetail/' + that.receiptId,
						method: 'get'
					};
					request.httpTokenRequest(opts).then(res => {
						if (res.code == 200) {
							that.documentNumber=res.data.documentNumber;
							that.inWarehouseTime=res.data.inWarehouseTime;
							that.warehouseName=res.data.warehouseName;
							that.inWarehouseTypeName=res.data.inWarehouseTypeName;
							that.remark=res.data.remark;
							that.dataList=res.data.receiptGoodsList;
						} else {
							uni.showToast({
						title: res.msg,
						icon: 'none'
					})
						}
					});
			},
			receiptApprove(){
				debugger
				var that = this;
					let opts = {
						url: 'warehouse/app/inReceiptApprove/'+that.receiptId,
						method:'get'
					};
					
					request.httpTokenRequest(opts).then(res => {
						if (res.code == 200) {
								uni.showToast({
									title:"审核通过",
									icon:'success',
									
								})
								setTimeout(function(){
									uni.navigateBack()
								},1000)
						} else {
							uni.showToast({
						title: res.msg,
						icon: 'none'
					})				}
					});
			},
			rejectApprove(){
				debugger
				var that = this;
					let opts = {
						url: 'warehouse/app/rejectReceiptApprove/'+that.receiptId,
						method:'get'
					};
					
					request.httpTokenRequest(opts).then(res => {
						if (res.code == 200) {
								uni.showToast({
									title:"已拒绝",
									icon:'success',
									
								})
								setTimeout(function(){
									uni.navigateBack()
								},1000)
						} else {
							uni.showToast({
						title: res.msg,
						icon: 'none'
					})				}
					});
			}

		}
	}
</script>
<style lang="scss" scoped>
	.container {
		width: 750rpx;
		color: #333333;

		.bg-top {
			margin-top: -1rpx;
			width: 750rpx;
			height: 220rpx;
			padding-top: 50rpx;
			border-radius: 0 0 20% 20%;

			.top-box {
				width: 700rpx;
				background-color: #FFFFFF;
				margin: 0 auto;
				border-radius: 20rpx;
				padding: 20rpx 30rpx 0rpx;
				position: relative;

				.qh-pic {
					position: absolute;
					right: 64rpx;
					top: -50rpx;
					border-radius: 12rpx;
				}

				.qh-title {
					width: 100%;
					height: 60rpx;
					line-height: 65rpx;
					padding-right: 190rpx;
				}

				.bxBox {
					position: relative;
					display: flex;
					/* padding: 0 30rpx; */
					min-height: 100rpx;
					/* background-color: #ffffff; */
					/* justify-content: space-between; */
					align-items: center;
					font-size: 30rpx;
					line-height: 1.6em;
					flex: 1;

					.bxImg {
						display: inline-block;
						margin-right: 10rpx;
						width: 1.6em;
						text-align: center;
					}
				}

			}
		}

		.center-box {
			color: #333333;
			width: 700rpx;
			background-color: #FFFFFF;
			margin: 0 auto;
			border-radius: 20rpx;
			padding: 0rpx 30rpx 0rpx;
			position: relative;
			margin-top: 20rpx;
		}
	}
</style>