<template>
	<view class="wrap shopOreder_wrap">
		<view class="status_bar index_status_bar"></view>
		<customnav
		 :ismsg="false" 
		 :isSearch="false" 
		 :midtitle="true"
		 navtitle="订单管理"
		 :cancletext="false"
		 :focus="false"
		 :backType="1"
		 backurl="/pages/personal/shopManage"
		 />
		 <view class="shoplist_content">
		 		<view class="shoplist_content_item" >
		 			<view class="shoplist_content_item_top">
		 				<view class="top_left">
		 					
		 						<image v-if="shopInfo.head_sculpture" class="top_img" :src="domain2+shopInfo .head_sculpture" mode="aspectFill"></image>
		 						<image v-else class="top_img" src="../../static/images/default-avatar.jpeg" mode="aspectFill"></image>
		 						
		 						<view class="top_info">
		 							<span>{{shopInfo.shopname}}</span>
		 							<span @click.stop="callsupplier(shopInfo.phone)"><i class="iconfont icon-dianhua"></i>{{shopInfo.phone}}</span>
		 						</view>
		 				</view>
		 				<view class="top_right_type">
		 					<span @click="levelUp">申请等级</span>
		 				</view>
		 			</view>
		 			<view class="shoplist_content_item_bottom">
		 				<i class="iconfont icon-weizhi"></i>
		 				<p>{{shopInfo.address}}</p>
		 			</view>
		 		</view>
		 </view>
		 <qstab
			 :tabs="tabs" 
			 @change="handleChange"
			 :current="currentidx"
			 animationMode="line1"
			 activeColor="#147AED"	
			 fontSize="30"
			 backgroundColor="#fff"
			 :width="tabwidth"
			 height="100"
			 animationLineWidth="70"
		 >
		 </qstab>
			 <scroll-view 
			 scroll-y="true" 
			 class="order_list" 
			 @scroll="scroll" 
			 @scrolltolower="handlescroll" 
			 :scroll-top="setMarkTop"
			 >
		 		 <view class="order_box" v-for="(item,idx) of orderList" :key="idx">
					 <!-- 订单号 -->
					 <view class="ordernum">
						 <view class="ordernum_left">
							 <span>订单编号:{{item.orderno}}</span>
							
						 </view>	
						 <span> 
							<i v-if="item.order_role==2" class="iconfont icon-daifu"></i>
							{{orderStatus[item.order_status-1]}}
						 </span>
					 </view>
					 <!-- 商品详情 -->
					 <view class="order_item" :class="{'disableGoods':goods.status == 3 }" @click="orderDetail(item.id,item.order_status)" v-for="(goods,gidx) of item.goods" :key="gidx">
						 <view class="o-left">
							<!-- <image class="order_img" src="../../static/images/activity.jpeg" mode="aspectFill"></image> -->
							 <image class=" order_img image" :class="{lazy:!goods.show}"  :data-index="goods.goodsIndex" @load="imageLoad" :src="goods.show ? goods.img:''" />
							 <view class="image placeholder loadimg" :class="{loaded:goods.loaded}" ><i class="iconfont icon-image"></i></view>	
						 </view>
						
						 <view class="omid">
							 <p>{{goods.name}}</p>
							 <span v-if="goods.status == 2">规格:{{goods.format_spec}}</span>
							 <span v-else-if="goods.store == 0">已售罄</span>
							 <span v-else>已下架</span>
						 </view>
						 <view class="o-right">
							 <span>￥{{goods.price}}</span>
							 <view class="or-bottom">
								 <span>× {{goods.quantity}}</span>
								 <span v-if="parseFloat(goods.coupon_fee)  != 0 ">优惠：{{goods.coupon_fee}}</span>
								 <span style="margin-top:5px">共{{item.number}}件</span>
							 </view>
							 	
						 </view>
					 </view>
					 
					 <!-- 订单总价 -->
					 <view class="total_box">
						 <view class="total_one">
								<view class="to_left">
									
									<span class="sh_btn" v-if="item.showShouhou">申请售后</span>
									
									<span class="qrcode_btn" @click="payment(item.total_no,idx,item.many,item.orderno)" v-if="item.showQrcode">生成二维码</span>
									 <span class="otype">{{item.order_type == 1 ? '账期订单' : (item.order_type == 2 ? '到付订单' : '现付订单')}}</span>
								</view>
								<view class="to_right">
								
									<span>运费:￥{{item.freight}}</span>
									<span>应付:￥{{item.cope_with}}</span>
								</view>
						 </view>
						<view class="total_two">
							<span>下单时间:{{item.create_time}}</span>
							<span>实付款:￥{{item.fact_price}}</span>
						</view>
					 </view>
					
				 </view>
				  <view class="loadfinshed_text" v-if="finshed">没有更多商品了</view>
				 
		 </scroll-view>
		
		 <level ref="levelRef"/>
		 <backTop :scrollTop="topval" @backTop="backTop" />
		<view  class="amask" :class="{'showMask' : showqrcode}" ></view>
		<view class="qr"  v-if="ifShow" :class="[isali ? 'aliBkg' : 'wechatBkg']">
			<i class="iconfont icon-ziyuan closeQrcode" @click="handleCloseQr"></i>
			<span><i class="iconfont icon-saoma"></i> 请使用{{typeText}}扫码支付</span>
			<view class="qrbkg" >
				<tki-qrcode
					class="qrcodeBox"
					cid="qrcode1" 
					ref="qrcode" 
					:val="codeUrl" 
					:size="size"
					:unit="unit" 
					:icon="icon" 
					:iconSize="iconsize"
					:onval="onval" 
					:usingComponents="true" />
			</view>
			
		</view>
			 <pageLoad :hide="hide" />	
		
	</view>
