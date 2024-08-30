<template>
	<view class="container">
		<cu-custom bgColor="bg-cyan" :isBack="true">
			<block slot="backText"></block>
			<block slot="content">出库扫码详情页</block>
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
							<text class="cuIcon-homefill text-green"></text>
							<text class="text-black">仓库名称</text>
						</view>
						<view class="action">
							<view class="cu-tag round bg-green light">{{warehouseName}}</view>
						</view>
					</view>

					<view class="cu-item" style="padding: 0;">
						<view class="content">
							<text class="cuIcon-timefill text-blue  margin-right-xs"></text>
							<text class="text-black">申请时间</text>
						</view>
						<view class="action">
							<view class="cu-tag round bg-blue light">{{outWarehouseTime}}</view>
						</view>
					</view>

					<view class="cu-item" style="padding: 0;">
						<view class="content">
							<text class="cuIcon-myfill text-red"></text>
							<text class="text-black">出库类型</text>
						</view>
						<view class="action">
							<text class="text-grey text-sm">{{outWarehouseTypeName}}</text>
						</view>
					</view>

					<view class="bxBox" style="border-top: 1rpx solid #eee;">
						<text class="bxImg cuIcon-selectionfill text-pink"></text>
						<view class="title text-black">备注</view>
					</view>

					<view class='padding-bottom-sm padding-left-sm bg-white'>
						{{remark}}
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
						<scroll-view style="height: 300px;" scroll-y="true" refresher-enabled="true" 
						            :refresher-threshold="100" refresher-background="lightgreen" @refresherpulling="onPulling"
						            @refresherrefresh="onRefresh" @refresherrestore="onRestore" @refresherabort="onAbort">
									<uni-table ref="table"  border stripe emptyText="暂无更多数据" style="display: static;overflow: visible;">
										<uni-tr>
											<uni-th width="50" align="center">物品名称</uni-th>
											<uni-th width="240" align="center">确认出库</uni-th>
										</uni-tr>
										<uni-tr v-for="(item, index) in dataList" :key="index">
											<uni-td align="center">{{ item.goodsName }}</uni-td>
											<uni-td align="center" v-if="!item.snCode==''">{{item.snCode}}</uni-td>
											<uni-td align="center" v-else >
												<uni-data-select
												  :clear="false"
												  v-model="item.snCode"
												  :localdata="goodScan"
												  @change="change(item)"
												></uni-data-select>
												<!-- <button size="mini" type="primary" @click="changeInGoodsList(item)">确认</button> -->
											</uni-td>
										</uni-tr>
									</uni-table>
									</scroll-view>
								</view>

				</view>
			</view>

			<!-- 加入按钮 -->
			<view class="padding-lr flex margin-top-sm" style="justify-content: space-between;">
				<button class="cu-btn bg-blue margin-top-sm lg" style="width: 46%;" @click="godetails()">物品扫码</button>
				<button class="cu-btn bg-green margin-top-sm lg" style="width: 46%;"
					@click="outReceiptScanSave()">确认完成</button>
			</view>

			<view class="padding-lr" style="margin-top: 15rpx; margin-bottom: 30rpx;">
				<text class="text-grey text-sm">"扫码" 即可对物品进行扫码</text>
			</view>

		</view>
	</view>
</template>

