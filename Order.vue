	<template>
		<view>订单列表
			<button @click="orderView">刷新订单</button>
			<div v-for="item in currentOrder" :key="item.id">
			{{parkingLot[item.parkingLotId].description}}
			{{item.beginTime}}至{{item.endTime}}  
			{{status[item.cancelFlag]}}
			<button @click="orderOperate" :data-info='item'>{{operate[item.cancelFlag]}}</button>
			</div>
		</view>
	</template>

	<script>
		export default {
			data() {
				return {
					id:'',
					token:'',
					chargeRule:[],
					distance:'',
					description:'',
					latitude:'',
					longitude:'',
					begTime:'',
					endTime:'',
					currentOrder:[],
					parkingLot:[],
					status:['已支付','已取消','已过期'],
					operate:['取消','恢复','评价(敬请期待)'],
					userId:0,
				}
			},
			onShow(e) {
				var that = this
				uni.getStorage({
					key:'userId',
					success(res) {
						that.userId=res.data;
						console.log(that.userId);
						that.userId=9;
						
					},
				})
				uni.getStorage({
					key:'token',
					success:function(r)
					{
						that.token = r.data
					}
				})
			},
			methods: {
				orderView:function(){//刷新订单
					var that = this
					uni.request({//刷新停车场信息
						url:'http://47.97.90.35:8080/findAllParkingLot',
						method:'GET',
						header:{token:JSON.parse(that.token)},
						success(res) {
							that.parkingLot=res.data.data
						}
					}),
					uni.request({//刷新用户订单
						url:'http://47.97.90.35:8080/findOrdersByUserId/'+that.userId,
						method:'GET',
						header:{token:JSON.parse(that.token)},
						success(res) {
							console.log(res)
							that.currentOrder=res.data.data
							
							var cur_time=new Date().getTime();
							for(var i=0;i<that.currentOrder.length;++i){
								if(new Date(that.currentOrder[i].beginTime).getTime() <= cur_time){
									console.log('过期');
									that.currentOrder[i].cancelFlag=2;
								}
								
							}
							
							if(that.currentOrder.length==0){
								uni.showToast({
									title:'暂无订单',
									icon:'loading',
								})
							}
						}
					})
				},
				orderOperate:function(e){
					console.log(e)
					var that = this
					var orderId = e.currentTarget.dataset.info.orderId
					if(e.currentTarget.dataset.info.cancelFlag==0)
					{
						uni.request({
							url:'http://47.97.90.35:8080/cancelOneOrderById/'+orderId,
							method:'GET',
							header:{token:JSON.parse(that.token)},
							success(res) {
								console.log(res)
								if(res.data.message=='操作成功')
								{
									uni.showToast({
										title:'取消成功'
									}),
									that.orderView();
								}else{
									console.log(res.data.message);
								}
							},
							fail() {
								uni.showToast({
									title:'网络错误！',
									icon:'error',
								})
							}
						})
					}
					else if(e.currentTarget.dataset.info.cancelFlag==1)
					{
						uni.request({
							url:'http://47.97.90.35:8080/restoreOneOrderById/'+orderId,
							header:{token:JSON.parse(that.token)},
							method:'GET',
							success(res) {
								console.log(res)
								if(res.data.message=='操作成功')
								{
									uni.showToast({
										title:'恢复成功'
									}),
									that.orderView();
								}
								else{
									console.log(res.data.message);
								}
							},
							fail() {
								uni.showToast({
									title:'网络错误！',
									icon:'error',
								})
									
							}
							
						})
					}
					
				}
				
				
			}
		}
	</script>

	<style>

	</style>
