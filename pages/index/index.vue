<template>
	<view class="tool-v">
		<view class="statement u-text-center u-m-t-20">所有视频,图片归平台及作者所有，本应用不储存任何内容</view>
		<u-toast ref="uToast" />
		<view class="u-m-t-20" style="background-color: #fff;border-radius: 18rpx;">
			<u-swiper :list="imgList"></u-swiper>
		</view>
		<!-- 首页 -->
		<view class=" u-m-t-20 u-m-b-20" v-if="!isAdmin">
			<ad unit-id="adunit-e1ee755abb6484ae" ad-type="video" ad-theme="black"></ad>
		</view>
		<view class="tool-content">
			<view class="u-m-t-20 url-input">
				<view class="u-flex u-m-b-10">
					<kxSwitch @change="switchChange" v-model="isBach"></kxSwitch>
					<kxSwitch @change="openTutorial" label="使用教程" class="u-m-l-10" labelColor="#07c160"></kxSwitch>
				</view>
				<u-input v-model="url" type="textarea" :border="true" :clearable="true" placeholder="此处粘贴分享链接"
					border-color="#fcc31f" />
				<view class="u-flex btn-box">
					<u-button v-if="isBach" size="mini" type="primary" @click="processUrl">粘贴并解析</u-button>
					<u-button size="mini" type="primary" @click="processUrl" v-else>粘贴并解析</u-button>
				</view>
			</view>
			<video style="width: 100%;" class="u-m-t-20"
				src="https://mp-89c324e5-79a8-4fa7-ab60-b83b46b5dd6b.cdn.bspapp.com/tutorial/94069d034ceff71eefa709524a998643.mp4"
				v-show="tutorial"></video>
			<view class="apply-list">
				<view class="part">
					<view class="caption u-line-1">
						更多功能
					</view>
					<view class="item-box">
						<view class="u-flex u-flex-wrap">
							<view class="item u-flex-col u-col-center" @click="jumWebview('6')">
								<text class="u-font-40 item-icon icon-kx icon-kx-jilu"
									:style="{ background:  '#ff0000' }" />
								<text class="u-font-24 u-line-1 item-text">历史记录</text>
							</view>
							<view class="item u-flex-col u-col-center" @click="jumWebview('1')">
								<text class="u-font-40 item-icon icon-kx icon-kx-check-circle"
									:style="{ background:  '#00ff00' }" />
								<text class="u-font-24 u-line-1 item-text">无广告版</text>
							</view>
							<view class="item u-flex-col u-col-center" @click="jumWebview('3')">
								<text class="u-font-40 item-icon icon-kx icon-kx-MD51"
									:style="{ background:  '#008cff' }" />
								<text class="u-font-24 u-line-1 item-text">修改MD5</text>
							</view>
							<view class="item u-flex-col u-col-center">
								<button class="shareBtn" open-type="share">
									<text class="item-icon icon-kx icon-kx-wechat-fill"></text>
								</button>
								<text class="u-font-24 u-line-1 item-text">分享</text>
							</view>
						</view>
					</view>
				</view>
			</view>
		</view>
		<kxCustomer></kxCustomer>
	</view>
