<template>
	<div v-if="pageLoading">
		<el-main>
			<el-card shadow="never">
				<el-skeleton :rows="1"></el-skeleton>
			</el-card>
			<el-card shadow="never" style="margin-top: 15px;">
				<el-skeleton></el-skeleton>
			</el-card>
		</el-main>
	</div>
	<work v-if="dashboard=='1'" @on-mounted="onMounted"></work>
	<stats v-else @on-mounted="onMounted"></stats>
</template>

<script>
	import { defineAsyncComponent } from 'vue';
	const stats = defineAsyncComponent(() => import('./stats'));
	const work = defineAsyncComponent(() => import('./work'));

	export default {
		name: "dashboard",
		components: {
			stats,
			work
		},
		data(){
			return {
				pageLoading: true,
				dashboard: '0'
			}
		},
		created(){
			this.dashboard = this.$TOOL.data.get("user").dashboard;
		},
		mounted(){

		},
		methods: {
			onMounted(){
				this.pageLoading = false
			}
		}
	}
</script>

<style>
</style>