</template>

<script>
	// 顶部导航
	import customnav from '@/components/customnav.vue'
	import qstab from '@/components/QS-tabs/QS-tabs.vue'
	import level from '@/components/level.vue'
	import popup from '@/components/uni-popup/uni-popup.vue'
	import tkiQrcode from "@/components/tki-qrcode/tki-qrcode.vue"
	
	export default {
		components:{customnav,qstab,level,popup,tkiQrcode},
		data() {
			return {
				hide:false,
				tabs:['全部','待付款','待发货','待收货'],
				tabwidth:0,
				currentidx:0,
				shopInfo:{},
				orderList:[],
				orderGoodsList:[],
				domain2:this.$store.state.domain2,
				domain:this.$store.state.domain,
				shopId:'',
				page:1,
				finshed:false,
				type:'',
				
				goodsIndex:-1,                  
				orderStatus:['待付款','待发货','待收货','已完成订单','交易关闭'],//订单类型：1 账期订单 2 到付订单 3 现付订单
				//图片懒加载
				windowHeight: 0,
				show: false,
				 //图片懒加载
				 setMarkTop:-1,
				 marktop1:0,
				 marktop2:0,
				 marktop3:0,
				 marktop4:0,
				 topval:0,
				 codeUrl:'',
				 ifShow: false,
				 size: 200, // 二维码大小
				 unit: 'px', // 单位
				 icon: '', // 二维码图标
				 iconsize: 40, // 二维码图标大小
				 onval: false, // val值变化时自动重新生成二维码
				 showqrcode:false,
				 matchStatus:-1,
				 matchStatus2:-1,
				 matchTimer:null,
				 typeText:'',
				 isali:false
			};
		},
		computed:{
			oldtype(){
				return this.$store.state.oldtype
			}
		},
		onLoad(option){
			//tabs宽度
			this.tabwidth = uni.getSystemInfoSync().windowWidth / 4;
			//图片懒加载
			this.windowHeight = uni.getSystemInfoSync().windowHeight
			this.shopId = option.shopId
			this.getOrderList()
			
		},
		onShow() {
			this.goodsIndex=-1
			this.type = this.oldtype || ''
			this.orderGoodsList=[]
			this.orderList = []
			this.show=false
			this.page=1
			this.getOrderList()
		},
		methods:{
			callsupplier(phonenum){
					uni.makePhoneCall({
					    phoneNumber: phonenum //仅为示例
					});
			},
			handleCloseQr(){
				this.showqrcode=false
				this.ifShow=false
				clearInterval(this.matchTimer)
			},
			payment(totalNum,idx,payType,orderno){
				if(payType == 1){ //多订单
					uni.showToast({
						icon:'none',
						title:'非业务员下单请到小店支付',
						duration:2000
					})
					return
				}
				this.$store.dispatch('paymentType',totalNum).then(opt=>{
					this.codeUrl = opt.info
					if(opt.idx == 0){
						this.typeText='支付宝'
						this.isali=true
					}else{
						this.typeText='微信'
						this.isali=false
					}
					this.showqrcode=true
					this.ifShow=true
					setTimeout(()=>{
						this.$refs.qrcode._makeCode()
					},500)
					
					// this.matchStatus = this.orderList[idx].order_status
				this.type=''
					this.matchTimer=setInterval(()=>{
						
						this.getOrderList(true,orderno)
					},2000)
				})
			},
			backTop(){
				this.setMarkTop = 0
				setTimeout(()=>{
					this.setMarkTop = -1
				},500)
			},	
			
			//图片懒加载
			scroll(e){
				this.topval = e.target.scrollTop
				this.load()
				//记录上次滚动位置，tab切换时回到上次位置
				switch(this.currentidx+1){
					case 1:this.marktop1 = e.detail.scrollTop
					break;
					case 2:this.marktop2 = e.detail.scrollTop
					break;
					case 3:this.marktop3 = e.detail.scrollTop
					break;
					case 4:this.marktop4 = e.detail.scrollTop
					break;
				}
				
			},
			load() {
				const query = uni.createSelectorQuery().in(this)
				query.selectAll('.lazy').boundingClientRect((images) => {
					
					images.forEach((image, index) => {
						if (image.top <= this.windowHeight) {
							this.orderGoodsList[image.dataset.index].show = true;
						}
					})
				}).exec()
			},
			imageLoad(e) {
				
				this.orderGoodsList[e.target.dataset.index].loaded = true
				if(this.currentidx == 0){
					this.setMarkTop = this.marktop1
				}
				//记录上次滚动位置，tab切换时回到上次位置
				switch(this.currentidx+1){
					case 1:this.setMarkTop = this.marktop1
					break;
					case 2:this.setMarkTop = this.marktop2
					break;
					case 3:this.setMarkTop = this.marktop3
					break;
					case 4:this.setMarkTop = this.marktop4
					break;
				}
				
			},
			handleChange(idx){
				
				this.currentidx = idx
				this.goodsIndex=-1
				this.type = idx
				//保存点击的状态
				this.$store.commit('SAVE_STATUS',idx)
				this.orderGoodsList=[]
				this.orderList = []
				this.show=false
				this.page=1
				this.getOrderList()
				
				
			},
			//滚动加载更多
			handlescroll(e){
					 if(this.finshed) return 
					this.page++
					this.getOrderList()
							
			},
			//查看订单详情
			orderDetail(orderId,status){
				
				uni.navigateTo({
					url:'/pages/personal/shopOrderDetail?orderId='+orderId+'&shopId='+this.shopId+'&orderStatus='+status
				})
			},
			//申请等级
			levelUp(shopId){
				this.$refs.levelRef.showLevel(shopId)
			},
			getOrderList(match,orderno){
				let _this = this
				this.$dyrequest({
					url:'/SmallShop/ordermanage',
					method:'POST',
					hideLoading:true,
					data:{
						id:this.shopId,
						page:this.page,
						order_status:this.type
					}
					
				}).then(res=>{
					this.shopInfo = res.data.data.shopMsg
					
					if(match){
							
						res.data.data.orderList.map(item=>{
							
							if(item.orderno == orderno){
								this.matchStatus = item
							
							}
						})
						
						if(this.matchStatus.order_type == 1 || this.matchStatus.order_type == 2){
								if(this.matchStatus.order_status == 4){
									clearInterval(this.matchTimer)
									this.showqrcode=false
									this.ifShow=false
									uni.showToast({
										icon:'none',
										title:'支付成功'
									})
									setTimeout(()=>{
										this.type=this.oldtype
										this.orderList=[]
										this.getOrderList()
									},2000)
								}
						}else{
								if(this.matchStatus.order_status == 2){
									clearInterval(this.matchTimer)
									this.showqrcode=false
									this.ifShow=false
									uni.showToast({
										icon:'none',
										title:'支付成功'
									})
									setTimeout(()=>{
									
									this.type=this.oldtype
									this.orderList=[]
										this.getOrderList()
									},2000)
								}
						}
						
					}else{
						
						res.data.data.orderList.map((item,idx)=>{
								 _this.$set(item,'showQrcode',false)
								 _this.$set(item,'showShouhou',false)
								item.order_status = Number(item.order_status)
								
								//订单状态为待收货显示售后按钮
								// if(item.order_status == 3){
								// 	 _this.$set(item,'showShouhou',true)
								// }
								
								//订单状态为待收货且订单类型为到付或账期是显示付款二维码按钮
								if(item.order_status == 3 && item.order_type == 1){
									 _this.$set(item,'showQrcode',true)
								}
								if(item.order_status == 3 && item.order_type == 2){
									 _this.$set(item,'showQrcode',true)
								}
								//订单状态为待付款且订单类型为现付显示付款二维码按钮
								if(item.order_status == 1 && item.order_type == 3){
									 _this.$set(item,'showQrcode',true)
								}
								//处理商品信息
								item.goods.map((gitem,idx)=>{
									_this.goodsIndex+=1
										 _this.$set(gitem,'show',false)
										 _this.$set(gitem,'loaded',false)
										 _this.$set(gitem,'format_spec',false)
										 _this.$set(gitem,'goodsIndex',_this.goodsIndex)
										 if(gitem.status == 3){
											 _this.$set(item,'showQrcode',false)
											 _this.$set(item,'showShouhou',false)
										 }
										// console.log(_this.goodsIndex)
										 //格式化规格
										 if(gitem.price_type == 'whole'){//整件价
											 if(gitem.pack_type == 1){//1，整件 2 ，散装
												 gitem.format_spec= gitem.net_weight+'×'+ gitem.spec+'/'+gitem.whole_unit
											 }else{
												  gitem.format_spec= gitem.whole_spec+gitem.bulk_unit
											 }
										 }else{//单价
											 if(gitem.pack_type == 1){
											 		gitem.format_spec= gitem.net_weight+'/'+gitem.retail_unit
											 }else{
											 		gitem.format_spec= gitem.retail_spec+gitem.bulk_unit
											 }
										 }
										 
										 gitem.img = _this.domain + gitem.img
										 //存储商品列表做懒加载
										 _this.orderGoodsList.push(gitem)
										 //待付款列表
										 
																 
								})
									_this.orderList.push(item)
								
						})
					}
					
					//数据加载完成后隐藏loading
					setTimeout(()=>{
						this.hide = true
					},1000)
						// console.log(_this.orderList)
						
						
					//图片懒加载
					
					
					if (!this.show) {
						this.show = true
						
						setTimeout(() => {
							this.load()
						}, 100)
					}
					//图片懒加载
					//判断没有数据，不再请求
					if(res.data.data.orderList.length == 0){
							this.finshed = true
					}
						//console.log( _this.orderList)
				})
				.catch(err=>{
						clearInterval(this.matchTimer)
					console.log('支付错误，请重新生成支付二维码')
				})
			}
		}
	}