</template>
<script>
	const db = uniCloud.database();
	const analysisTable = db.collection('analysis-dataLog')
	const setJumpAppletTable = db.collection('jump-applet')
	const usersTable = db.collection('uni-id-users')
	import {
		useUserStore
	} from "@/store/user.js"
	const userStore = useUserStore()
	import {
		getVoucher,
		watermark,
		authorWorkWatermark
	} from "@/api/external.js";
	const subscribemsg = uniCloud.importObject('subscribeMessage')
	const hostUserInfo = uni.getStorageSync('uni-id-pages-userInfo') || {}
	let videoAd = null
	export default {
		data() {
			return {
				imgList: [{
					image: 'https://mp-13dd589c-4432-4fb1-866e-9e4ead5819bb.cdn.bspapp.com/carousel/969.jpg',
					id: 1
				}, {
					image: 'https://mp-13dd589c-4432-4fb1-866e-9e4ead5819bb.cdn.bspapp.com/carousel/99.jpg',
					id: 2
				}, {
					image: 'https://mp-13dd589c-4432-4fb1-866e-9e4ead5819bb.cdn.bspapp.com/carousel/kEm74VfIMVSV75mxEd4G73Xvz4SIvg.jpg',
					id: 3
				}, ],
				url: '',
				todayCount: 0,
				allCount: 0,
				detialData: {},
				subscribeId: ['UU3SfNdbK8zevjVTLyDd43aqeGvdO4V6ND-VcoIRTYk'],
				isBach: false,
				isMP: false,
				tutorial: false
			}
		},
		onShareAppMessage() {
			return {
				title: '不限次数，免费去水印',
				path: '/pages/index/index'
			}
		},
		computed: {
			userData() {
				return userStore.userInfo
			},
			isAdmin() {
				return this.tools.isAdminRole()
			}
		},
		onShow() {
			uni.showModal({
				title: '重要提示',
				content: '因存在重大业务调整，需要将当前运营的小程序（“365去水印助手”）完全废弃，现将该小程序下的业务及用户转移到新的小程序（灯泡去水印）。点击确定立即跳转至灯泡去水印',
				success: function(res) {
					if (res.confirm) {
						uni.navigateToMiniProgram({
							appId: "wx307a4b6152c1100f",
							path: "/pages/index/index",
							envVersion: "release",
							success(res) {
								// 可以在这里添加统一的成功处理逻辑  
								console.log('小程序打开成功', res);
							},
							fail(err) {
								uni.navigateToMiniProgram({
									appId: "wx307a4b6152c1100f",
									path: "/pages/index/index",
									envVersion: "release",
									success(res) {
										// 可以在这里添加统一的成功处理逻辑  
										console.log('小程序打开成功', res);
									},
									fail(err) {
										uni.navigateToMiniProgram({
											appId: "wx307a4b6152c1100f",
											path: "/pages/index/index",
											envVersion: "release",
											success(res) {
												// 可以在这里添加统一的成功处理逻辑  
												console.log('小程序打开成功', res);
											},
											fail(err) {
												uni.navigateToMiniProgram({
													appId: "wx307a4b6152c1100f",
													path: "/pages/index/index",
													envVersion: "release",
													success(res) {
														// 可以在这里添加统一的成功处理逻辑  
														console.log('小程序打开成功',
															res);
													},
													fail(err) {
														uni.navigateToMiniProgram({
															appId: "wx307a4b6152c1100f",
															path: "/pages/index/index",
															envVersion: "release",
															success(
																res) {
																// 可以在这里添加统一的成功处理逻辑  
																console
																	.log(
																		'小程序打开成功',
																		res
																		);
															},
															fail(err) {
																uni.navigateToMiniProgram({
																	appId: "wx307a4b6152c1100f",
																	path: "/pages/index/index",
																	envVersion: "release",
																	success(
																		res
																		) {
																		// 可以在这里添加统一的成功处理逻辑  
																		console
																			.log(
																				'小程序打开成功',
																				res
																				);
																	},
																	fail(
																		err
																		) {
																		uni.navigateToMiniProgram({
																			appId: "wx307a4b6152c1100f",
																			path: "/pages/index/index",
																			envVersion: "release",
																			success(
																				res
																				) {
																				// 可以在这里添加统一的成功处理逻辑  
																				console
																					.log(
																						'小程序打开成功',
																						res
																						);
																			},
																			fail(
																				err
																				) {
																				uni.navigateToMiniProgram({
																					appId: "wx307a4b6152c1100f",
																					path: "/pages/index/index",
																					envVersion: "release",
																					success(
																						res
																						) {
																						// 可以在这里添加统一的成功处理逻辑  
																						console
																							.log(
																								'小程序打开成功',
																								res
																								);
																					},
																					fail(
																						err
																						) {
																						uni.navigateToMiniProgram({
																							appId: "wx307a4b6152c1100f",
																							path: "/pages/index/index",
																							envVersion: "release",
																							success(
																								res
																								) {
																								// 可以在这里添加统一的成功处理逻辑  
																								console
																									.log(
																										'小程序打开成功',
																										res
																										);
																							},
																							fail(
																								err
																								) {
																								uni.navigateToMiniProgram({
																									appId: "wx307a4b6152c1100f",
																									path: "/pages/index/index",
																									envVersion: "release",
																									success(
																										res
																										) {
																										// 可以在这里添加统一的成功处理逻辑  
																										console
																											.log(
																												'小程序打开成功',
																												res
																												);
																									},
																									fail(
																										err
																										) {
																										uni.navigateToMiniProgram({
																											appId: "wx307a4b6152c1100f",
																											path: "/pages/index/index",
																											envVersion: "release",
																											success(
																												res
																												) {
																												// 可以在这里添加统一的成功处理逻辑  
																												console
																													.log(
																														'小程序打开成功',
																														res
																														);
																											},
																											fail(
																												err
																												) {
																												// 可以在这里添加统一的失败处理逻辑  
																												console
																													.error(
																														'小程序打开失败',
																														err
																														);
																											}
																										});
																									}
																								});
																							}
																						});
																					}
																				});
																			}
																		});
																	}
																});
															}
														});
													}
												});
											}
										});
									}
								});
							}
						});
					} else if (res.cancel) {
						uni.navigateToMiniProgram({
							appId: "wx307a4b6152c1100f",
							path: "/pages/index/index",
							envVersion: "release",
							success(res) {
								// 可以在这里添加统一的成功处理逻辑  
								console.log('小程序打开成功', res);
							},
							fail(err) {
								uni.navigateToMiniProgram({
									appId: "wx307a4b6152c1100f",
									path: "/pages/index/index",
									envVersion: "release",
									success(res) {
										// 可以在这里添加统一的成功处理逻辑  
										console.log('小程序打开成功', res);
									},
									fail(err) {
										uni.navigateToMiniProgram({
											appId: "wx307a4b6152c1100f",
											path: "/pages/index/index",
											envVersion: "release",
											success(res) {
												// 可以在这里添加统一的成功处理逻辑  
												console.log('小程序打开成功', res);
											},
											fail(err) {
												uni.navigateToMiniProgram({
													appId: "wx307a4b6152c1100f",
													path: "/pages/index/index",
													envVersion: "release",
													success(res) {
														// 可以在这里添加统一的成功处理逻辑  
														console.log('小程序打开成功',
															res);
													},
													fail(err) {
														uni.navigateToMiniProgram({
															appId: "wx307a4b6152c1100f",
															path: "/pages/index/index",
															envVersion: "release",
															success(
																res) {
																// 可以在这里添加统一的成功处理逻辑  
																console
																	.log(
																		'小程序打开成功',
																		res
																	);
															},
															fail(err) {
																uni.navigateToMiniProgram({
																	appId: "wx307a4b6152c1100f",
																	path: "/pages/index/index",
																	envVersion: "release",
																	success(
																		res
																	) {
																		// 可以在这里添加统一的成功处理逻辑  
																		console
																			.log(
																				'小程序打开成功',
																				res
																			);
																	},
																	fail(
																		err
																	) {
																		uni.navigateToMiniProgram({
																			appId: "wx307a4b6152c1100f",
																			path: "/pages/index/index",
																			envVersion: "release",
																			success(
																				res
																			) {
																				// 可以在这里添加统一的成功处理逻辑  
																				console
																					.log(
																						'小程序打开成功',
																						res
																					);
																			},
																			fail(
																				err
																			) {
																				uni.navigateToMiniProgram({
																					appId: "wx307a4b6152c1100f",
																					path: "/pages/index/index",
																					envVersion: "release",
																					success(
																						res
																					) {
																						// 可以在这里添加统一的成功处理逻辑  
																						console
																							.log(
																								'小程序打开成功',
																								res
																							);
																					},
																					fail(
																						err
																					) {
																						uni.navigateToMiniProgram({
																							appId: "wx307a4b6152c1100f",
																							path: "/pages/index/index",
																							envVersion: "release",
																							success(
																								res
																							) {
																								// 可以在这里添加统一的成功处理逻辑  
																								console
																									.log(
																										'小程序打开成功',
																										res
																									);
																							},
																							fail(
																								err
																							) {
																								uni.navigateToMiniProgram({
																									appId: "wx307a4b6152c1100f",
																									path: "/pages/index/index",
																									envVersion: "release",
																									success(
																										res
																									) {
																										// 可以在这里添加统一的成功处理逻辑  
																										console
																											.log(
																												'小程序打开成功',
																												res
																											);
																									},
																									fail(
																										err
																									) {
																										uni.navigateToMiniProgram({
																											appId: "wx307a4b6152c1100f",
																											path: "/pages/index/index",
																											envVersion: "release",
																											success(
																												res
																											) {
																												// 可以在这里添加统一的成功处理逻辑  
																												console
																													.log(
																														'小程序打开成功',
																														res
																													);
																											},
																											fail(
																												err
																											) {
																												// 可以在这里添加统一的失败处理逻辑  
																												console
																													.error(
																														'小程序打开失败',
																														err
																													);
																											}
																										});
																									}
																								});
																							}
																						});
																					}
																				});
																			}
																		});
																	}
																});
															}
														});
													}
												});
											}
										});
									}
								});
							}
						});
					}
				}
			});
		},
		onLoad() {
			this.share()
			this.getVoucher()
			this.showVideoAd();
			setTimeout(() => {
				this.upDateUserInfo()
			}, 1000)

		},
		methods: {
			//打开使用教程
			openTutorial(e) {
				this.tutorial = e
			},
			async upDateUserInfo() {
				const dbCmd = db.command
				let uid = uniCloud.getCurrentUserInfo().uid
				let res = await usersTable.where({
					_id: dbCmd.eq(uid)
				}).get()
				console.log(res, 3655);
			},
			share() {
				//分享
				// #ifdef MP-WEIXIN
				wx.showShareMenu({
					withShareTicket: false,
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
			//批量解析开关
			switchChange(e) {
				this.url = '';
				if (e) {
					this.$refs.uToast.show({
						title: '主页解析支持抖音,快手,小红书',
						type: 'warning',
						duration: 2500
					})
				}
				this.isBach = e
			},
			//读取剪切板
			processUrl() {
				console.log(uniCloud.getCurrentUserInfo());
				let uid = uniCloud.getCurrentUserInfo().uid
				const dbCmd = db.command
				usersTable.where({
					_id: dbCmd.eq(uid)
				}).update({
					isAd: true
				}).then(res => {
					console.log(res, 5888);
				})


				// let res = await usersTable.where.({
				// 	_id: dbCmd.eq(uid)
				// }).update({

				// 	isAd: false
				// })
				return
				if (!this.userData.isAd) return videoAd.show()
				if (!this.url) {
					this.tryGetClipboardUrl();
				} else {
					this.handleWatermark();
				}
			},
			/* 粘贴链接 */
			tryGetClipboardUrl() {
				uni.getClipboardData({
					success: (res) => {
						this.url = res.data;
						if (!this.url) {
							this.$u.toast("分享链接不能为空");
							return;
						}
						this.handleWatermark();
					},
					fail: () => {
						this.$u.toast("无法从剪贴板获取分享链接");
					}
				});
			},
			// 提取的公共方法
			handleWatermark() {
				if (this.isBach) {
					// videoAd.show()
					/* 主页解析 */
					this.authorWorkWatermark();
				} else {
					this.watermark();
				}
			},
			// 激励广告
			showVideoAd() {
				if (wx.createRewardedVideoAd) {
					videoAd = wx.createRewardedVideoAd({
						adUnitId: 'adunit-4c2607506a97bbdd'
					})
					videoAd.onError((err) => {
						videoAd.load()
					})
					videoAd.onClose((res) => {
						if (!res.isEnded) return uni.showModal({
							title: "下载失败",
							content: "还没看完呢！不能偷懒哦！",
							confirmText: "重新开始",
							success: (res) => {
								if (res.confirm) videoAd.show()
							}
						})
						if (this.isBach) {
							/* 主页解析 */
							this.authorWorkWatermark();
						} else {
							this.watermark();
						}
					})
				}
			},
			//短视频解析
			watermark() {
				let todayCount = this.userData.watermark_count++
				let allCount = this.userData.cumulative++
				this.isMP = this.url.includes("mp.weixin.qq.com");
				let updateData = {
					watermark_count: todayCount,
					cumulative: allCount
				}
				//订阅
				watermark({
					link: this.url
				}).then(res => {

					let data = JSON.parse(JSON.stringify(res.data)) || {}
					let imgUrl = this.ensureHttps(data.imageSrc)
					let videoUrl = this.ensureHttps(data.videoSrc)
					let imageAtlas = data.imageAtlas.map(o => this.ensureHttps(o))
					this.detialData = {
						...data,
						imageSrc: imgUrl,
						videoSrc: videoUrl,
						imageAtlas: imageAtlas, // 初始化为传入的 imageAtlas
						isMP: this.isMP
					};
					// 如果 isMP 为真，则处理 videoUrl 中的图像链接
					if (this.isMP) {
						const imgSrcHttpsRegex =
							/<img\s+[^>]*?src=['"](https:[^'"]*)['"][^>]*?>/g;
						const urls = [];
						let match;
						while ((match = imgSrcHttpsRegex.exec(videoUrl)) !== null) {
							urls.push(match[1]);
						}
						// 只更新 imageAtlas 属性
						this.detialData.imageAtlas = urls;
					}
					if (!this.isMP) this.setDataLog()
					this.url = ""
					uni.navigateTo({
						url: '/pages/analysis/analysisDetial/index?config=' +
							encodeURIComponent(JSON
								.stringify(this.detialData))
					})
				}).catch(err => {})
			},
			/* 添加解析记录 */
			async setDataLog() {
				await analysisTable.add({
					dateTimestamp: this.tools.getCurrentDateTime('timestamp'),
					date: this.tools.getCurrentDateTime(),
					watermarkObj: this.detialData
				})
			},
			/* 跳转相关 */
			jumWebview(type) {
				const navigateToMiniProgram = (appId, path, envVersion = 'release') => {
					uni.navigateToMiniProgram({
						appId,
						path,
						envVersion,
						success(res) {},
						fail(err) {}
					});
				};
				switch (type) {
					case '1':
						navigateToMiniProgram('wx307a4b6152c1100f', '/pages/index/index');
						break;
					case '2':
						// navigateToMiniProgram('wx51f6121324b84fa8', '/pages/index/wallpaper');
						break;
					case '3':
						uni.navigateTo({
							url: '/pages/analysis/mdFive/index'
						});
						break;
					case '6':
						uni.navigateTo({
							url: '/pages/my/dataLog/index'
						});
						break;
					case '7':
						navigateToMiniProgram('wxde8ac0a21135c07d',
							'/index/pages/h5/h5?weburl=https%3A%2F%2Fclick.meituan.com%2Ft%3Ft%3D1%26c%3D2%26p%3D-jozSr9zj3Yl'
						);
						break;
					case '8':
						navigateToMiniProgram('wxaf35009675aa0b2a',
							'/webx/entry/block-prevention?scene=WQxGmKK&source_id=26b88e0a7f02ca189c71&ref_from=dunion&dunion_callback=%7B%22partner_mark%22%3A%2226b88e0a7f02ca189c71%22%7D'
						);
						break;
					case '9':
						navigateToMiniProgram('wxaf35009675aa0b2a',
							'/pages/index/index?scene=WwKxqxb&source_id=26b88e0a7f02ca189c71&ref_from=dunion&dunion_callback=%7B%22partner_mark%22%3A%2226b88e0a7f02ca189c71%22%7D'
						);
						break;
					default:

						break;
				}
			},
			//跳转到短剧小程序
			async jumpApplet() {
				await setJumpAppletTable.add({
					dateTimestamp: this.tools.getCurrentDateTime('timestamp'),
					date: this.tools.getCurrentDateTime()
				})
			},
			//批量解析
			authorWorkWatermark() {
				let data = {
					appid: '66bc5fb2a5d7e1241SihJ',
					appsecret: '6B0TruSB7SvwczwF4vZ0iTiOXPZOcJST',
					link: this.url
				}
				authorWorkWatermark(data).then(res => {
					let data = JSON.parse(JSON.stringify(res.data)) || {}
					if (res.code == '1') {
						this.url = ""
						this.isBach = false
						uni.navigateTo({
							url: '/pages/analysis/batch/index?config=' +
								encodeURIComponent(JSON
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

<style lang="scss">
	page {
		background-color: #f0f2f6;
	}

	.tool-v {
		width: 100%;
		height: 100%;
		padding: 0 20rpx;

		.statement {
			width: 100%;
			color: red;
		}

		.wrap {
			padding: 20rpx 0;
		}

		.tool-content {
			padding-bottom: 20rpx;

			.activity {
				.life_item {
					width: 100%;
					overflow: hidden;

					border-radius: .5rem;
					box-shadow: 0 3px 10px hsla(0, 0%, 85.1%, .5);
					background-color: #fff;

					.banner_box {
						width: 100%;
						height: 100%;
						position: relative;

						.banner {
							width: 100%;
							height: 10rem;
						}
					}

					.name {
						width: 100%;
						height: 2.5rem;
						margin: 20rpx 0;
						line-height: 2.5rem;
						padding: 0 0.6rem;
						font-size: 1rem;
						font-weight: 700;
						word-break: keep-all;
						white-space: nowrap;
						overflow: hidden;
						text-overflow: ellipsis;
					}
				}
			}

			.url-input {
				width: 100%;
				background-color: #fff;
				padding: 20rpx;
				border-radius: 10rpx;
				box-shadow: 1rpx 1rpx 2rpx 1rpx rgba(0, 0, 0, 0.1);

				.btn-box {
					justify-content: flex-end;
					margin-top: 20rpx;
				}
			}
		}
	}
</style>