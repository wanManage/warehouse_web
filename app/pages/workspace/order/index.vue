<template>
	<view>
		<cu-custom bgColor="bg-cyan" :isBack="true">
			<block slot="backText">返回</block>
			<block slot="content">下单要货</block>
		</cu-custom>
		<view class="margin-top">
			<form>
				<view class="cu-form-group">
					<view class="title">要货日期</view>
					<picker mode="date" :value="form.applyTime" start="2023-09-01" end="2050-12-31"
						@change="DateChange">
						<view class="picker">
							{{form.applyTime}}
						</view>
					</picker>
				</view>

				<view class="cu-form-group" @click=select_Warehouse(0)>
					<view class="title">要货仓库</view>
					<view class="">
						<view v-if="!!form.applyWarehouseName">{{form.applyWarehouseName}}</view>
						<text v-else class="cuIcon-right text-gray text-lg"></text>
					</view>
				</view>
				<view class="cu-form-group" @click=select_Warehouse(1)>
					<view class="title">入库仓库</view>
					<view class="">
						<view v-if="!!form.applyInWarehouseName">{{form.applyInWarehouseName}}</view>
						<text v-else class="cuIcon-right text-gray text-lg"></text>
					</view>
				</view>
				<view class="cu-form-group">
					<textarea maxlength="250" @input="textareaAInput" placeholder="可填写备注"></textarea>
				</view>
			</form>
		</view>
		<view class="margin-top bg-white flex justify-center">
			<button style="width: 60%;" class="cu-btn block bg-blue margin-tb-sm lg"
				@click="goGoods()">
				<text class="cuIcon-goodsfill"></text> 手动选择商品</button>
		</view>
		<!-- 仓库选择 -->
		<view class="cu-modal drawer-modal justify-start" :class="personShow?'show':''" @tap="personShow=false">
			<view class="pop-con cu-dialog basis-lg" @tap.stop=""
				:style="[{top:CustomBar+'px',height:'calc(100vh - ' + CustomBar + 'px)'}]">
				<view class="top">
					<view class="pop-title">仓库选择</view>
					<view class="search-box">
						<u-search v-model="keyword" @custom="actionClick" @focus="focus" :show-action="showAction"
							action-text="搜索" @change="change" @clear="clearClick"></u-search>
					</view>
				</view>

				<view style="margin-top: 180rpx;height: calc(100% - 180rpx);overflow-y: scroll;">
					<view class="magin-top">
						<DaTreeVue2 v-if="warehouse_type==0" ref="DaTreeRef" :data="treeData" labelField="warehouseName" valueField="warehouseId"
							expandChecked onlyRadioLeaf :showRadioIcon="false" @change="handleTreeChange"></DaTreeVue2>
						<DaTreeVue2 v-else ref="DaTreeRef" :data="myTreeData" labelField="warehouseName" valueField="warehouseId"
								expandChecked onlyRadioLeaf :showRadioIcon="false" @change="handleTreeChange"></DaTreeVue2>
					</view>
					<view style="height: 25px;"></view>
				</view>
			</view>

		</view>

		<view class="magin-top bg-white">
			<zb-table :columns="column" :stripe="true" :border="true" :data="form.receiptGoodsList"></zb-table>
		</view>

		<view v-show="form.receiptGoodsList.length>0" class="margin-top  flex justify-center">
			<button style="width: 60%;" class="cu-btn round block bg-green margin-tb-sm lg" @click="submitOrder()">
				<text class="cuIcon-check"></text> 确认提交</button>
		</view>

	</view>
</template>

