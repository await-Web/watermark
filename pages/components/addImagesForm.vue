<template>
	<u-popup ref="popup" width="100%" v-model="show">
		<view class="title-box">
			<uni-icons type="closeempty" style="float: left;margin-left: 20rpx;" size='24' @click="close"></uni-icons>
			<view class="title">添加解析链接</view>
		</view>
		<view class="u-p-l-20 u-p-r-20" style="background-color: #fff;">
			<u-form :model="dataForm" :errorType="['toast']" label-width="180" label-align="left" ref="dataForm">
				<u-form-item label="链接">
					<u-input input-align='right' v-model="dataForm.link" type="textarea" placeholder="请输入" />
				</u-form-item>
				<u-form-item label="标题">
					<u-input input-align='right' v-model="dataForm.title" placeholder="请输入" />
				</u-form-item>
				<u-form-item label="封面地址">
					<u-input input-align='right' v-model="dataForm.imageSrc" placeholder="请输入" />
				</u-form-item>
				<u-form-item label="图片集">
					<u-input input-align='right' v-model="dataForm.imageAtlas" type="textarea" placeholder="请输入" />
				</u-form-item>
			</u-form>
		</view>
		<view class="flowBefore-actions">
			<u-button class="buttom-btn" type="primary" @click.stop="getResult('confirm')">确定</u-button>
			<u-button class="buttom-btn" @click.stop="getResult('cancel')">取消</u-button>
		</view>
	</u-popup>
</template>

<script>
	import {
		getVoucher,
		watermark,
		authorWorkWatermark
	} from "@/api/external.js";
	export default {
		props: {
			modelValue: {
				type: Boolean,
				default: false
			}
		},
		data() {
			return {
				show: false,
				dataForm: {
					link: '',
					title: '',
					imageSrc: '',
					imageAtlas: ''
				},
				detialData: {}

			}
		},
		watch: {
			modelValue: {
				immediate: true,
				handler(val) {
					this.show = val
				}
			}
		},
		methods: {
			close() {
				this.$emit("update:modelValue", false);
				uni.$emit('refresh')
			},
			getResult(type) {
				if (type === 'cancel') return this.close()
				this.watermark()


			},
			//短视频解析
			watermark() {
				if (!this.dataForm.link) return this.$u.toast("分享链接不能为空")
				let data = {
					link: this.dataForm.link,
					appid: '66bc5fb2a5d7e1241SihJ',
					appsecret: '6B0TruSB7SvwczwF4vZ0iTiOXPZOcJST',
				}
				watermark(data).then(res => {
					let data = JSON.parse(JSON.stringify(res.data)) || {}
					this.dataForm.link = this.ensureHttps(data.link)
					this.dataForm.imageSrc = this.ensureHttps(data.imageSrc)
					this.close()
					this.$emit("confirm", this.dataForm);
				}).catch(err => {})
			},
			ensureHttps(url) {
				return url.replace(/^http:\/\//i, 'https://');
			}
		}
	}
</script>

<style lang="scss">
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
</style>