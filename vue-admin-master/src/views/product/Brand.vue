<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input v-model="filters.keyword" placeholder="关键字" size="mini"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getBrands" icon="el-icon-search" size="mini" round>查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="success" v-on:click="clearQuery" icon="el-icon-zoom-out" size="mini" round>清空查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="warning" @click="handleAdd" icon="el-icon-circle-plus-outline" size="mini" round>新增</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="brands" border :header-cell-style="cellStyle" :cell-style="cellStyle" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
            <el-table-column type="selection" width="50">
            </el-table-column>
            <el-table-column type="index" width="50">
            </el-table-column>
            <el-table-column prop="name" label="名称" sortable>
            </el-table-column>
            <el-table-column prop="englishName" label="英文名称" sortable>
            </el-table-column>
            <el-table-column prop="firstLetter" label="首字母" sortable>
            </el-table-column>
            <el-table-column prop="sortIndex" label="序号" sortable>
            </el-table-column>
            <el-table-column prop="logo" label="品牌logo"  width="310px" sortable>
            </el-table-column>
            <el-table-column prop="productType.name" label="商品类型" sortable>
            </el-table-column>
            <el-table-column prop="description" label="描述" sortable>
            </el-table-column>
            <el-table-column label="操作" width="180">
                <template scope="scope">
                    <el-button size="mini" icon="el-icon-edit" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button type="danger" size="mini" icon="el-icon-circle-close" @click="handleDel(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>

        <!--工具条-->
        <el-col :span="24" class="toolbar">
            <el-button type="danger" size="mini" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
            <el-pagination
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="page"
                    :page-sizes="[5, 10, 15, 20]"
                    :page-size="size"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="total"
                    style="float:right;">
            </el-pagination>
        </el-col>

        <!--新增/编辑界面-->
        <el-dialog title="新增/编辑" :visible.sync="brandVisible" width="40%" center :close-on-click-modal="false" @close="clearForm">
            <el-form :model="brand" label-width="80px" :rules="brandRules" ref="brand">
                <el-form-item label="名称" prop="name">
                    <el-col :span="20">
                        <el-input v-model="brand.name" auto-complete="off"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="英文名称" prop="englishName">
                    <el-col :span="20">
                        <el-input v-model="brand.englishName" auto-complete="off"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="首字母">
                    <el-col :span="20">
                        <el-input v-model="brand.firstLetter" :value="getFirstLetter()" :disabled="true"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="描述">
                    <el-col :span="20">
                        <el-input type="textarea" v-model="brand.description"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="排序索引">
                    <el-col :span="20">
                        <el-input v-model="brand.sortIndex"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="类型">
                    <el-col :span="20">
                        <!--<el-select v-model="brand.productTypeId" clearable filterable placeholder="请选择类型">
                            <el-option v-for="item in productTypes" :label="item.name" :value="item.id">
                            </el-option>
                        </el-select>-->
                        <el-cascader v-model="chosedProductTypes" placeholder="请选择类型" :props="defaultProps"
                                     :options="productTypes" clearable filterable change-on-select>
                        </el-cascader>
                    </el-col>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="cancelForm">取消</el-button>
                <el-button type="primary" @click.native="submitForm" :loading="submitLoading">提交</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>

    export default {
        data() {
            var validateKeyword = (rule, value, callback) => {
                if (!value) {
                    callback(new Error('请输入名称'));
                } else {
                    this.$http.post("/product/brand/page",{
                        keyword:value
                    }).then((res)=>{
                        let data = res.data;
                        if(data.rows.length>0){
                            callback(new Error('该名称已存在'));
                        }else{
                            callback();
                        }
                    })
                }
            };
            return {
                filters: {
                    keyword: ''
                },
                brands: [],
                productTypes:[],
                chosedProductTypes:[],
                total: 0,
                page: 1,
                size:10,
                listLoading: false,
                sels: [],//列表选中列
                brandVisible: false,//新增界面是否显示
                submitLoading: false,
                brandRules: {
                    name: [
                        { required:true, validator: validateKeyword, trigger: 'blur' }
                    ],
                    englishName: [
                        { required:true, validator: validateKeyword, trigger: 'blur' }
                    ]
                },
                //新增界面数据
                brand: {
                    id:null,
                    name: null,
                    englishName: null,
                    firstLetter: null,
                    description: null,
                    sortIndex: null,
                    productTypeId:null
                },
                defaultProps: {
                    value: 'id',
                    label: 'name'
                }
            }
        },
        methods: {
            //列表单元格样式
            cellStyle({row,column,rowIndex,columnIndex}) {
                return "text-align:center;color:#303133;font-size:14px;padding:7px 0px";
            },
            //当前页大小改变
            handleSizeChange(val) {
                this.size = val;
                this.getBrands();
            },
            //当前页码改变
            handleCurrentChange(val) {
                this.page = val;
                this.getBrands();
            },
            getFirstLetter(){
                this.brand.firstLetter = (null==this.brand.englishName)?null:this.brand.englishName.substr(0,1);
            },
            //获取商品品牌列表
            getBrands() {
                this.listLoading = true;
                this.$http.post("/product/brand/page",{
                    page:this.page,
                    size:this.size,
                    keyword:this.filters.keyword
                }).then((res)=>{
                    this.listLoading = false;
                    this.brands = res.data.rows;
                    this.total = res.data.total;
                })
            },
            //回到第一页展示数据
            queryBrands() {
                this.page = 1;
                this.getBrands();
            },
            //获取商品类型
            getProductType() {
                this.$http.get("/product/productType/tree").then((res)=>{
                    this.productTypes = this.clearChildren(res.data);
                })
            },
            //递归清除children为[]的情况
            clearChildren(arr){
                for(let i = 0;i<arr.length;i++){
                    if(arr[i].children.length==0){
                        arr[i].children = null;
                    }else{
                        this.clearChildren(arr[i].children);
                    }
                }
                return arr;
            },
            //清空查询则查所有
            clearQuery() {
                this.filters.keyword = null;
                this.queryBrands();
            },
            clearBrand(){
                this.brand = {
                    id:null,
                    name: null,
                    englishName: null,
                    firstLetter: null,
                    description: null,
                    sortIndex: null,
                    productTypeId:null
                };
            },
            //清空表单
            clearForm(){
                this.clearBrand();
                this.chosedProductTypes=[];
                //对话框关闭清空表单验证
                this.$nextTick(() => {
                    this.$refs.brand.clearValidate();
                });
            },
            //取消表单
            cancelForm() {
                this.clearForm();
                this.brandVisible = false;
            },
            //显示编辑界面
            handleEdit: function (index, row) {
                this.brandVisible = true;
                this.$http.get("/product/brand/"+row.id).then((res)=>{
                    this.brand = res.data;
                    //递归方法回显
                    this.chosedProductTypes = this.cascaderSelect(this.brand.productTypeId,this.productTypes);
                })
            },
            //显示新增界面
            handleAdd: function () {
                this.brandVisible = true;
                this.clearBrand();
            },
            //级联选择器回显
            cascaderSelect(key,treeData){
                let actionArr = []; // 在递归时操作的数组
                let returnArr = []; // 存放结果的数组
                let depth = 0; // 定义全局层级
                // 定义递归函数
                function childrenEach(arr, depthN) {
                    for (var i = 0; i < arr.length; i++) {
                        depth = depthN; // 将执行的层级赋值 到 全局层级
                        actionArr[depthN] = (arr[i].id);
                        if (arr[i].id == key) {
                            returnArr = actionArr.slice(0, depthN+1); //将目前匹配的数组，截断并保存到结果数组，并终止递归
                            break;
                        } else {
                            //未匹配上就继续递归调用
                            if (arr[i].children) {
                                depth ++;
                                childrenEach(arr[i].children, depth);
                            }
                        }
                    }
                    return returnArr;
                }
                return childrenEach(treeData, depth);
            },
            //新增/编辑
            submitForm: function () {
                this.$refs.brand.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.submitLoading = true;
                            this.brand.productTypeId = this.chosedProductTypes[this.chosedProductTypes.length-1];
                            this.$http.post("/product/brand", this.brand)
                                .then((result) => {
                                    //成功后的回调
                                    let data = result.data;
                                    if (data.success) {
                                        //成功
                                        this.$message({
                                            message: data.message,
                                            type: 'success'
                                        });
                                        this.queryBrands();
                                    } else {
                                        //失败
                                        this.$message.error(data.message);
                                    }
                                    this.brandVisible = false;
                                    this.submitLoading = false;
                                });
                        });
                    }
                });
            },
            //选中项
            selsChange: function (sels) {
                this.sels = sels;
            },
            //删除
            handleDel: function (index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.$http.delete("/product/brand/"+row.id).then((res)=>{
                        let data = res.data;
                        if(data.success){
                            this.$message({
                                showClose: true,
                                type: 'success',
                                message: data.message
                            });
                            this.queryBrands();
                        }
                    })
                })
            },
            //批量删除
            batchRemove: function () {
                var ids = this.sels.map(item => item.id);
                this.$confirm('确认删除选中记录吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.$http.delete("/product/brand/batch/"+ids).then((res)=>{
                        let data = res.data;
                        if(data.success){
                            this.$message({
                                showClose: true,
                                type: 'success',
                                message: data.message
                            });
                            this.queryBrands();
                        }
                    })
                })
            }
        },
        mounted() {
            this.getBrands();
            this.getProductType();
        }
    }

</script>

<style scoped>

</style>