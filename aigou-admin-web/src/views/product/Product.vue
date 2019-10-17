<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="关键字"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getProducts">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="products" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" width="50">
			</el-table-column>
			<el-table-column prop="name" label="标题" width="150" sortable>
			</el-table-column>
			<el-table-column prop="subName" label="副标题" width="120" sortable>
			</el-table-column>
			<el-table-column prop="productType.name" label="商品类型" width="120" sortable>
			</el-table-column>
			<el-table-column prop="brand.name" label="品牌" width="120" sortable>
			</el-table-column>
			<el-table-column prop="onSaleTime" label="上架时间" width="115" sortable :formatter="formatOnSaleTime">
			</el-table-column>
			<el-table-column prop="offSaleTime" label="下架时间" width="115" sortable :formatter="formatOffSaleTime">
			</el-table-column>
			<el-table-column prop="state" label="状态" min-width="100" sortable>
				<template scope="scope">
					<span style="color: green" v-if="scope.row.state==1">上架</span>
					<span style="color: red" v-else>下架</span>
				</template>
			</el-table-column>
			<el-table-column label="操作" width="150">
				<template scope="scope">
					<el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
					<el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="rows" :total="total" style="float:right;">
			</el-pagination>
		</el-col>

		<!--编辑界面-->
		<el-dialog title="编辑" v-model="editFormVisible" :close-on-click-modal="false">
			<el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
				<el-form-item label="标题" prop="name">
					<el-input v-model="editForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="副标题" prop="subName">
					<el-input v-model="editForm.subName"></el-input>
				</el-form-item>
				<div class="block" style="margin-left: 15px">
					<span class="demonstration">商品类型</span>
					<el-cascader
							:show-all-levels="false"
							@change="handleChange"
							:change-on-select="true"
							:props="defaultParams"
							placeholder='请选择商品类型'
							:options="options"
							v-model="productOptions"
							:clearable="true">
					</el-cascader>
				</div>
				<el-form-item label="媒体属性" prop="logo">
					<template>
						<el-upload
								class="upload-demo"
								action="http://172.16.4.5:9527/services/common/file"
								list-type="picture"
								:on-remove="handleRemove"
								:on-success="handleSuccess"
								:before-upload="handleBeforeUpload"
								:file-list="fileList"
						>
							<el-button size="small" type="primary">点击上传</el-button>
						</el-upload>
					</template>
				</el-form-item>
				<el-form-item label="商品描述">
					<el-input type="textarea" v-model="editForm.description"></el-input>
				</el-form-item>
				<el-form-item label="商品详情">
					<el-input type="textarea" v-model="editForm.richContent"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="editFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--新增界面-->
		<el-dialog title="新增" v-model="addFormVisible" :close-on-click-modal="false">
			<el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
				<el-form-item label="标题" prop="name">
					<el-input v-model="addForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="副标题" prop="subName">
					<el-input v-model="addForm.subName" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="商品类型">
					<el-cascader
							:show-all-levels="false"
							:change-on-select="true"
							placeholder='请选择商品类型'
							:props="defaultParams"
							:options="options"
							v-model="addForm.productTypeId"
							:clearable="true">
					</el-cascader>
				</el-form-item>
				<el-form-item label="品牌">
					<el-select v-model="addForm.brandId" clearable placeholder="请选择品牌">
						<el-option
								v-for="item in brands"
								:label="item.name"
								:value="item.id">
						</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="媒体属性">
					<template>
						<el-upload
								class="upload-demo"
								action="http://172.16.4.5:9527/services/common/file"
								list-type="picture"
								:on-success="handleSuccess"
								:file-list="fileList"
						>
							<el-button size="small" type="primary">点击上传</el-button>
						</el-upload>
					</template>
				</el-form-item>
				<el-form-item label="商品描述">
					<el-input type="textarea" v-model="addForm.ext.description" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="商品详情">
					<quill-editor
							ref="QuillEditor"
							v-model="addForm.ext.richContent"
					></quill-editor>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="addFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
			</div>
		</el-dialog>
	</section>
