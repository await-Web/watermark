<template>
	<view>
		<view class="">
			<!-- 解析完成页 -->
			<view class=" u-m-t-20 u-m-b-20">
				<ad-custom unit-id="adunit-3632b21645b42b52" ad-intervals="30"></ad-custom>
			</view>
			<view class="u-flex-col content  u-p-l-20 u-p-r-20">
				<!-- 图片 -->
				<view class="imgs-box u-flex" v-if="detialData?.imageAtlas?.length">
					<scroll-view scroll-y="true" class="scroll-Y" @scrolltoupper="upper" @scrolltolower="lower"
						@scroll="scroll">
						<view class="u-flex scroll-box">
							<view class="img-item " v-for="(item,index) in detialData.imageAtlas" :key="index">
								<image :src="item" class="image-sty" @tap="previewImage(index)"></image>
								<u-button type="primary" size="mini" @click="handleDownloads(item,'img')"
									style="position: absolute;bottom: 8rpx;left: 8rpx;">下载</u-button>
							</view>
							<view class="glare-effect" @click="jump">
								查看更多壁纸
							</view>
						</view>
					</scroll-view>
				</view>
				<!-- 视频 -->
				<view class="u-m-t-20 video-box" v-else>
					<video id="myVideo" :src="detialData.videoSrc" controls></video>
				</view>
				<!-- 描述 -->
				<!-- <view class="u-flex-col u-m-t-10">
					<text class="u-font-30 u-m-b-10">{{detialData.title}}</text>
					<text>{{detialData.description}}</text>
				</view> -->
				<view class="u-flex btn-box" v-if="detialData.videoSrc">
					<u-button type="primary" size="medium"
						@click="handleDownloads(detialData.videoSrc,'video')">下载视频</u-button>
					<u-button type="primary" size="medium"
						@click="handleDownloads(detialData.imageSrc,'img')">下载封面</u-button>
					<u-button type="success" size="medium" @click="copy(detialData.videoSrc)">复制无水印视频链接</u-button>
					<u-button type="success" size="medium" @click="copy(detialData.imageSrc)">复制无水印封面链接</u-button>
				</view>
			</view>
		</view>
		<kxCustomer></kxCustomer>
	</view>
