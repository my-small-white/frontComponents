<template>
	<view>

		<u-navbar :is-back="true" title="选择城市" title-color="#000" back-icon-color="#000"></u-navbar>
		<view class="city-content" :style="'height:'+height+'px'">
			<view class="content-box">
				<view class="content-left">
					<view class="city-name" :class="{'acitvieColor':index==selectDefalutNum}"
						v-for="(item,index) in provices" :key="index">
						<text @click="changeCity(index)" :class="{'acitiveTxt':item.checked==true}">{{item.areaName}}</text>
					</view>
				</view>
				<view class="content-right">
					<u-checkbox v-model="isAllcheck" @change="selectall" ><text :class="{'acitvieColor':isAllcheck==true}">全选</text> </u-checkbox>
					<u-checkbox-group @change="checkboxGroupChange">
						
						<u-checkbox @change="checkboxChange($event,item,index)" v-model="item.checked"
							v-for="(item, index) in provices[selectDefalutNum].children" :key="index"
							:name="item.areaName"><text :class="{'acitvieColor':item.checked==true}">{{item.areaName}}</text></u-checkbox>
					</u-checkbox-group>
				</view>
			</view>
		</view>
	</view>

</template>

<script>
	let provices = require('./provice.json')
	const app = getApp();
	export default {
		data() {
			return {
				isAllcheck: false,
				cityList: [],
				selectVal: [],
				height: 0,
				provices: provices,
				selectDefalutNum: 0, //默认选择第一个
				navBarHeight: app.globalData.navBarHeight,
				menuHeight: app.globalData.menuHeight,
				menuBotton: app.globalData.menuBotton,
				menuTop: app.globalData.menuTop
			}
		},
		watch: {
		},
		methods: {
			selectall(e){
				/*同步问题 去反*/
				if (!this.isAllcheck) {
					
					//表示全选
					this.provices[this.selectDefalutNum].checked=true
					this.provices[this.selectDefalutNum].children.map((item) => {
						return item.checked = true
					})
					
				
				} else {
				
					this.provices[this.selectDefalutNum].checked=false
					//表示全不选
					this.provices[this.selectDefalutNum].children.map((item) => {
						return item.checked = false
					})
				}
				this.getAllselectVal()
			},
			changeCity(num) {
				this.isAllcheck=false
				this.selectDefalutNum = num
				for (let item of this.provices[this.selectDefalutNum].children) {
					
					if (!item.checked) {
						item.checked = false
					}
					if (!item.disabled) {
						item.disabled = false
					}

				}
					/*判断一下是否被选择*/
					this.checkSelectType()
			},
			// 选中某个复选框时，由checkbox时触发
			checkboxChange(e, val, num) {
				this.$set(this.provices[this.selectDefalutNum].children, num, val)
				
			},
			// 选中任一checkbox时，由checkbox-group触发
			checkboxGroupChange(e) {
				
				//当数组勾选全部时 全选选中
				this.checkSelectType()
				this.getAllselectVal()
				
			},
			checkSelectType(){
				
				
				if(this.provices[this.selectDefalutNum].children.length==0){
					this.provices[this.selectDefalutNum].checked=false
					this.$set(this.provices, this.selectDefalutNum, this.provices[this.selectDefalutNum])
						this.isAllcheck=false
					return
				}else{
					
					let data=this.provices[this.selectDefalutNum].children.filter((item)=>{
						if(item.checked){
							return item
						}
					})
					
					
					if(data.length==this.provices[this.selectDefalutNum].children.length){
					
						this.provices[this.selectDefalutNum].checked=true
						this.$set(this.provices, this.selectDefalutNum, this.provices[this.selectDefalutNum])
						this.isAllcheck=true
					}else if(data.length>0){
						
						this.provices[this.selectDefalutNum].checked=true
						this.$set(this.provices, this.selectDefalutNum, this.provices[this.selectDefalutNum])
					}else{
							
						this.provices[this.selectDefalutNum].checked=false
						this.$set(this.provices, this.selectDefalutNum, this.provices[this.selectDefalutNum])
						this.isAllcheck=false
					}
					
				}
				
		
			},
			//获取所有选中的值
			getAllselectVal(){
				let provicedata;
				let sendprovicedata=[];
				let citydata=[];
				 provicedata=this.provices.filter(item=>{
					if(item.checked){
						let { children, ...params } = item
						sendprovicedata.push(params)
						return item
						
					}
				})
				for(let items of provicedata){
					items.children.filter(item=>{
						if(item.checked){
							//去除掉三级 
							let { children, ...params } = item
							citydata.push(params)
						}
					})
				}
				//传递数据给搜索页
				let datas={
					"provice":sendprovicedata,
					"city":citydata
				}
				
					this.$store.commit('ADD_SELECTCITY', JSON.stringify(datas))
					uni.setStorageSync('ADD_SELECTCITY', JSON.stringify(datas))
				
				
			},
			dealData(data){
				if(data.provice.length>0 || data.city.length>0){
									
					for(let item of this.provices){
						for(let m of data.provice ){
							if(item.areaCode==m.areaCode){
								item.checked=m.checked
								for(let k of item.children){
									for(let v of data.city){
										if(v.areaCode==k.areaCode){
											k.checked=v.checked
										}
									}
								}
							}
						}
					}
				}
			},
			defalutData(){
				for (let item of this.provices[this.selectDefalutNum].children) {
					item.checked = false
					item.disabled = false
				}
			}
		},
	
		onLoad() {
			let that = this;
			// 获取系统信息
			wx.getSystemInfo({
				success: function(res) {
					// 获取可使用窗口宽度
					let clientHeight = res.windowHeight;
					// 设置高度
					that.setData({
						height: clientHeight - (that.navBarHeight + 8)
					});
				}
			})
		},
		created() {
			/*获取stroe数据  回显*/
			for(let val of this.provices){
				val.checked=false
			}
					if(uni.getStorageSync("ADD_SELECTCITY")){
						this.dealData(JSON.parse(uni.getStorageSync("ADD_SELECTCITY")))
						//检查默认的是否全选了
						this.checkSelectType()
					}else{
						this.defalutData()
					}	
		},
		mounted() {


		}
	}