</template>

<script>
	// import util from '../../common/js/util'
	// //import NProgress from 'nprogress'
	// import { getUserListPage, removeUser, batchRemoveUser, editUser, addUser } from '../../api/api';

	export default {
		data() {
			return {
				filters: {
					keyword: ''
				},
				products: [],
				options: [],
				fileList:[],//文件上传  用作回显
				brands:[],
				defaultParams: {
					label: 'name',
					value: 'id',
					children: 'children'
				},
				total: 0,
				page: 1,
				rows : 10,
				listLoading: false,
				sels: [],//列表选中列

				editFormVisible: false,//编辑界面是否显示
				editLoading: false,
				editFormRules: {
					name: [
						{ required: true, message: '请输入标题', trigger: 'blur' }
					],
					subName: [
						{ required: true, message: '请输入副标题', trigger: 'blur' }
					]
				},
				//编辑界面数据
				editForm: {
					name: '',
					subName: '',
					productTypeId: null,
					description:'',
					brandId: null,
					medias:'',
					productTypes:[],
					ext:{
						description:'',
						richContent:''
					}
				},

				addFormVisible: false,//新增界面是否显示
				addLoading: false,
				addFormRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//新增界面数据
				addForm: {
					name: '',
					subName: '',
					productTypeId: null,
					brandId: null,
					medias:'',
					productTypes:[],
					ext:{
						description:'',
						richContent:''
					}
				}

			}
		},
		methods: {
			//文件上传成功后的钩子函数
			handleSuccess(response, file, fileList){
				let {success,message,restObj} = response;
				if(success){
					this.addForm.logo = restObj;
					this.$message({
						message: '上传成功',
						type: 'success'
					});
					this.editForm.logo = restObj;
				}else{
					this.$message({
						message: message,
						type: 'error'
					});
				}
				this.fileList = fileList;
			},
			//品牌
			getBrands(){
				this.$http.get("/product/brand/list")
						.then(res=>{
							this.brands = res.data;
						})
			},
			//加载类型树
			loadTypeTree(){
				this.$http.get("/product/productType/loadTypeTree")
						.then(res=>{
							this.options = this.getTreeData(res.data);
						})
			},
			// 递归方法:去掉tree里面的空数组
			getTreeData(data) {
				// 循环遍历json数据
				for (var i = 0; i < data.length; i++) {
					if (data[i].children.length < 1) {
						// children若为空数组，则将children设为undefined
						data[i].children = undefined;
					} else {
						// children若不为空数组，则继续 递归调用 本方法
						this.getTreeData(data[i].children);
					}
				}
				return data;
			},
			//格式化上架时间
			formatOnSaleTime(row, column){
				return this.formatTime(row.onSaleTime);
			},
			//格式化下架时间
			formatOffSaleTime(row, column){
				return this.formatTime(row.offSaleTime);
			},
			//格式化时间方法
			formatTime(time) {
				if (!time){
					return null;
				}
				let date = new Date(time);
				let year = date.getFullYear();
				let month = date.getMonth()+1;
				let day = date.getDate();
				let hour = date.getHours();
				let minute = date.getMinutes();
				let second = date.getSeconds();
				let timeStr = year+"-"+this.formatTimeNum(month)+"-"+this.formatTimeNum(day)
						+" "+this.formatTimeNum(hour)+":"+this.formatTimeNum(minute)+":"+this.formatTimeNum(second);
				return timeStr;
			},
			formatTimeNum(num){
				if(num>=10){
					return num;
				}
				return "0"+ num;
			},
			//性别显示转换
			formatSex: function (row, column) {
				return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
			},
			handleCurrentChange(val) {
				this.page = val;
				this.getProducts();
			},
			//获取商品列表
			getProducts() {
				let para = {
					page: this.page,
					rows: this.rows,
					keyword: this.filters.keyword
				};
				this.listLoading = true;
				this.$http.post("/product/product/json",para)
						.then(res=>{
							this.listLoading = false;
							let {total,rows} = res.data;
							this.total = total;
							this.products = rows;
						})
			},
			//删除
			handleDel: function (index, row) {
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					this.$http.delete("/product/product/delete/"+ row.id)
							.then(res=>{
								this.listLoading = false;
								let {success,message,restObj} = res.data;
								if (success){
									this.$message({
										message: '删除成功',
										type: 'success'
									});
									this.getProducts();
								}else {
									this.$message({
										message: message,
										type: 'error'
									});
								}
							})
				}).catch(() => {
				});
			},

			//显示编辑界面
			handleEdit: function (index, row) {
				this.fileList = [];
				this.editFormVisible = true;
				this.editForm = Object.assign({}, row);
				this.fileList.push({"url":"http://172.16.4.164/" + row.medias})
				this.loadGetPath(index, row)
			},
			//显示新增界面
			handleAdd: function () {
				this.fileList = [];
				this.addFormVisible = true;
				this.addForm = {
					name: '',
					subName: '',
					productTypeId: null,
					brandId: null,
					medias:'',
					productTypes:[],
					ext:{
						description:'',
						richContent:''
					}
				};
			},
			//编辑
			editSubmit: function () {
				this.$refs.editForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.editLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.editForm);
							para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
							editUser(para).then((res) => {
								this.editLoading = false;
								//NProgress.done();
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['editForm'].resetFields();
								this.editFormVisible = false;
								this.getProducts();
							});
						});
					}
				});
			},

			/*编辑获取回填菜单*/
			loadGetPath(index, row){
				this.$http.get("/product/productType/"+row.productTypeId).then(res=>{
					var productOptions = res.data.path.split('.');
					let paths = [];
					let indexs ='';
					for (let index = 0 ; index < productOptions.length ; index++){
						if (productOptions[index] != ''){
							indexs = parseInt(productOptions[index]);
							paths.push(indexs);
						}
					}
					console.debug(paths);
					this.productOptions = paths;
				}).catch({});
			},

			//新增
			addSubmit: function () {
				this.$refs.addForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.addLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.addForm);
							for(var i = 0;i<this.addForm.productTypeId.length ; i++){
								var b = this.addForm.productTypeId[i];
							}
							//从fileList中获取文件路径用，拼接给medias赋值
							para.medias = this.fileList.map(file=>file.response.restObj).join(",");
							para.productTypeId = b;
							this.$http.post("/product/product/add",para).then(
									res=>{
										let {success,message,restObj} = res.data;
										if(success){
											this.addLoading = false;
											//NProgress.done();
											this.$message({
												message: '添加成功',
												type: 'success'
											});
											this.$refs['addForm'].resetFields();
											this.addFormVisible = false;
											this.getbrands();
										}else{
											this.$message({
												message: '添加失败',
												type: 'error'
											});
										}
									}
							).catch({

							});
						});
					}
				});
			},
			selsChange: function (sels) {
				this.sels = sels;
			},
			//批量删除
			batchRemove: function () {
				var ids = this.sels.map(item => item.id).toString();
				this.$confirm('确认删除选中记录吗？', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					this.$http.delete("/product/product/deleteBatch?ids="+ids)
							.then(res=>{
								this.listLoading = false;
								let {success,message,restObj} = res.data;
								if (success){
									this.$message({
										message: '删除成功',
										type: 'success'
									});
									this.getProducts();
								}else {
									this.$message({
										message: message,
										type: 'error'
									});
								}
							})
				}).catch(() => {
				});
			}
		},
		mounted() {
			this.getProducts();
			this.loadTypeTree();
			this.getBrands();
		}
	}

</script>

<style scoped>

</style>