<script>
	import request from '@/utils/request.js';
	export default {
		data() {
			return {
				documentNumber: '',
				outWarehouseTime: '',
				remark: '',
				outWarehouseTypeName: '',
				warehouseOutReceiptId: '',
				warehouseName: "",
				warehouseId: 0,
				column: [{
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
					}, {
						name: 'snValidate',
						label: '是否需要扫码',
						filters: {
							'Y': '需扫描',
							'N': '无需'
						},
						align: "center"
					}, {
						name: 'scanSuccess',
						label: '扫码状态',
						filters: {
							true: '已扫码',
							'': '未扫码'
						},
						align: "center"
					},
				],
				noScanList: [],
				dataList: [],
				goodScan: [],
			}
		},
		onShow() {
		},
		onLoad(option) {
			let that=this;
			let params = JSON.parse(option.params)
			this.documentNumber = params.documentNumber;
			this.outWarehouseTime = params.outWarehouseTime;
			this.remark = params.remark;
			this.outWarehouseTypeName = params.outWarehouseTypeName;
			this.warehouseOutReceiptId = params.warehouseOutReceiptId;
			this._getOutReceiptScanDetail();
			// this._getGoodScan()
			uni.$on('scanList', res => {
				that.dataList = res
			})
		},
		onUnload() {
			uni.$off("scanList")
		},
		methods: {
			_getGoodScan() {
				var that = this;
				let opts = {
					url: 'warehouse/app/getGoodScan/' + that.warehouseOutReceiptId+'?warehouseId='+that.warehouseId,
					method: 'get'
				};
				request.httpTokenRequest(opts).then(res => {
					if (res.code == 200) {
						res.data.forEach(item => {
							let data = { value: item, text: item ,disable:false}
							that.goodScan.push(data)
						})
						console.log(that.goodScan);
					} else {
						uni.showToast({
						title: res.msg,
						icon: 'none'
					})
					}
				});
			},
			_getOutReceiptScanDetail() {
				var that = this;
				let opts = {
					url: 'warehouse/app/outReceiptScanDetail/' + that.warehouseOutReceiptId,
					method: 'get'
				};
				request.httpTokenRequest(opts).then(res => {
					if (res.code == 200) {
						console.log(res.data)
						that.warehouseId = res.data.warehouseId
						that.warehouseName = res.data.warehouseName;
						that.dataList = res.data.goodsVoList;
						this._getGoodScan()
					} else {
						uni.showToast({
						title: res.msg,
						icon: 'none'
					})
					}
				});
			},
			godetails() {
				uni.showToast({
					title: '暂未开放',
					icon: 'none'
				});
				// let that = this;
				// if (that.dataList.length <= 0) {
				// 	that.mes("无‘待扫码物品’")
				// 	return
				// } else {
				// 	let snValidate = false
				// 	that.dataList.forEach(item => {
				// 		if (item.snValidate) snValidate = true
				// 	})
				// 	if (!snValidate) {
				// 		that.mes("无‘待扫码物品’")
				// 		return
				// 	}

				// }
				// uni.setStorageSync('scanList', JSON.stringify(that.dataList))
				// that.Godetails("/pages/workspace/barcode/index",{
				// 	fromPage: 'out'
				// });
			},
			//入库扫码时 改变物品状态
			changeInGoodsList(item) {
				item.scanSuccess = true
			},
			change(e){
				console.log("delete");
				console.log(this.dataList);
				this.goodScan.forEach(item => {
					 if (item.value === e.snCode){
					    item.disable = true
						e.scanSuccess = true
					 }
				})
				// this.goodScan[0].disable=true
			},
			outReceiptScanSave() {
				var that = this;
				let num = 0;
				let shouldNum = 0
				that.dataList.forEach(item => {
					if (item.scanSuccess) num++
					if (item.snValidate) shouldNum++
				})
				if (shouldNum > num) {
					uni.showToast({
						title: "存在部分‘须扫码’物品未进行扫码",
						icon: 'none',
					})
					return;
				}

				let opts = {
					url: 'warehouse/app/outReceiptScanSave',
				};
				let params = {
					warehouseOutReceiptId:that.warehouseOutReceiptId,
					goodsVoList: that.dataList
				}
				request.httpTokenRequest(opts, params).then(res => {
					if (res.code == 200) {
						uni.showToast({
							title: "提交成功",
							icon: 'success',
						})
						setTimeout(function(){
							uni.navigateBack()
						},1000)
					} else {
						uni.showToast({
						title: res.msg,
						icon: 'none'
					})			}
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