<template>
	<view class="content">
		<image :src="m" @click="go"></image>
		<view v-for="item in arr" style="display: flex;">
			<view>{{item.name}} </view>
			<view>{{item.score}}</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				m:"/static/logo.png",
				arr:[]
			}
		},
		onLoad() {

		},
		methods: {
			go() {
				let that=this;
				uni.chooseImage({
					count: 1, //默认9
					sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
					success: function(chooseImageRes) {
						let tempFilePaths = chooseImageRes.tempFilePaths;
						that.m=  tempFilePaths[0];
						//该方法只能app 中使用
						let index = tempFilePaths[0].lastIndexOf("."); //获取图片地址最后一个点的位置
						let img_type = tempFilePaths[0].substring(index + 1, tempFilePaths[0].length); //截取图片类型如png jpg
						let img_yuanshi = tempFilePaths[0].substring(0, index); //截取图片原始路径
						let d2 = new Date().getTime(); //时间戳
						//压缩图片
						plus.zip.compressImage({
							src: tempFilePaths[0], //你要压缩的图片地址
							dst: img_yuanshi + d2 + '.' +img_type, //压缩之后的图片地址(注意压缩之后的路径最好和原生路径的位置一样，不然真机上报code-5)
							width: '600px',
							height: 'auto'
						}, function(e) {
							plus.io.resolveLocalFileSystemURL(e.target, function(entry) {
								entry.file(function(file) {
									var fileReader = new plus.io.FileReader();
									fileReader.readAsDataURL(file);
									fileReader.onloadend = function(evt) {
										let imagebase64 = evt.target.result;
										uni.request({
											url:"http://10.20.5.3:3000/animal",
											data:{"image":imagebase64},
											method:"POST",
											success(re) {
												console.log(re);
												that.arr= re.data.result;
											}
										})
									}
								});
							});
						});
					}
				});
			}
		},
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
</style>