</template>
<script>
	// #ifdef MP-WEIXIN
	const fs = wx.getFileSystemManager()
	// #endif
	export default {
		data() {
			return {
				show: false,
				showTips: false,
				newTime: +new Date(),
				detialData: {}
			}
		},
		onLoad(e) {
			/* 插屏广告 */
			this.tools.wxAd('adunit-4396c45df56802c9')
			this.detialData = JSON.parse(decodeURIComponent(e.config));
		},
		methods: {
			jump() {
				uni.navigateTo({
					url: '/pages/my/dataLog/index?index=1'
				});
			},
			// 预览图片
			previewImage(i) {
				uni.previewImage({
					urls: this.detialData.imageAtlas,
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
			//处理解析后的数据
			handleDownloads(url, type) {
				var downloadTask = uni.downloadFile({
					url: url,
					success: (res) => {
						if (res.statusCode === 200) {
							if (type === 'img') this.saveImage(res.tempFilePath)
							if (type === 'video') this.handleVideoFile('video', `${this.newTime}.mp4`, res
								.tempFilePath)
						}
					},
					fail: (err) => {
						if (err.errMsg == 'downloadFile:fail exceed max file size') {
							uni.showModal({
								title: "下载失败",
								content: "该文件超过200M，无法直接在微信下载，请复制链接去外部浏览器下载",
								confirmText: "复制链接",
								showCancel: false
							})
							return
						}
						uni.showToast({
							title: "下载失败，请复制链接去浏览器下载",
							icon: "none"
						})
					}
				});
				downloadTask.onProgressUpdate((res) => {
					uni.showLoading({
						title: '正在下载' + res.progress + '%'
					})
					if (res.progress == 100) uni.hideLoading()
				});
			},
			// 保存图片
			saveImage(tempFilePath) {
				uni.saveImageToPhotosAlbum({
					filePath: tempFilePath,
					success: () => {
						uni.showToast({
							title: '已保存在手机相册中',
							icon: 'none',
						});
					},
					fail: (err) => {
						uni.showToast({
							title: '无法保存到手机,复制无水印视频链接',
							icon: 'none',
						});
					}
				})
			},

			//保存视频
			saveVideoFile(tempFilePath) {
				uni.saveVideoToPhotosAlbum({
					filePath: tempFilePath,
					success: (res) => {
						uni.showToast({
							title: '已保存在手机相册中',
							icon: 'none',
						});
					},
					fail: (err) => {
						if (err.errMsg == 'saveVideoToPhotosAlbum:fail cancel') return //取消，不做操作
						uni.showToast({
							title: '无法保存到手机,复制无水印视频链接',
							icon: 'none',
						});
					}
				})
			},
			//处理视频文件
			handleVideoFile(dir, filePath, tempFilePath) {
				this.handleDirectory(dir)
				this.$nextTick(() => {
					fs.saveFile({
						tempFilePath,
						filePath: `${wx.env.USER_DATA_PATH}/${dir+'/'+filePath}`,
						success: (res) => {
							this.saveVideoFile(res.savedFilePath)
						},
						fail(res) {
							uni.showToast({
								title: '下载失败，请查看是否打开下载相册权限或联系客服',
								icon: "none"
							})
						}
					})
				})
			},
			//处理文件目录
			handleDirectory(url) {
				//判断文件/目录是否存在
				try {
					fs.accessSync(`${wx.env.USER_DATA_PATH}/${url}`)
				} catch (e) {
					try {
						fs.mkdirSync(`${wx.env.USER_DATA_PATH}/${url}`, true)
					} catch (e) {
						console.error('fsmkdir', e)
					}
				}
			},
			//复制
			copy(text) {
				uni.setClipboardData({
					data: text,
					showToast: false,
					success: () => {
						uni.hideToast(); // 隐藏弹出提示
						uni.hideKeyboard(); //  隐藏软键盘
						uni.showToast({
							title: '已复制'
						})
					}
				});
			}
		}
	}
</script>

<style lang="scss" scoped>
	page {
		background-color: #f0f2f6;
	}

	.title-box {
		display: flex;
		align-items: center;
		height: 100rpx;
		padding: 0 16rpx !important;

		.backIcon {
			font-size: 40rpx;
			color: #000;
		}

		.title {
			flex: 1;
			text-align: center;
			padding-right: 40rpx;
			font-size: 32rpx;
		}
	}

	.content {
		background-color: #f0f2f6;

		.notice-bar-box {
			border-radius: 40rpx;
			margin-top: 20rpx;

			::v-deep uni-notice-bar {
				.uni-noticebar {
					border-radius: 80rpx;
				}
			}
		}

		.video-box {
			width: 100%;

			::v-deep video {
				width: 100%;
			}
		}

		.btn-box {
			flex-wrap: wrap;
			justify-content: space-between;
			margin-top: 20rpx;

			::v-deep button {
				width: 340rpx !important;
				margin-bottom: 20rpx;
			}
		}

		.imgs-box {
			border: 1px solid #d5d5d5;
			overflow-y: scroll;
			flex-wrap: wrap;
			justify-content: space-between;
			margin-top: 20rpx;
			padding: 20rpx;
			border-radius: 8rpx;

			.scroll-Y {
				max-height: 1000rpx;

				.scroll-box {
					flex-wrap: wrap;
					justify-content: space-between;
				}

				.img-item {
					position: relative;
					width: 48%;
					height: 460rpx;
					align-items: center;
					flex-wrap: wrap;
					justify-content: space-between;
					margin-bottom: 10rpx;

					.image-sty {
						width: 100%;
						height: 100%;
					}
				}

				.glare-effect {
					position: relative;
					width: 48%;
					height: 460rpx;
					/* 根据需要设置高度 */
					background-color: rgba(255, 255, 255, 0.8);
					/* 背景颜色 */
					backdrop-filter: blur(8rpx);
					/* 模糊度 */
					display: flex;
					align-items: center;
					justify-content: center;
					font-size: 36rpx;
					letter-spacing: 8rpx;
					color: orangered;


					&::before {
						content: '';
						position: absolute;
						top: 0;
						left: 0;
						right: 0;
						bottom: 0;
						// backdrop-filter: blur(2rpx);
						// background-color: rgba(255, 255, 255, 0.4);
					}
				}


			}
		}
	}





	.content {}
</style>