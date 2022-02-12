<template>
	<view>
		<text>停车app</text>
		<text>用户名{{username}}</text>
		<button @click="LogOut">退出登录</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				username:'',
				token:'',
			}
		},
		onLoad(e) {
			console.log("index开始加载");
			var that=this;
			uni.getStorage({
				key:'userInfo',
				fail() {
					console.log('userInfo获取失败');
					uni.redirectTo({
						url:'/pages/Login/Login',
					})
				},
				success:function(res){
					console.log('userInfo获取成功');
					that.username=JSON.parse(res.data)
					console.log(that.username)
				}
			})
			uni.getStorage({
				key:'token',
				fail() {
					console.log('token获取失败');
					uni.redirectTo({
						url:'/pages/Login/Login',
					})
				},
				success:function(res){
					console.log('token获取成功');
					that.token=res.data
					console.log(that.token)
					uni.switchTab({//执行顺序有些问题
						url:'/pages/Home/Home',
					})
				}
			})
		},
		methods: {
			LogOut(){
				var that=this
				console.log(this.token)
				uni.request({
					url:'http://47.97.90.35:8080/logOut',
					method:'GET',
					header:{token:JSON.parse(that.token)},
					success(res) {
						console.log(res)
					}
				})
				uni.clearStorage();
				uni.redirectTo({
					url:'/pages/Login/Login',
				})
			}
		}
	}
</script>

<style>

</style>
