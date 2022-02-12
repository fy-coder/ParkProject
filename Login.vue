<template>
	<view class="screen">
		
		<form @submit="formSubmit">
			<input class="input" name="username" placeholder="请输入用户名" />
			<input class="input" name="password" placeholder="请输入密码" password=true/>
			<button class="button" form-type="submit" >登录</button>
		</form>
		<navigator class="button"  url="../Register/Register"><button>注册</button></navigator>
		
		<view class="uni-form-item uni-column">
		    <view class="title"><text class="uni-form-item__title">可查看密码的输入框</text></view>
		    <view class="uni-input-wrapper">
		        <input class="uni-input" placeholder="请输入密码" :password="showPassword" />
		        <text class="uni-icon" :class="[!showPassword ? 'uni-eye-active' : '']" @click="changePassword"> &#xe568; </text>
		    </view>
		</view>
	</view>
</template>

<script>
	export default {
		onLoad() {
			
		},
		data() {
			return {
				token:'',
				isSuccess:false,
				latitude:0,
				longitude:0,
				showPassword: true,
			}
		},
		onReady() {
			uni.getStorage({
				key:'userInfo',
				success:function(res)
				{
					uni.showToast({
						title:'您已登录',
					})
				}
			})
		},
		methods: {
			changePassword: function() {
			    this.showPassword = !this.showPassword;
			},
			formSubmit:function(e){
				console.log("formSubmit");
				
				uni.request({
					url:'http://47.97.90.35:8080/login',
					method:'POST',
					data:JSON.stringify(e.detail.value),
					success: res => {
						if(res.data.code=="200"){
							uni.showToast({
								title: '登录成功'
							});
							this.token=res.data.token;
							uni.setStorage({
								key:'userInfo',
								data:JSON.stringify(e.detail.value.username),
								complete(data) {
									console.log('设置userInfo完成')
									console.log(data)
								}
							})
							uni.setStorage({
								key:'token',
								data:JSON.stringify(res.data.data.token),
								complete(data) {
									console.log('设置token完成')
									console.log(data)
									console.log('即将跳转');
									uni.switchTab({//执行顺序有些问题
										url:'/pages/Home/Home',
									})
								}
							})
							this.isSuccess=true;
							
							// uni.getLocation({
							// 	success(res) {
							// 		uni.redirectTo({
							// 			url:'../Map/Map?data='+JSON.stringify({latitude:res.latitude,longitude:res.longitude}) ,
							// 			complete() {
							// 				console.log(res.latitude)
							// 				console.log(res.longitude)
							// 			}
							// 		})
								
							// 	}
							// })	
						}
						else{
							uni.showToast({
								title: '请检查用户名或密码',
								icon:"none"
							});
						}
					},
					fail() {
						uni.showToast({
							title:"网络错误，请检查网络"
						})
					}
				})
				
			},
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
	.input{
		margin: 70rpx;
	
	}
	
	.screen{
			
	}
	.button{
		margin: 50rpx;
		font:'Gill Sans', 
		
	}
	
	
	
	
	.nvue-page-root {
	    background-color: #F8F8F8;
	    padding-bottom: 20px;
	}
	
	.page-title {
	    /* #ifndef APP-NVUE */
	    display: flex;
	    /* #endif */
	    flex-direction: row;
	    justify-content: center;
	    align-items: center;
	    padding: 35rpx;
	}
	
	.page-title__wrapper {
	    padding: 0px 20px;
	    border-bottom-color: #D8D8D8;
	    border-bottom-width: 1px;
	}
	
	.page-title__text {
	    font-size: 16px;
	    height: 48px;
	    line-height: 48px;
	    color: #BEBEBE;
	}
	
	.title {
	    padding: 5px 13px;
	}
	
	.uni-form-item__title {
	    font-size: 16px;
	    line-height: 24px;
	}
	
	.uni-input-wrapper {
	    /* #ifndef APP-NVUE */
	    display: flex;
	    /* #endif */
	    padding: 8px 13px;
	    flex-direction: row;
	    flex-wrap: nowrap;
	    background-color: #FFFFFF;
	}
	
	.uni-input {
	    height: 28px;
	    line-height: 28px;
	    font-size: 15px;
	    padding: 0px;
	    flex: 1;
	    background-color: #FFFFFF;
	}
	
	.uni-icon {
	    font-family: uniicons;
	    font-size: 24px;
	    font-weight: normal;
	    font-style: normal;
	    width: 24px;
	    height: 24px;
	    line-height: 24px;
	    color: #999999;
	}
	
	.uni-eye-active {
	    color: #007AFF;
	}
	
</style>
