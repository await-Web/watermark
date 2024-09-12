<template>
	<!-- 视频 -->
	<view class=" video-box u-flex-col">
		<image v-for="(item,index) in src" :key="index" :src="item" @tap=" previewImage(index)" showmenu loop indicator
			mode="widthFix">
		</image>
		<view class="">
			<text
				style="color: red;padding-bottom: 40rpx;font-size: 22px;">可以解析图片，也可以解析视频。如果解析的是图片又下载不了，就直接点击图片，放大之后，长按保存!
			</text>
			<text style="color: red;padding-bottom: 40rpx;font-size: 22px;">快手，抖音，B站操作方法一样的，找到作品分享链接复制链接即可</text>
		</view>

	</view>
</template>

<script>
	export default {
		data() {
			return {
				src: ['/static/image/1.jpg', '/static/image/2.jpg', '/static/image/3.jpg', '/static/image/4.jpg']
			}
		},
		methods: {
			// 预览图片
			previewImage(i) {
				uni.previewImage({
					urls: this.src,
					current: i,
					longPressActions: {
						itemList: ['发送给朋友', '保存图片', '收藏'],
						success: function(data) {
							console.log('选中了第' + (data.tapIndex + 1) + '个按钮,第' + (data.index + 1) + '张图片');
						},
						fail: function(err) {
							console.log(err.errMsg);
						}
					}
				});
			},
		}
	}
</script>

<style lang="scss">
	page {
		background-color: #f0f2f6;
	}

	.video-box {
		width: 100%;
		padding: 20rpx;

		::v-deep image {
			width: 100%;
			margin-bottom: 20rpx;
		}
	}
</style>