</script>

<style scoped>
	.city-content {
		width: 100%;
		background: white;
		overflow: hidden;


	}

	.content-box {
		width: 100%;
		height: 100%;
		overflow: hidden;
		display: flex;
		justify-content: space-between;


	}

	.content-left {

		width: 280rpx;
		height: auto;
		background: #F2F2F2;
		overflow-y: scroll;
	}

	.content-right {
		width: 470rpx;
		height: auto;
		overflow-y: scroll;

	}

	.city-name {
		width: 100%;
		text-align: center;
		height: 96rpx;
		line-height: 96rpx;
		color: #595959;
		font-size: 28rpx;
	}

	.city-name text {
		display: block;
		width: 100%;
		height: 100%;
		text-align: center;
	}

	.acitvieColor {
		background: white;
		color: #0B67F1;
	}

	.acitvieColor text {
		color: #0B67F1;
	}
	.acitiveTxt{
		color: #0B67F1;
	}

	.city-name-select text {
		width: auto;
	}

	.city-name-select {
		display: flex;
		justify-content: space-around;
	}

	/deep/ .u-checkbox {
		display: inline-block !important;
		width: 100% !important;
		height: 96rpx;
		line-height: 96rpx;
		justify-content: space-evenly;

	}

	/deep/.u-checkbox__label {
		float: left;
		height: 96rpx;
		line-height: 96rpx;
		margin-left: 40rpx !important;
	}

	/deep/.u-checkbox__icon-wrap {
		float: right;
		height: 96rpx;
		line-height: 96rpx;
		margin-right: 30rpx;
		margin-top: 31rpx;
	}
</style>
