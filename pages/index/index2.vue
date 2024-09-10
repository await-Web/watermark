<template>
	<view class="index-v">
		<view class="">
			<home-head></home-head>
		</view>
		<view class="body">
			<view class="banner">
				<view class="banner-up">
					<view class="left">
						<view class="title-box">
							<view class="txt">今日存钱</view>
							<view class="txt money-symbol">￥：<text class="money-sty">{{todayTotalMoney}}</text>
							</view>
						</view>
						<view class="title-box">
							<view class="txt">累计存钱</view>
							<view class="txt">￥：<text class="money-sty">{{totalMoney}}</text></view>
						</view>
					</view>
					<view class="right">
						<image src="../../static/image/deposit.gif" mode="" class="right-image">
						</image>
					</view>
				</view>
				<view class="bottom-slogan">
					<!-- 距离暴富还有300天呦！加油宝子！你是最棒的! -->
					点击下方日历，开始存钱吧！
				</view>
			</view>

			<view class="" style="margin-top: 20rpx;">
				<uni-calendar class="uni-calendar--hook" :selected="calendar_data" :showMonth="false"
					@change="calendarChange" :start-date="tools.getYMD()" :endDate="tools.getYMD()" />
			</view>
		</view>
		<u-modal v-model="showModal" title="记录" @confirm="confirm">
			<view class="slot-content">
				<ZjfInputNumber v-model="money">
				</ZjfInputNumber>
			</view>
		</u-modal>
	</view>
</template>
<script>
	const db = uniCloud.database();
	const depositTable = db.collection('deposit-sign-in')
	import homeHead from '@/components/homeHead.vue'
	export default {
		components: {
			homeHead
		},
		data() {
			return {
				info: '',
				money: '',
				calendar_data: [],
				check_ins: [],
				todayTotalMoney: 0,
				totalMoney: 0,
				showModal: false
			}
		},
		computed: {
			currentUser() {
				let hostUserInfo = uni.getStorageSync('uni-id-pages-userInfo') || {}
				return hostUserInfo._id
			}
		},
		onLoad() {
			uni.hideToast(); // 隐藏弹出提示
			uni.hideKeyboard(); //  隐藏软键盘
		},
		onShow() {
			uni.hideToast(); // 隐藏弹出提示
			uni.hideKeyboard(); //  隐藏软键盘
			// #ifdef MP-WEIXIN
			wx.showShareMenu({
				withShareTicket: true,
				menus: ['shareAppMessage', 'shareTimeline']
			})
			// #endif
			this.init()
		},
		//分享
		onShareAppMessage() {
			return {
				title: '坚持存钱，不做月光族',
				path: '/pages/index/index',
				imageUrl: '/static/image/365.png'
			}
		},
		methods: {
			/* 初始化数据 */
			async init() {
				this.money = ''
				uni.showLoading({
					title: '正在加载',
					mask: true
				})
				uniCloud.callFunction({
					name: 'initData',
					data: {
						user_id: this.currentUser
					},
				}).then(res => {
					this.close()
					let data = res.result.data || []
					this.calendar_data = data
					this.handleDeposit()
				});
			},
			/* 弹窗 */
			calendarChange() {
				this.showModal = true
			},
			close() {
				this.showModal = false
			},
			//新增
			async addData() {
				this.check_ins.push({
					money: this.money,
					info: '已存钱'
				})
				await depositTable.add({
					dateTimestamp: this.tools.convertToTimestamp(this.tools.getDate(new Date()).fullDate),
					date: this.tools.getDate(new Date()).fullDate,
					info: '已存钱',
					calendar_data: this.check_ins
				})
				this.init()
			},
			/* 点击确认 */
			async confirm() {
				let num = Number(this.money) || 0
				if (num <= 0) return this.tools.toast('请输入大于等于0的数字')
				uni.showLoading({
					title: '正在存钱',
					mask: true
				})
				let isUpdate = false
				let curTimestamp = this.tools.convertToTimestamp(this.tools.getDate(new Date()).fullDate)
				let curData = this.calendar_data.filter(o => o.dateTimestamp === curTimestamp)
				if (curData.length) {
					let curId = curData[0].user_id
					uniCloud.callFunction({
						name: 'updateDeposit',
						data: {
							id: curId,
							money: this.money,
							curDate: curTimestamp,
							user_id: this.currentUser
						}
					}).then(() => {
						this.init()
					});
				} else {
					this.addData()
				}
				uni.hideLoading()
			},
			/* 处理今日，累计存钱 */
			handleDeposit() {
				let arr = this.calendar_data
				let totalMoney = 0
				for (let i = 0; i < arr.length; i++) {
					let calendar = arr[i].calendar_data;
					let today = arr[i].dateTimestamp;
					const sum = calendar.reduce((acc, obj) => acc + Number(obj.money), 0);
					if (today === this.tools.convertToTimestamp(this.tools.getDate(new Date()).fullDate)) this
						.todayTotalMoney = sum;
					totalMoney += sum;
				}
				this.totalMoney = totalMoney
				uni.hideLoading()
			}
		}
	}
</script>
<style lang="scss" scoped>
	.index-v {
		display: flex;
		flex-direction: column;
		/* #ifdef MP-WEIXIN */
		height: 100vh;
		/* #endif */
		/* #ifndef MP-WEIXIN */
		height: calc(100vh - 2.85rem);
		/* #endif */
		background-color: $background-color
	}

	.body {
		flex: 1;
		overflow: hidden;
		padding: 0rpx 20rpx 20rpx 20rpx;

		.banner {
			width: 100%;
			height: 300rpx;
			background-color: #b1befc;
			border-radius: 20rpx;
			padding: 0 20rpx;

			.banner-up {
				display: flex;
				justify-content: space-between;
				align-items: center;

				.left {
					.title-box {
						.txt {
							color: #fff;
							font-size: 28rpx;

							.money-sty {
								font-size: 32rpx;
								color: #ecef67;
							}

						}

						.money-symbol {
							margin-bottom: 28rpx;
						}
					}
				}

				.right {
					width: 220rpx;
					height: 220rpx;

					.right-image {
						width: 100%;
						height: 100%;
					}
				}
			}

			.bottom-slogan {
				margin-top: 20rpx;
				color: #fffc80;
				font-size: 34rpx;
				text-align: center;
				font-weight: bold;
				font-family: cursive;
			}
		}
	}

	.slot-content {
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 40rpx 0;
	}
</style>