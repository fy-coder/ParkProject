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
				password:'',
				token:'',
			}
		},
		onLoad(e) {
			console.log("index开始加载");
			var that=this;
			uni.getStorage({
				key:'username',
				fail() {
					console.log('username获取失败');
					uni.redirectTo({
						url:'/pages/Login/Login',
					})
				},
				success:function(res){
					console.log('username获取成功');
					that.username=JSON.parse(res.data)
					//console.log(that.username)
				}
			})
			uni.getStorage({
				key:'password',
				fail() {
					console.log('password获取失败');
					uni.redirectTo({
						url:'/pages/Login/Login',
					})
				},
				success:function(res){
					console.log('password获取成功');
					that.password=JSON.parse(res.data)
					//console.log(that.password)
					
					//自动登录
					let p={};
					let key='username';
					let value=that.username;
					p[key]=value;
					key='password';
					value=that.password;
					p[key]=value;
					//console.log(p);
					uni.request({
						url:'http://47.97.90.35:8080/login',
						method:'POST',
						data:JSON.stringify(p),
						success: res => {
							console.log(p);
							console.log(res);
							if(res.data.code=="200"){
								console.log('自动登录成功');
								
								uni.setStorage({
									key:'token',
									data:JSON.stringify(res.data.data.token),
									complete(data) {
										console.log('设置token完成')
										//console.log(data)
										console.log('即将跳转');
										uni.switchTab({//执行顺序有些问题
											url:'/pages/Map/Map',
										})
									}
								})
					
							}
							else{
								console.log('自动登录失败');
								//uni.clearStorage();
								uni.redirectTo({
									url:'/pages/Login/Login',
								})
							};
							
						},
						fail() {
							console.log('自动登录失败2.0');
							//uni.clearStorage();
							uni.redirectTo({
								url:'/pages/Login/Login',
							})
						}
					})
				}
			})
			
			
		},
		methods: {
			LogOut(){
				var that=this;
				uni.request({
					url:'http://47.97.90.35:8080/logOut',
					method:'GET',
					header:{token:that.token},
					success(res) {
						console.log('退出登录成功');
						uni.clearStorage();
						uni.redirectTo({
							url:'/pages/Login/Login',
						})
					},
					fail() {
						console.log('退出登录失败');
					}
				})
			}
		}
	}
</script>

<style>

</style>
