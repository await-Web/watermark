<template>
	<view class="tool-v">
		<u-alert-tips :show="true" type="error" @close="showTips = false" title="主页链接请点击红色的批量解析按钮,目前仅支持抖音平台"
			:close-able="true"></u-alert-tips>
		<view class="tool-content">
			<view class="u-m-t-20 url-input">
				<uni-easyinput type="textarea" v-model="url" placeholder="此处粘贴视频分享链接" :clearable="true"></uni-easyinput>
				<view class="u-flex">
					<button class="u-m-t-16 btn" @click="watermark">单个解析</button>
					<button class="u-m-t-16 btn" type="warn" @click="url = ''">清空</button>
				</view>
			</view>
			<view class="u-m-t-20">
				<button type="primary" @click="jumWebview">这是教程</button>
			</view>
			<view class="batch u-m-t-20 u-m-b-20" @click="authorWorkWatermark">
				<text>主页批量解析</text>
			</view>
			<view class="share u-flex-col">
				<view class="u-flex share-inner">
					<button open-type="share">
						<image src="../../static/image/wx.png" mode=""></image>
						<text class="u-m-l-10 u-m-r-10 u-font-30">分享给好友，共同解锁有趣的视频</text>
					</button>
				</view>
			</view>
			<view class="u-m-t-20">
				<button type="primary" open-type="contact">不懂就问</button>
			</view>
			<view class="statement">视频归平台及作者所有，本应用不储存任何视频及图片</view>
		</view>
		<AnalysisDetial :detialData="detialData" v-model="showAnalysisDetial" v-if="showAnalysisDetial">
		</AnalysisDetial>
	</view>
</template>
<script>
	import {
		getVoucher,
		watermark,
		authorWorkWatermark
	} from "@/api/external.js";
	import AnalysisDetial from '../components/AnalysisDetial.vue'
	const subscribemsg = uniCloud.importObject('subscribeMessage')
	export default {
		components: {
			AnalysisDetial
		},
		data() {
			return {
				url: "",
				detialData: {},
				showAnalysisDetial: false,
				subscribeId: ['UU3SfNdbK8zevjVTLyDd43aqeGvdO4V6ND-VcoIRTYk']
			}
		},
		onShareAppMessage() {
			return {
				title: '免费去水印,不限次数',
				path: '/pages/index/index'
			}
		},
		computed: {
			currentUser() {
				let hostUserInfo = uni.getStorageSync('uni-id-pages-userInfo') || {}
				return hostUserInfo
			}
		},
		onLoad() {
			this.share()
			this.getVoucher()
		},
		onShareAppMessage() {
			return {
				title: '免费去水印,不限次数',
				path: '/pages/index/index'
			}
		},
		methods: {
			share() {
				//分享
				// #ifdef MP-WEIXIN
				wx.showShareMenu({
					withShareTicket: true,
					menus: ['shareAppMessage', 'shareTimeline']
				})
				// #endif
			},
			//获取接口调用凭据
			getVoucher() {
				let data = {
					appid: '66bc5fb2a5d7e1241SihJ',
					appsecret: '6B0TruSB7SvwczwF4vZ0iTiOXPZOcJST'
				}
				getVoucher(data).then(res => {
					uni.setStorageSync('externalToken', res.data.token) || ''
				})
			},
			//获取次数
			getWatermarkCount() {
				uniCloud.callFunction({
					name: 'getWatermark',
					data: {
						user_id: this.currentUser._id
					},
				}).then(res => {});
			},

			//短视频解析
			watermark() {
				//订阅
				// uni.requestSubscribeMessage({
				// 	tmplIds: this.subscribeId
				// });
				if (!this.url) return this.$u.toast("分享链接不能为空")
				let data = {
					link: this.url
				}
				watermark(data).then(res => {
					let data = JSON.parse(JSON.stringify(res.data)) || {}
					let imgUrl = this.ensureHttps(data.imageSrc)
					let videoUrl = this.ensureHttps(data.videoSrc)
					let imageAtlas = data.imageAtlas.map(o => this.ensureHttps(o))
					this.detialData = {
						...data,
						imageSrc: imgUrl,
						videoSrc: videoUrl,
						imageAtlas: imageAtlas
					}
					this.showAnalysisDetial = true
				}).catch(err => {})
			},
			jumWebview() {
				uni.navigateTo({
					url: '/pages/webview/index'
				});
			},
			//批量解析
			authorWorkWatermark() {
				if (!this.url) return this.$u.toast("分享链接不能为空")
				let data = {
					appid: '66bc5fb2a5d7e1241SihJ',
					appsecret: '6B0TruSB7SvwczwF4vZ0iTiOXPZOcJST',
					link: this.url
				}
				authorWorkWatermark(data).then(res => {
					let data = JSON.parse(JSON.stringify(res.data)) || {}
					if (res.code == '1') {
						uni.navigateTo({
							url: '/pages/batch/index?config=' + encodeURIComponent(JSON
								.stringify(data))
						})
					}
				})
			},
			ensureHttps(url) {
				return url.replace(/^http:\/\//i, 'https://');
			}
		}
	}
</script>

<style lang="scss" scoped>
	page {
		background-color: #f0f2f6;
	}

	.tool-v {
		width: 100%;
		height: 100%;
		padding: 0 20rpx;

		.tool-content {

			.batch {
				width: 100%;
				height: 120rpx;
				background-image: linear-gradient(45deg, #ff9700, #ed1c24);
				color: #ffffff;
				border-radius: 20rpx;
				text-align: center;
				line-height: 120rpx;
				font-size: 32rpx;
			}

			.url-input {
				.btn {
					width: 46%;
					background-color: #16afc3;
					color: #fff;
					font-size: 32rpx;
				}
			}

			.del-watermark {
				width: 100%;
				justify-content: space-between;
				margin-top: 20rpx;

				.block {
					width: 350rpx;
					height: 160rpx;
					background-color: blue;
					color: #fff;
					border-radius: 20rpx;
					align-items: center;
					justify-content: center;
					font-size: 28rpx;

					.txt-top {
						font-size: 46rpx;
					}

					.txt-bottom {
						font-size: 28rpx;
					}
				}

				.course {
					background-color: blue;
				}

				.invitation {
					background-color: red;
				}
			}

			.share {
				.share-inner {
					width: 100%;

					button {
						display: flex;
						width: 100% !important;
						height: 180rpx;
						border-radius: 20rpx;
						justify-content: center;
						align-items: center;
						color: #fff;
						background: radial-gradient(60% 200px at right top, #19b2bc, transparent),
							radial-gradient(20% 200px at left top, #0a96e4, transparent),
							radial-gradient(80% 200px at left top, #048af4, transparent);

						image {
							width: 84rpx;
							height: 84rpx;
						}
					}
				}

				width: 100%;
				height: 180rpx;
				margin-top: 20rpx;
				border-radius: 20rpx;
				background: radial-gradient(60% 200px at right top, #19b2bc, transparent),
				radial-gradient(20% 200px at left top, #0a96e4, transparent),
				radial-gradient(80% 200px at left top, #048af4, transparent);

				justify-content: center;
				align-items: center;
			}

			.statement {
				width: 100%;
				height: 80rpx;
				line-height: 80rpx;
				text-align: center;
				color: red;
			}
		}
	}
</style>