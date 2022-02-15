<!-- 点击定位跳转到用户当前位置，点击停车将会根据地图中心点来就近获取停车场，点击列表将定位到停车场（这时候再点击停车会得到当前停车场距离为0，看似睿智实际影响不大，因为目的地必须离停车场较近） -->
<template>
	<view>
		<view style="display: flex; flex-direction: row;">
			<button style="flex: 1;" @click="update_location">定位</button>
			<button style="flex: 1;" @click="update_park">停车</button>
		</view>
		<map id="Mymap"  
			style="width:750rpx; height: 750rpx;"
			:latitude="X"
			:longitude="Y"
			scale="11" 
			@regionchange="regionchange"
			show-location="true" :markers="parklot" @callouttap="callouttap">
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
				id:0,
				token:'',
				X:32,//
				Y:112,
				_X:32,//（响应拖动事件）
				_Y:112,
				// latitude:32,//用户中心点
				// longitude:112
				parkingLot:[],//停车场经纬度信息
				parklot:[],//停车场完整信息（按距离排序）
				description:'',
				distance:'',
				spare:'',
				charge:[],
			}
		},
		onShow() {//从order切换回来时不应该更新
			this.update_location();
			this.update_park();
		},
		methods: {
			callouttap(e){
				//console.log(e)
				var id=e.detail.markerId;
				uni.redirectTo({
					url:'../order/order?id='+id
				})
			},
			openOrder(i){//跳转到停车场位置
				var that=this;
				//console.log(i);
				var idx=i.target.dataset.id;
				// console.log(that.parkingLot[idx].latitude);
				that.X=that.parkingLot[idx].latitude;
				that.Y=that.parkingLot[idx].longitude;
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
			regionchange(e){
				var that=this;
				if(e.type=='end'){
					that._X=e.detail.centerLocation.latitude;
					that._Y=e.detail.centerLocation.longitude;
				}
				console.log('latitude:'+that._X);
				console.log('longitude:'+that._Y);
			},
			update_location(){
				var that=this;
				that.X=that._X;
				that.Y=that._Y;
				uni.getLocation({
					success(res) {
						console.log(res);
						that.X=res.latitude;
						that.Y=res.longitude;
						that._X=res.latitude;
						that._Y=res.longitude;
						console.log('获取位置成功');
						console.log('latitude:'+that.X);
						console.log('longitude:'+that.Y);
					},
					fail() {
						console.log('获取位置失败');
					}
				})
			},
			update_park(){
				var that=this;
				that.X=that._X;
				that.Y=that._Y;
				uni.getStorage({
					key:'token',
					success(r) {
						that.token=r.data
						uni.request({
							url:'http://47.97.90.35:8080/findAllParkingLot',
							method:'GET',
							header:{token:JSON.parse(that.token)},
							success(res) {
								//console.log(res.data.data)
								that.parkingLot=res.data.data
								that.description=res.data.data[0].description
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
									key = 'iconPath'
									value = '/static/point.png'
									p[key]=value
									key='id'
									value=i
									p[key]=value
									key='callout'
									value={content: 'id:'+i,
												borderRadius: 5,
												display: "ALWAYS",
												padding: 7,
												bgColor: "#FFFFFF"}
									p[key]=value
									key='distance'
									value=that.getMapDistance(
										that.X,that.Y,that.parkingLot[i].latitude,that.parkingLot[i].longitude)
									p[key]=value
									key='description'
									value=res.data.data[i].description
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
					complete() {
						console.log('complete')
					}
				})
			},
		}
	}
		
	
</script>

<style>
	
</style>