<script>
	import {
		getUserInfo
	} from '@/utils/auth';
	import request from '@/utils/request.js';
	import DaTreeVue2 from '@/components/da-tree-vue2/index.vue'
	export default {
		components: {
			DaTreeVue2
		},
		data() {
			return {
				CustomBar: this.CustomBar,
				personShow: false,
				keyword: "",
				dataList: [],
				showAction: false,
				allTreeData: [],
				treeData: [],
				myTreeData: [],
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
						name: 'goodsNumber',
						label: '要货数量',
						align: "center"
					}
				],
				form: {
					applyTime: "",
					applyType: "1", //	要货类型,1向公司要货，2向个人要货	
					applyUserId: getUserInfo().userId, //	要货人ID	
					applyUserName: getUserInfo().nickName,
					applyWarehouseId: "", //要货仓库	
					applyWarehouseName: "",
					documentNumber: "", //要货单号
					receiptGoodsList: [], //要货物品明细	物品信息
					remark: "",
					requestedUserId: "",
					requestedUserName: "",
					applyInWarehouseId: 0,
					applyInWarehouseName:"",
				},
				warehouse_type: 0

			}
		},
		onLoad() {
			let that = this;
			uni.$on('select', res => {
				that.form.receiptGoodsList = res
			})
			this.init();
			this.applyTypeChange();
			//生成要货单
			this.getApplyCode();
		},
		onUnload() {
			uni.$off("select")
		},
		methods: {
			init() {
				this.form.applyTime = this.$u.timeFormat()
			},
			getApplyCode() {
				var that = this;
				let opts = {
					url: 'warehouse/app/applyCode',
					method: 'get',
				};
				request.httpTokenRequest(opts).then(res => {
					if (res.code == 200) {
						that.form.documentNumber = res.msg;
					} else {
						uni.showToast({
						title: res.msg,
						icon: 'none'
					})
					}
				});
			},
			DateChange(e) {
				this.form.applyTime = e.detail.value
			},
			applyTypeChange() {
				// this.getWarehouse("applyTypeChange")
				if (this.treeData.length <= 0) this.getWarehouse();
				// this.form.applyType = e.detail.value;
				// if (this.form.applyType == "1") {
				// 	this.form.requestedUserId = "";
				// 	this.form.requestedUserName = "";
				// } else {
				// 	//获取人员
				// 	if (this.treeData.length <= 0) this.getWarehouse();
				// }
			},
			select_Warehouse(type){
				this.warehouse_type = type
				this.personShow =true
			},
			getWarehouse() {
				console.log("getWarehouse");
				var that = this;
				let opts = {
					url: 'warehouse/app/warehouseList',
				};
				
				let params = {}

				request.httpTokenRequest(opts, params).then(res => {
					if (res.code == 200) {
						console.log(res.rows);
						let depId = getUserInfo().deptId
						for (var k = 0, length = res.rows.length; k < length; k++) {
							let newgoods = res.rows[k];
							if(newgoods.deptId == depId){
								this.myTreeData.push(newgoods)
							}else{
								this.treeData.push(newgoods)
							}
						}
						console.log(this.myTreeData);
						
						// that.treeData = res.rows;
						that.allTreeData = JSON.parse(JSON.stringify(res.rows));
					} else {
						uni.showToast({
						title: res.msg,
						icon: 'none'
					})
					}
				});
			},
			// getUser() {
			// 	var that = this;
			// 	let opts = {
			// 		url: 'warehouse/app/fixedUser/list',
			// 		method: 'get',
			// 	};
				
			// 	request.httpTokenRequest(opts).then(res => {
			// 		if (res.code == 200) {
			// 			that.treeData = res.rows;
			// 			that.allTreeData = JSON.parse(JSON.stringify(res.rows));
			// 		} else {
			// 			uni.showToast({
			// 			title: res.msg,
			// 			icon: 'none'
			// 		})			}
			// 	});
			// },
			submitOrder() {
				let that = this;
				
			
				uni.showModal({
					title: "提示",
					content: "是否确认提交",
					success: function(res) {
						if (res.confirm) {
							that.submit()
						}
					}
				})
			},
			submit() {
				var that = this;
				let opts = {
					url: 'warehouse/app/applyReceipt',
				};
				for (let i = 0; i < this.form.receiptGoodsList.length; i++) {
					const element = this.form.receiptGoodsList[i];
					element.goodsAmount = (element.goodsNumber * element.goodsPrice).toFixed(2);
				}

				that.form.applyTime = that.form.applyTime + " 00:00:00"
				request.httpTokenRequest(opts, that.form).then(res => {
					if (res.code == 200) {
						that.mes("提交成功")
						setTimeout(function() {
							uni.navigateBack()
						}, 1000)
					} else {
						that.muni.showToast({
						title: res.msg,
						icon: 'none'
					})}
				});
			},
			textareaAInput(e) {
				this.form.remark = e.detail.value
			},
			focus() {
				this.showAction = true;
			},
			clearClick() {
				this.showAction = false;
				this.keyword = "";
				this.treeData = JSON.parse(JSON.stringify(this.allTreeData));
			},
			actionClick() {
				this.change();
			},
			change() {
				let that = this
				this.treeData = JSON.parse(JSON.stringify(this.allTreeData.filter(item => {
					return item.warehouseName.indexOf(that.keyword) >= 0
				})));
			},
			handleTreeChange(values, currentItem) {
				console.log("handleTreeChange")
				console.log(values)
				console.log(currentItem)
				if (this.warehouse_type ==0){
					//数据赋值
					this.form.applyWarehouseId = values;
					this.form.applyWarehouseName = currentItem.label;
				}else{
					this.form.applyInWarehouseId = values
					this.form.applyInWarehouseName = currentItem.label;
				}
				//关闭弹框
				this.personShow = false;
			},
			goGoods(){
				let params = {
					"warehouseIds": this.form.applyWarehouseId 
				}
				this.Godetails('/pages/workspace/order/goods/index',params);
			}


		}
	}
</script>


<style lang="scss" scoped>
	.pop-con {
		.top {
			position: fixed;
			top: 0;
			width: 100%;
			left: 0;
			z-index: 2;
			background-color: #FFf;

			.pop-title {
				text-align: center;
				height: 96rpx;
				line-height: 96rpx;
				background: #F8F8F8;
			}

			.search-box {
				padding: 12rpx;
			}
		}

		.bot-btn {
			background: #FFFFFF;
			position: fixed;
			text-align: center;
			z-index: 9;
			border-top: 1px solid #f7f7f7;
			width: 95%;
			bottom: 0;
			padding: 12rpx;
		}
	}
</style>