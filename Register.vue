<template>
	<view>
		<uni-forms ref="form" :modelValue="formData" :rules="rules">
			<uni-forms-item name="username">
				<uni-easyinput  v-model="formData.username" placeholder="请输入用户名" />
			</uni-forms-item>
			<uni-forms-item name="password">
				<uni-easyinput  v-model="formData.password" placeholder="请输入密码" type="password"/>
			</uni-forms-item>
			<uni-forms-item name="passwordConfirm">
				<uni-easyinput  v-model="formData.passwordConfirm" placeholder="确认密码" type="password" />
			</uni-forms-item>
		</uni-forms>
		<button @click="submit">注册</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				token:'',
				formData:{
					username:'',
					password:'',
					passwordConfirm:'',
				},
				rules:{
					username:{
						rules:[
							{
								required:true,
								errorMessage:"请输入用户名",
							},
							{
								maxLength:8,
								errorMessage:"用户名不能超过8个字符",
							},
							{
								minLength:3,
								errorMessage:"用户名不应少于3个字符"
							}
						]
					},
					password:{
						rules:[
							{
								required:true,
								errorMessage:"请输入密码",
							},
							{
								minLength:6,
								errorMessage:"密码应长于6个字符",
							},
							{
								maxLength:15,
								errorMessage:"密码不应长于15个字符"
							},
						]
					},
					passwordConfirm:{
						rules:[
							{
								required:true,
								errorMessage:"请确认密码",
							},
							{
								validateFunction:function(rule,value,data,callback){
									if(value!=data.password)
									{
										callback('两次密码输入不一致')
									}
									return true
								}
							}
						]
						
					},
				}
			}
		},
		onReady() {
			this.$refs.form.setRules(this.rules)
		},
		methods: {
			submit:function(){
				console.log("注册");
				this.$refs.form.validate().then(res=>{
					console.log("ok");
					uni.request({
						url:'http://47.97.90.35:8080/register',
						data:JSON.stringify(res),
						method:'POST',
						success(r) {
							console.log(r);
							if(r.data.code=="200")
							{
								if(r.data.msg=="注册成功"){
									uni.showToast({
										title:'注册成功',//未实现注册后自动登录
										icon:'none',
										complete() {//页面切换太快这弹窗基本看不见
											uni.redirectTo({
												url:'../Login/Login'
											});
										}
									});
										
								}else{
									uni.showToast({
										title:'用户名重复',
										icon:'error',
									});
								}					
							}
							else{
								uni.showToast({
									title:'注册失败',
									icon:'error',
								});
							}
						},
						fail(){
							uni.showToast({
								title:'网络错误，请重试',
								icon:'error'
							});
						}
				 	})
				}
				).catch(err=>{
					console.log("no");
					console.log(err);
				})
			},
		}
	}
</script>

<style>

</style>
