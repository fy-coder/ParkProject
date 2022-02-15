<template>
	<view>
		<uni-list border-full="true">
			<uni-list-item title="用户id:" :rightText="userId"/>
			<uni-list-item title="用户名:" :rightText="username"/>
			<uni-list-item title="车牌号:" :rightText="carNum" showArrow="true" clickable="true" to="Info/CarNum"/> 
			<uni-list-item title="手机号:" :rightText="phoneNum" showArrow="true" clickable="true" to="Info/PhoneNum"/> 
			<uni-list-item title="注册时间:" :rightText="registerTime"/>
			<uni-list-item title="密码:" rightText="******" showArrow="true" clickable="true" to="Info/Password"/>
		</uni-list>
		<button @click="LogOut" >退出登录</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				username:'null',
				token:'',
				userId:'0',
				carNum:'未填写',
				phoneNum:'未填写',
				registerTime:'null',
				password:'',
			}
		},
		onTabItemTap(e) {
			//console.log(e);
		},
		onShow() {
			var that=this;
			uni.getStorage({
				key:'username',
				fail() {
					uni.navigateTo({
						url:'/pages/Login/Login',
					})
				},
				success:function(res){
					that.username=JSON.parse(res.data);
					console.log(that.username);
				}
			})
			uni.getStorage({
				key:'password',
				fail() {
					uni.navigateTo({
						url:'/pages/Login/Login',
					})
				},
				success:function(res){
					that.password=JSON.parse(res.data);
					console.log(that.password);
				}
			})
			uni.getStorage({
				key:'token',
				fail() {
					uni.navigateTo({
						url:'/pages/Login/Login',
					})
				},
				success:function(res){
					that.token=JSON.parse(res.data);
					//console.log(that.token);
					uni.request({//获取用户信息
						url:'http://47.97.90.35:8080/user/findUserByUserName/'+that.username,
						method:'GET',
						header:{token:that.token},
						success: res => {
							console.log(res);
							if(res.data.code=='200'){
								console.log("获取用户信息成功");
								that.carNum=res.data.data.carNum==null?"未填写":res.data.data.carNum;
								that.userId=res.data.data.userId;
								// that.password=res.data.data.password;
								that.registerTime=res.data.data.registerTime;
								that.phoneNum=res.data.data.phoneNum==null?"未填写":res.data.data.phoneNum;
								uni.setStorage({
									key:'userId',
									data:res.data.data.userId,
									complete(data) {
										console.log('设置userId完成');
									},
									fail() {
										console.log('设置userId失败');
									}
								})
							}else{
								console.log("获取用户信息失败");
							}
						},
						fail() {
							console.log("获取用户信息失败2.0");
						},
					})
					
				},
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
					},
					complete() {
						console.log('完成');
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