</script>

<style lang="scss">
	@import '@/static/css/style.scss';
.order_list{
	width:100%;
	 height:calc(100vh - 199px - var(--status-bar-height)); 
	//height:541px;
}

//订单列表
.uni-popup__wrapper-box{
		width:80%;
	}
.order_box{
	background:#fff;
	margin-bottom:10px;
	padding:10px 10px 0;
	.ordernum{
		display:flex;
		justify-content: space-between;
		padding:10px 0;
		&_left{
			display: flex;
			align-items: center;
			color:#999;
			
		}
		span{
			display: flex;
			align-items: center;
			color:#E5640D;
			i{
				color:#147AED;
				margin-right:5px;
			}
		}
	}
	.order_item{
	
		display: flex;
		justify-content: space-between;
		align-items: center;
		border-top:1px solid #eee;
		padding:15px 0;
		.o-left{
			position:relative;
			width:50px;
			height:50px;
			flex-shrink: 0;
			overflow: hidden;
			    border-radius: 5px;
			.order_img{
				width:100%;
				height:100%;
				border:1px solid #eee;
			}
		}
		.omid{
			display: flex;
			flex-direction: column;
			color:#505050;
			padding-left:10px;
			flex:1;
			justify-content: space-between;
			p{
				@include longtext2;
				max-height:35px;
			}
			span{
				
				@include longtext;
				max-width:100px;
				background:#f5f5f5;
				color:#999;
				border-radius: 20px;
				padding:3px 10px;
				margin-top:5px;
				align-self: baseline;
			}
		}
		.o-right{
			display: flex;
			flex-direction: column;
			justify-content: space-between;
			flex-shrink: 0;
			align-self:stretch;
			text-align:right;
			margin-left:10px;
			span{
				color:#CF2503
			}
			.or-bottom{
				display: flex;
				flex-direction: column;
				margin-top:10px;
				span:last-child{
					color:#999;
					
				}
			}
		}
	}
	.total_box{
		border-top:1px solid #eee;
		.total_one{
			padding:10px 0 ;
			display: flex;
			justify-content: center;
			.to_left{
				display: flex;
				align-items: center;
				width:50%;
				.otype{
					
						
						color:#147AED;
						border-radius:4px;
						padding:3px 5px;
						margin-left:5px
					
				}
				.qrcode_btn{
					border-radius:20px;
					padding:3px 10px;
					color:#fff;
					text-align: center;
					margin-right:10px;
					box-sizing: border-box;
				}
				.sh_btn{
					background:#078CF3
				}
				.qrcode_btn{
					background:#EE453C
				}
				
			}
			.to_right{
				width:50%;
				display: flex;
				flex-direction: column;
				align-items: flex-end;
				justify-content: center;
				color:#999;
				
				span{
					margin-left:1rem;
					
				}
				
			}
			
		}
		.total_two{
			
			display: flex;
			justify-content: space-between;
			align-items: center;
			color:#999;
			padding-bottom:10px;
			span:last-child{
				font-size:14px;
				color:#EE453C;
				font-weight: 600;
			}
		}
	}
}

