<template>
	<el-container>
		<el-aside width="300px">
			<el-container>
				<el-header>
					<el-input placeholder="输入关键字进行过滤" v-model="menuFilterText" clearable></el-input>
				</el-header>
				<el-main class="nopadding">
					<el-tree ref="menu" class="menu" node-key="name" :data="menuList" :props="menuProps" draggable highlight-current :expand-on-click-node="false" check-strictly show-checkbox :filter-node-method="menuFilterNode" @node-click="menuClick" @node-drop="nodeDrop">

						<template #default="{node, data}">
							<span class="custom-tree-node el-tree-node__label">
								<span class="label">{{ node.label }}</span>
								<span class="do">
									<i class="el-icon-plus" @click.stop="add(node, data)"></i>
								</span>
							</span>
						</template>

					</el-tree>
				</el-main>
				<el-footer style="height:51px;">
					<el-button type="primary" size="mini" icon="el-icon-plus" @click="add()"></el-button>
					<el-button type="danger" size="mini" plain icon="el-icon-delete" @click="delMenu"></el-button>
				</el-footer>
			</el-container>
		</el-aside>
		<el-container>
			<el-main class="nopadding" style="padding:20px;">
				<save ref="save" :menu="menuList"></save>
			</el-main>
		</el-container>
	</el-container>
</template>

<script>
	let newMenuIndex = 1;
	import save from './save'

	export default {
		name: "settingMenu",
		components: {
			save
		},
		data(){
			return {
				menuList: [],
				menuProps: {
					label: (data)=>{
						return data.meta.title
					}
				},
				menuFilterText: ""
			}
		},
		watch: {
			menuFilterText(val){
				this.$refs.menu.filter(val);
			}
		},
		mounted() {
			this.getMenu();
		},
		methods: {
			//加载树数据
			async getMenu(){
				var res = await this.$API.menu.list.get();
				this.menuList = res.data;
			},
			//树点击
			menuClick(data, node){
				var pid = node.level==1?undefined:node.parent.data.name;
				this.$refs.save.setData(data, pid)
			},
			//树过滤
			menuFilterNode(value, data){
				if (!value) return true;
				var targetText = data.meta.title;
				return targetText.indexOf(value) !== -1;
			},
			//树拖拽
			nodeDrop(draggingNode, dropNode, dropType){
				this.$message(`拖拽对象：${draggingNode.data.meta.title}, 释放对象：${dropNode.data.meta.title}, 释放对象的位置：${dropType}`)
			},
			//增加
			add(node){
				var newMenuName = "未命名" + newMenuIndex++;
				var newMenuData = {
					name: newMenuName,
					path: "",
					component: "",
					meta:{
						title: newMenuName,
						type: "menu"
					}
				}
				if(node){
					this.$refs.menu.append(newMenuData, node)
					var lastNode = node.childNodes[node.childNodes.length-1]
					this.$refs.menu.setCurrentKey(lastNode.data.name)
					var pid = node.data.name;
					this.$refs.save.setData(newMenuData, pid)
				}else{
					this.$refs.menu.append(newMenuData)
					var newNode = this.menuList[this.menuList.length-1]
					this.$refs.menu.setCurrentKey(newNode.name)
					this.$refs.save.setData(newMenuData)
				}

			},
			//删除菜单
			delMenu(){
				var CheckedNodes = this.$refs.menu.getCheckedNodes()
				if(CheckedNodes.length == 0){
					this.$message.warning("请选择需要删除的项")
					return false;
				}
				CheckedNodes.forEach(item => {
					this.$refs.menu.remove(item)
				})
			}
		}
	}
</script>

<style scoped>
	.custom-tree-node {display: flex;flex: 1;align-items: center;justify-content: space-between;font-size: 14px;padding-right: 24px;height:100%;}
	.custom-tree-node .label {display: flex;align-items: center;;height: 100%;}
	.custom-tree-node .label .el-tag {margin-left: 5px;}
	.custom-tree-node .do {display: none;}
	.custom-tree-node .do i {margin-left:5px;color: #999;padding:5px;}
	.custom-tree-node .do i:hover {color: #333;}

	.custom-tree-node:hover .do {display: inline-block;}
</style>
