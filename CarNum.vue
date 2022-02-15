<template>
	<view>
		<uni-nav-bar left-icon="left" leftText="返回" rightText="保存" title="车牌修改" @clickLeft="clickLeft" @clickRight="clickRight"/>
		<uni-forms ref="form" :modelValue="formData" :rules="rules">
			<uni-forms-item name="carNum">
				<uni-easyinput  v-model="formData.carNum" placeholder="请输入车牌号" />
			</uni-forms-item>
		</uni-forms>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				userId:'',
				token:'',
				formData:{
					carNum:'',
				},
				rules:{
					carNum:{
						rules:[
							{
								required:true,
								errorMessage:"请输入车牌号",
							}
						]
					},
				}
			}
		},
		onLoad() {
			this.$refs.form.setRules(this.rules)
			var that=this;
			uni.getStorage({
				key:'userId',
				fail() {
					console.log('userId获取失败');
					uni.switchTab({
						url:'pages/User/User'
					})
				},
				success:function(res){
					console.log('userId获取成功');
					that.userId=JSON.parse(res.data)
					//console.log(that.userId)
				}
			});
			uni.getStorage({
				key:'token',
				fail() {
					console.log('token获取失败');
					uni.switchTab({
						url:'pages/User/User'
					})
				},
				success:function(res){
					console.log('token获取成功');
					that.token=JSON.parse(res.data)
				}
			})
		},
		methods: {
			clickLeft(){
				uni.navigateBack({
					delta: 1,
				})
			},
			clickRight(){
				var that=this;
				this.$refs.form.validate().then(res=>{
					console.log("ok");
					res["userId"]=that.userId;
					console.log(res);
					uni.request({
						url:'http://47.97.90.35:8080/user/updateUserById',
						data:res,
						header:{token:that.token},
						method:'PUT',
						success(r) {
							console.log(r);
							if(r.data.code=="200")
							{
								uni.showToast({
									title:'修改成功',
									icon:'none',
									complete() {//页面切换太快这弹窗基本看不见
										that.clickLeft();
									}
								});			
							}
							else{
								uni.showToast({
									title:'修改失败',
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