//头部业务员信息
.shoplist_content{
	margin-bottom:10px;
}
.shoplist_content_item{
		padding:.5rem;
		background:#fff;
		border-radius: 4px;
		
	}
	.shoplist_content_item_top{
		
		display: flex;
		justify-content: space-between;
		
	}
	.top_left{
		display: flex;
	}
	.top_info{
		display: flex;
		flex-direction: column;
		justify-content: center;
		margin-left:.5rem;
		a{
			display:flex;
			color:#1D88E2;
			
		}
		&>span:first-child{
			font-size:14px;
			margin-bottom:5px;
		}
		&>span:last-child{
			display:flex;
			color:#1D88E2;
			i{
				vertical-align: bottom;
				margin-right:5px;
			}
		}
	}
	.top_img{
		width:50px;
		height:50px;
		border-radius: 4px;
	}
	.shoplist_content_item_bottom{
		display: flex;
		align-items: center;
		margin-top:.5rem;
		i{
		
			color:#44AE69;
			font-size:12px;
		}
		p{
			color:#888;
			padding-left:5px;
			@include longtext;
		}
	}
	.top_right_type{
		display: flex;
		align-items: center;
		color:#888;
		span{
			border-radius: 20px;
			background:#23A6F1;
			color:#fff;
			padding:3px 10px;
			font-size:14px;
		}
	}
	
</style>
