<template>
	<view class="page_v">
		<uni-section title="个人资料" type="line">
			<view class="example">
				<!-- 基础用法，不包含校验规则 -->
				<uni-forms ref="form" :modelValue="formData" label-position="top" :rules="rules">
					<uni-forms-item label="昵称" required name="username">
						<uni-easyinput v-model="formData.username" placeholder="请输入姓名" />
					</uni-forms-item>
					<uni-forms-item label="年龄" required>
						<uni-easyinput v-model="formData.age" placeholder="请输入年龄" name="age" />
					</uni-forms-item>
					<uni-forms-item label="性别" required>
						<uni-data-checkbox v-model="formData.gender" :localdata="genderList" name="gender" />
					</uni-forms-item>
					<!-- <uni-forms-item label="选择存钱类型" required label-width="200">
						<ZjfSelect :options="depositTypeList" v-model="formData.depositType" @change="selectChange"
							name="depositType">
						</ZjfSelect>
					</uni-forms-item> -->
				</uni-forms>
				<!-- <view
					style="width: 100%;height: 100%;border-radius: 8rpx;background-color: #d9d9d8;margin-bottom: 20rpx;padding: 20rpx;">
					<text style="color: #333;font-size: 36rpx; font-weight: 600;"
						class="title">{{depositValue+'说明'}}</text>
					<view class="" style="margin-top: 20rpx;letter-spacing: 2px;line-height: 56rpx;font-size: 32rpx;">
						{{this.illustrate}}
					</view>
				</view> -->
			</view>
			<view class="" style="position: fixed;bottom: 0;width: 100%;">
				<button type="primary" @click="submit()" :loading="loading">提交</button>
			</view>
		</uni-section>
	</view>
</template>
<script>
	import {
		useUserStore
	} from "@/store/user.js"
	const userStore = useUserStore()
	export default {
		data() {
			return {
				formData: {
					username: '',
					age: "28",
					gender: 0,
					depositType: "0"
				},
				show: false,
				depositValue: '',
				loading: false,
				illustrate: '第1天存1元，第2天存2元，依次类推。每递增1元，第365天存365元。1年下来能存下66795元。适合人群:任何人。',
				depositTypeList: [{
					fullName: '365存钱法',
					id: '0',
					illustrate: '第1天存1元，第2天存2元，依次类推。每递增1元，第365天存365元。1年下来能存下66795元。适合人群:任何人。'
				}, {
					fullName: '333存钱法',
					id: '1',
					illustrate: '每个月收入分成3份。1份开支、1份储蓄、1份投资理财，比例可以是333、631、532。一定要先存钱再消费。适合人群:任何人。'
				}, {
					fullName: '存零法',
					id: '2',
					illustrate: '每天晚上回家把身上的100元整数倍之外的所有零钱存起来!比如你身上有158元钱就存58元,有329元就存29元只留100元的整数倍的钱。适合人群:现金族'
				}, {
					fullName: '52周存钱法',
					id: '3',
					illustrate: '第1周存10元，第2周20元，依次类推，每周递增10元。第52周存520元。52周刚好一年，一年就能存13780元。适合人群:月光族，学生党、刚参加工作的职场新人，tou资小白，理财新手。'
				}, {
					fullName: '星期存钱法',
					id: '4',
					illustrate: '星期一存10元，星期二存20元，依次递增到星期天存70元。每周可存下280元1年可以存下14560元。适用人群:学生党，职场人士。'
				}, {
					fullName: '10%强制存钱法',
					id: '5',
					illustrate: '每个月发工资后，强制存下工资的10%。存钱比例可以根据自身情况增加或降低。目的是无痛存钱，先行动起来。慢慢养成存钱的习惯。适合人群:月光族、学生党和低收入人群存钱很痛苦的人群。'
				}, {
					fullName: '30天倒数法',
					id: '6',
					illustrate: '每月1号存30元，2号存29元。依次递减，到30号存1元。每个月这样存。1年可以存下5580元。适合人群:青少年儿童、学生党、小白'
				}],
				genderList: [{
					text: '男',
					value: 0
				}, {
					text: '女',
					value: 1
				}, {
					text: '保密',
					value: 2
				}, {
					text: '其他',
					value: 3
				}],
				rules: {
					username: {
						rules: [{
							required: true,
							errorMessage: '请填写姓名',
						}]
					},
					age: {
						rules: [{
							required: true,
							errorMessage: '请填写年龄',
						}, {
							validateFunction: (rule, value, data, callback) => {
								// 异步需要返回 Promise 对象
								return new Promise((resolve, reject) => {
									setTimeout(() => {
										if (value >= 18) {
											// 通过返回 resolve
											resolve()
										} else {
											// 不通过返回 reject(new Error('错误信息'))
											reject(new Error('年龄必须大于18岁'))
										}
									}, 2000)
								})
							}
						}]
					},
					gender: {
						rules: [{
							required: true,
							errorMessage: '请选择性别',
						}]
					},
					depositType: {
						rules: [{
							required: true,
							errorMessage: '请选择存钱类型',
						}]
					},
				}
			}
		},
		onReady() {
			// 需要在onReady中设置规则
			this.$nextTick(() => {
				this.$refs.form.setRules(this.rules)
			})

		},
		onShow() {
			this.init()
		},
		computed: {
			userInfo() {
				return userStore.userInfo
			}
		},
		methods: {
			/* 初始化 */
			init() {
				this.formData = {
					'username': this.userInfo.username,
					'age': this.userInfo?.age || "28",
					'depositType': this.userInfo?.depositType || "0",
					'gender': this.userInfo?.gender || 1
				}
			},
			/* 选择存钱类型 */
			selectChange(e) {
				this.formData.depositType = e.id
				this.depositValue = e.fullName || ''
				this.illustrate = e?.illustrate || ''
			},
			/* 更新信息 */
			submit() {
				this.$refs.form.validate().then(res => {
					userStore.updateUserInfo(this.formData)
					setTimeout(() => {
						uni.navigateBack()
					}, 1000)
				})
			}
		}
	}
</script>

<style lang="scss">
	.page_v {
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

	.example {
		padding: 15px;
		background-color: #fff;
	}

	.segmented-control {
		margin-bottom: 15px;
	}

	.button-group {
		margin-top: 15px;
		display: flex;
		justify-content: space-around;
	}

	.form-item {
		display: flex;
		align-items: center;
	}

	.button {
		display: flex;
		align-items: center;
		height: 35px;
		margin-left: 10px;
	}
</style>