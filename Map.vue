<!-- 点击定位跳转到用户当前位置，点击停车将会根据地图中心点来就近获取停车场，点击列表将定位到停车场（这时候再点击停车会得到当前停车场距离为0，看似睿智实际影响不大，因为目的地必须离停车场较近） -->
<template>
	<view>
		<view style="display: flex; flex-direction: row;">
			<button style="flex: 1;" @click="update_location">定位</button>
			<button style="flex: 1;" @click="update_park">停车</button>
		</view>
		<map id="Mymap"  
			style="width:750rpx; height: 750rpx;"
			
			scale="16" 
			
			show-compass="true"
			enable-rotate="true"
			enable-overlooking="true"
			enable-traffic="true"
			enable-poi="true"
			enable-building="true"
			show-location="true"
			
			
			:markers="parklot" 
			:circles="circle"
			
			
			@markertap="markertap">
		</map>
		<view v-for="(item,index) in parklot" :key="index" >
			<button @click="openOrder" :data-id="item.id">id:{{item.id}}{{item.description}}{{item.distance}}km</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				token:'',
				parkingLot:[],//停车场经纬度信息
				parklot:[],//停车场完整信息（按距离排序）     id等价于该停车场在parkingLot中的下标
				circle:[],//这是一个圆
				
			}
		},
		onShow() {
			var that=this;
			uni.getStorage({
				key:'token',
				success(res) {
					that.token=res.data;
					//console.log(that.token);
					that.update_location();
				},
			})
		},
		methods: {
			markertap(e){
				console.log(e);
				var that=this;
				var idx=e.detail.markerId;
				console.log(that.parkingLot[idx].description);
				console.log(that.parkingLot[idx].distance);
				// uni.redirectTo({
				// 	url:'./Info/Order?id='+id
				// })
			},
			openOrder(i){//跳转到停车场位置
				var that=this;
				//console.log(i);
				var idx=i.target.dataset.id;
				//console.log(that.parkingLot[idx].latitude);
				var X=that.parkingLot[idx].latitude;
				var Y=that.parkingLot[idx].longitude;
				
				that._map=uni.createMapContext("Mymap",that);
				that._map.moveToLocation({
					latitude:X,
					longitude:Y,
					success(r) {
						//console.log('获取位置成功');
					},
					fail(){
						console.log('定位位置失败');
					},
				})
			},
			Rad(d) {
				return d * Math.PI / 180.0; //经纬度转换成三角函数中度分表形式。
			},
			/*
			 计算距离，参数分别为第一点的纬度，经度；第二点的纬度，经度
			 默认单位km
			*/
			getMapDistance(lat1, lng1, lat2, lng2) {
				var radLat1 = this.Rad(lat1);
				var radLat2 = this.Rad(lat2);
				var a = radLat1 - radLat2;
				var b = this.Rad(lng1) - this.Rad(lng2);
				var s = 2 * Math.asin(Math.sqrt(Math.pow(Math.sin(a / 2), 2) +
					Math.cos(radLat1) * Math.cos(radLat2) * Math.pow(Math.sin(b / 2), 2)));
				s = s * 6378.137; // EARTH_RADIUS;
				s = Math.round(s * 10000) / 10000; //输出为公里
				//s=s.toFixed(2);
				return s;
			},
			update_location(){
				var that=this;
				uni.getLocation({
					success(res) {
						var X=res.latitude;
						var Y=res.longitude;
						that._map=uni.createMapContext("Mymap",that);
						that._map.moveToLocation({
							latitude:X,
							longitude:Y,
							success(r) {
								//console.log('获取位置成功');
								
							},
							fail(){
								console.log('定位位置失败');
							},
							complete(){
								that.circle=[];
								that.circle[0]={
													"latitude":X,
													"longitude":Y,
													"radius":300,
													"color":'#00aa00',
												};
							}
						})
						
						// that.control[0]={
						// 					"position":{
						// 									"left":0,
						// 									"top":0,
						// 								},
						// 					"iconPath":'/static/point.png',
						// 				};
					},
					fail() {
						console.log('获取位置失败');
						uni.showToast({
							title:'请重试',
							icon:'loading',
						})
					}
				})
			},
			update_park(){
				var that=this;
				that._map=uni.createMapContext("Mymap",that);
				that._map.getCenterLocation({
					success(res) {
						console.log(res);
						var X=res.latitude;
						var Y=res.longitude;
						that.circle=[];
						that.circle[0]={
											"latitude":X,
											"longitude":Y,
											"radius":300,
											"color":'#00aa00',
										};
						uni.request({
							url:'http://47.97.90.35:8080/findAllParkingLot',
							method:'GET',
							header:{token:JSON.parse(that.token)},//无语
							success(res) {
								//console.log(res);
								if(res.data.code!="200"){
									console.log("停车场信息获取失败！");
									return;
								}
								//console.log("停车场信息获取成功！");
								that.parkingLot=res.data.data
								console.log(that.parkingLot);
								//that.description=res.data.data[0].description
								that.parklot=[];
								for(var i=0;i<that.parkingLot.length;++i)
								{
									let p={}
									let key='latitude'
									let value=that.parkingLot[i].latitude
									p[key] = value
									key = 'longitude'
									value = that.parkingLot[i].longitude
									p[key]=value
									// key='parkingLotId'
									// value=that.parkingLot[i].parkingLotId
									// p[key]=value
									// key='category'
									// value=that.parkingLot[i].category
									// p[key]=value
									// key='chargeRuleId'
									// value=that.parkingLot[i].chargeRuleId
									// p[key]=value
									key='description'
									value=that.parkingLot[i].description
									p[key]=value
									
									key='callout'
									value={
												content: 'id:'+i,
												borderRadius: 5,
												display: "ALWAYS",
												padding: 7,
												bgColor: "#FFFFFF",
											}
									p[key]=value
									key='distance'
									value=that.getMapDistance(
										X,Y,that.parkingLot[i].latitude,that.parkingLot[i].longitude)
									p[key]=value
									
									that.parkingLot[i][key]=value
									
									key = 'iconPath'
									value = '/static/point.png'
									p[key]=value
									key='id'
									value=i
									p[key]=value
									
									
									
									that.parklot.push(p)
									//console.log(p)
								}
								that.parklot.sort(
								(a,b)=>{
									return (a.distance<b.distance) ? -1 : (a.distance>b.distance) ? 1 :0
								}
								)
							},
							fail() {
								console.log('fail')
							}
						})
					},
					fail(){
						console.log('定位失败');
					},
				})
				
			},
		}
	}
		
	
</script>

<style>
	
</style>
