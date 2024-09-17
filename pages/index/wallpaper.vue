<template>
	<view class="wallpaper">
		<mescroll-body ref="mescrollRef" @down="downCallback" :down="downOption" :sticky="false" @up="upCallback"
			:up="upOption" :bottombar="false" @init="mescrollInit" top="20">
			<view class="content">
				<view class="u-flex img-item-box">
					<view class="img-item u-flex" v-for="(item,index) in imgList" :key="index"
						@click="watermark(item.imgeObj)">
						<image :src="item.link" class="image-sty"></image>
						<u-button type="primary" size="mini" @click="watermark(item.imgeObj)"
							style="position: absolute;bottom: 8rpx;left: 8rpx;">点击查看详情</u-button>
					</view>
				</view>
			</view>
		</mescroll-body>
		<view class="com-addBtn" @click="add" v-if="isAdmin">
			<u-icon name="plus" size="48" color="#fff" />
		</view>
		<addImagesForm v-model="showAddImagesForm" v-if="showAddImagesForm" @confirm="confirm"></addImagesForm>
	</view>
</template>

<script>
	import {
		getVoucher,
		watermark,
		authorWorkWatermark
	} from "@/api/external.js";
	import MescrollMixin from "@/uni_modules/mescroll-uni/components/mescroll-uni/mescroll-mixins.js";
	const db = uniCloud.database();
	const depositTable = db.collection('wallpaper-list')
	import addImagesForm from '../components/addImagesForm.vue'
	export default {
		mixins: [MescrollMixin],
		components: {
			addImagesForm
		},
		data() {
			return {
				showAddImagesForm: false,
				imgList: [],
				downOption: {
					use: false,
					auto: false,
				},
				pageSize: 20,
				skipNumber: 1,
				upOption: {
					use: true,
					auto: true,
					page: {
						num: 0,
						size: 200,
						time: null,
					},
					empty: {
						use: true,
						icon: '',
						tip: "暂无数据",
						fixed: false,
						top: "560rpx",
					},
					textNoMore: "没有更多数据",
				},
			}
		},
		computed: {
			currentUser() {
				let hostUserInfo = uni.getStorageSync('uni-id-pages-userInfo') || {}
				return hostUserInfo._id
			},
			isAdmin() {
				return this.tools.isAdminRole()
			}
		},
		onLoad() {
			this.getVoucher()
			uni.$on('refresh', () => {
				this.imgList = [];
				this.mescroll.resetUpScroll();
			})
		},
		onShow() {
			this.imgList = [];
			this.mescroll.resetUpScroll();
		},
		methods: {
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
			add() {
				this.showAddImagesForm = true
			},
			//新增
			async confirm(data) {
				await depositTable.add({
					dateTimestamp: this.tools.convertToTimestamp(this.tools.getDate(new Date()).fullDate),
					date: this.tools.getDate(new Date()).fullDate,
					imgeObj: data
				})
			},
			//短视频解析
			watermark(item) {
				if (!item.link) return this.$u.toast("分享链接不能为空")
				let data = {
					link: item.link
				}
				watermark(data).then(res => {
					let data = JSON.parse(JSON.stringify(res.data)) || {}
					uni.navigateTo({
						url: '/pages/wallpaper/index?config=' + encodeURIComponent(JSON
							.stringify(data))
					})
				}).catch(err => {
					this.authorWorkWatermark(item)
				})
			},
			ensureHttps(url) {
				return url.replace(/^http:\/\//i, 'https://');
			},
			//批量解析
			authorWorkWatermark(item) {
				if (!item.link) return this.$u.toast("分享链接不能为空")
				let data = {
					appid: '66bc5fb2a5d7e1241SihJ',
					appsecret: '6B0TruSB7SvwczwF4vZ0iTiOXPZOcJST',
					link: item.link
				}
				authorWorkWatermark(data).then(res => {
					let data = JSON.parse(JSON.stringify(res.data)) || {}
					if (res.code == '1') {
						uni.navigateTo({
							url: '/pages/wallpaper/index?config=' + encodeURIComponent(JSON
								.stringify(data))
						})
					}
				})
			},
			upCallback(page) {
				let query = {
					/* 页数 */
					skipNumber: page.num - 1,
					/* 条数 */
					pageSize: 200
				};
				uniCloud.callFunction({
					name: 'getImgList',
					data: {
						skipNumber: query.skipNumber,
						pageSize: query.pageSize
					},
				}).then(res => {
					let list = res.result.data || []
					this.mescroll.endSuccess(list.length);
					if (page.num == 1) this.list = [];
					this.imgList = this.imgList.concat(list);
				});
			},
			init() {
				const skipNumber = (this.skipNumber - 1) * this.pageSize;
				uniCloud.callFunction({
					name: 'getImgList',
					data: {
						skipNumber: skipNumber,
						pageSize: this.pageSize
					},
				}).then(res => {
					let list = res.result.data || []
					this.mescroll.endSuccess(list.length);
					this.imgList = this.imgList.concat(list);
				});
			}
		}
	}
</script>

<style lang="scss">
	page {
		background-color: #09081a;
	}

	.wallpaper {
		padding: 0 20rpx 20rpx 20rpx;

		.content {

			.img-item-box {
				flex-wrap: wrap;
				justify-content: space-between;

				.img-item {
					position: relative;
					align-items: center;
					margin-bottom: 10rpx;
					width: 48%;
					height: 400rpx;
					border: 1px solid #fff;
					border-radius: 8rpx;

					.image-sty {
						width: 100%;
						height: 100%;

					}
				}
			}
		}
	}
</style>