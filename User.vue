<!-- 在User文件夹下还有一个Info文件夹未上传，不影响主功能 -->
<template>
	<view>
		<button class="user_row">
			<text>用户id：</text>
			<text>{{userId}}</text>
		</button>
		<button class="user_row">
			<text>用户名：</text>
			<text>{{username}}</text>
		</button>
		<navigator url="./Info/CarNum"><button class="user_row">
			<text>车牌号：</text>
			<text>{{carNum}} &gt</text>
		</button></navigator>
		<navigator url="./Info/PhoneNum"><button class="user_row">
			<text>手机号：</text>
			<text>{{phoneNum}} &gt</text>
		</button></navigator>
		<button class="user_row">
			<text>注册时间：</text>
			<text>{{registerTime}}</text>
		</button>
		<navigator url="./Info/Password"><button class="user_row">
			<text>密码：</text>
			<text>****** &gt</text>
		</button></navigator>
		<button @click="LogOut" >退出登录</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				username:'null',
				token:'null',
				userId:'0',
				carNum:'未填写',
				phoneNum:'未填写',
				registerTime:'null',
				password:'******',
			}
		},
		onTabItemTap() {
			console.log("User onTabItemTap");
			var that=this
			uni.getStorage({
				key:'userInfo',
				fail() {
					uni.showToast({
						title:'您尚未登录',
					})
					uni.navigateTo({
						url:'/pages/Login/Login',
					})
				},
				success:function(res){
					console.log('User已登录')
					that.username=JSON.parse(res.data)
					//console.log(that.username)
				},
				fail() {
					console.log('fail');
				}
			})
			uni.getStorage({
				key:'token',
				success:function(res){
					//console.log(res.data)
					that.token=res.data
				},
				fail() {
					console.log('fail');
				}
			})
			uni.request({//获取信息
				url:'http://47.97.90.35:8080/user/findUserByUserName/'+that.username,
				method:'GET',
				header:{token:JSON.parse(that.token)},
				success: res => {
					if(res.data.code=='200'){
						console.log("获取用户信息成功");
						that.carNum=res.data.data.carNum==null?"未填写":res.data.data.carNum;
						that.userId=res.data.data.userId;
						that.password=res.data.data.password;
						that.registerTime=res.data.data.registerTime;
						that.phoneNum=res.data.data.phoneNum==null?"未填写":res.data.data.phoneNum;
					}else{
						console.log("获取用户信息失败");
					}
					console.log(res);
				},
				fail() {
					console.log("获取用户信息失败2.0");
				},
			})
			console.log('over');
		},
		methods: {
			LogOut(){
				console.log("退出登录");
				var that=this
				uni.request({
					url:'http://47.97.90.35:8080/logOut',
					method:'GET',
					header:{token:JSON.parse(that.token)},
					success(res) {
						console.log(res);
						uni.clearStorage();
						uni.navigateTo({
							url:'/pages/Login/Login',
						})
					},
					fail() {
						console.log("退出失败");
					}
				})
				
			}
			
		}
	}
</script>

<style>
	.user_row{
		display: flex;
		width: 100%;
		justify-content: space-between;
	}

</style>
