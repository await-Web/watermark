<script setup>
	import {
		onLaunch,
		onShow,
		onHide
	} from "@dcloudio/uni-app"
	import {
		useUserStore
	} from "@/store/user.js"
	const uniIdCo = uniCloud.importObject('uni-id-co')
	const userStore = useUserStore()
	onLaunch(() => {
		if ((uniCloud.getCurrentUserInfo().tokenExpired - Date.now()) < 1000 * 60 * 60 * 5) userStore.logout();
	})
	onShow(() => {
		console.log('App Show')
	})
	onHide(() => {
		console.log('App Hide')
	})
</script>

<style lang="scss">
	@import "@/uni_modules/vk-uview-ui/index.scss";
	@import "@/assets/scss/common-style.scss";
	@import "@/assets/iconfont/iconfont.css";
</style>