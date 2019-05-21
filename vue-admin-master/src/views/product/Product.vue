<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input v-model="filters.keyword" placeholder="关键字" size="mini"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getProducts" icon="el-icon-search" size="mini" round>查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="success" v-on:click="clearQuery" icon="el-icon-zoom-out" size="mini" round>清空查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="warning" @click="handleAdd" icon="el-icon-circle-plus-outline" size="mini" round>新增</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd" icon="el-icon-view" size="mini" round>显示属性</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="success" @click="handleAdd" icon="el-icon-more" size="mini" round>SKU属性</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="warning" @click="handleAdd" icon="el-icon-upload2" size="mini" round>上架</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="danger" @click="handleAdd" icon="el-icon-download" size="mini" round>下架</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="products" border :header-cell-style="cellStyle" :cell-style="cellStyle" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
            <el-table-column type="selection" width="50">
            </el-table-column>
            <el-table-column type="index" width="50">
            </el-table-column>
            <el-table-column prop="name" label="标题" width="250">
            </el-table-column>
            <el-table-column prop="subName" label="副标题" width="150">
            </el-table-column>
            <el-table-column prop="productType.name" label="商品类型">
            </el-table-column>
            <el-table-column prop="brand.name" label="商品品牌">
            </el-table-column>
            <el-table-column prop="onSaleTime" label="上架时间" sortable width="160">
            </el-table-column>
            <el-table-column prop="offSaleTime" label="下架时间" sortable width="160">
            </el-table-column>
            <el-table-column prop="state" label="状态" sortable width="80">
                <template scope="scope">
                    <span v-if="scope.row.state==0" style="background-color: #ff1a39;padding: 2px 10px;border-radius: 3px;color: #fff">下架</span>
                    <span v-else style="background-color: #29ce57;padding: 2px 10px;border-radius: 3px;color: #fff">上架</span>
                </template>
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
            <el-button type="danger" size="mini" icon='el-icon-error' @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
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
        <el-dialog title="新增/编辑" :visible.sync="productVisible" width="40%" center :close-on-click-modal="false" @close="clearForm">
            <el-form :model="product" label-width="80px" :rules="productRules" ref="product">
                <el-form-item label="标题" prop="name">
                    <el-col :span="20">
                        <el-input v-model="product.name" auto-complete="off"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="副标题" prop="subName">
                    <el-col :span="20">
                        <el-input v-model="product.subName" auto-complete="off"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="品牌" prop="brandId">
                    <el-col :span="20">
                        <el-select style="width: 100%" v-model="product.brandId" clearable filterable placeholder="请选择品牌">
                            <el-option v-for="item in brands" :label="item.name" :value="item.id">
                            </el-option>
                        </el-select>
                    </el-col>
                </el-form-item>
                <el-form-item label="类型" prop="productTypeId">
                    <el-col :span="20">
                        <el-cascader style="width: 100%" v-model="product.productTypeId" placeholder="请选择类型" :props="defaultProps"
                                     :options="productTypes" clearable filterable change-on-select>
                        </el-cascader>
                    </el-col>
                </el-form-item>
                <el-form-item label="媒体属性">
                    <el-col :span="20">
                        <el-upload
                                class="upload-demo"
                                action="http://localhost:9527/services/common/file/upload"
                                :before-remove="handleLogoRemove"
                                :file-list="mediasList"
                                list-type="picture"
                                :on-success="handleUploadSeccess"
                                :on-exceed="handleOutOfRange"
                                :auto-upload="true"
                                :on-change="handleChange">
                            <el-button size="small" type="primary">点击上传</el-button>
                        </el-upload>
                    </el-col>
                </el-form-item>
                <el-form-item label="商品描述">
                    <el-col :span="20">
                        <el-input type="textarea" v-model="product.description"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="商品详情">
                    <el-col :span="20">
                        <SquillEditorFastdfs host="http://192.168.43.127" v-model="product.content" uploadUrl='http://localhost:9527/services/common/file/upload'></SquillEditorFastdfs>
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
    import SquillEditorFastdfs from '@/components/SquillEditorFastdfs.vue';
    export default {
        components:{
            SquillEditorFastdfs //富文本框上传组件
        },
        data() {
            var validateKeyword = (rule, value, callback) => {
                if (!value) {
                    callback(new Error('请输入名称'));
                } else {
                    if(!this.product.id){//添加的时候发请求验证名称是否存在
                        this.$http.post("/product/product/page",{
                            keyword:value
                        }).then((res)=>{
                            let data = res.data;
                            if(data.rows.length>0){
                                callback(new Error('该名称已存在'));
                            }else{
                                callback();
                            }
                        })
                    }else{
                        callback();
                    }
                }
            };
            var validateNumber = (rule, value, callback) => {
                if (!value) {
                    callback(new Error('请输入排序索引'));
                } else {
                    if(!Number.isInteger(parseInt(value))) {
                        callback(new Error('请输入数字值'));
                    }else{
                        if(value < 0) {
                            callback(new Error('不能小于0'));
                        }else {
                            callback();
                        }
                    }
                }
            };
            return {
                filters: {
                    keyword: ''
                },
                products: [],
                productTypes:[],
                brands:[],
                mediasList: [],
                logoFile:{
                    url:null,
                    fileId:null
                },
                total: 0,
                page: 1,
                size:10,
                listLoading: false,
                sels: [],//列表选中列
                productVisible: false,//新增界面是否显示
                submitLoading: false,
                productRules: {
                    name: [
                        { required:true, validator: validateKeyword, trigger: 'blur' }
                    ],
                    subName: [
                        { required:true, validator: validateKeyword, trigger: 'blur' }
                    ],
                    productTypeId: [
                        { required:true, message:'请选择类型', trigger: 'blur' }
                    ]
                },
                //新增界面数据
                product: {
                    id:null,
                    name:'',
                    subName:'',
                    productTypeId:null,
                    brandId:null,
                    maxPrice:null,
                    minPrice:null,
                    description:'',
                    content:'',
                    mediasArr:[]
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
                this.getProducts();
            },
            //当前页码改变
            handleCurrentChange(val) {
                this.page = val;
                this.getProducts();
            },
            //文件上传成功钩子函数
            handleUploadSeccess(response){
                this.product.mediasArr.push(response.data);
            },
            //文件超出个数
            handleOutOfRange(){
                this.$message({
                    message: '只能上传一张图片',
                    type: 'warning'
                });
            },
            //选择文件时回调
            handleChange(file, fileList){
                this.mediasList = fileList;
            },
            //删除文件
            handleLogoRemove(file,fileList){
                let fileId = null;
                if(file.response){
                    fileId = file.response.data;
                }else if(file.fileId){
                    fileId = file.fileId;
                }
                this.$http.delete("/common/file/delete",{
                    params:{
                        fileId:fileId
                    }
                }).then(res=>{
                    let data = res.data;
                    if(data.success){
                        this.$message({
                            message: data.message,
                            type: 'success'
                        });
                        //如果是修改，删除文件后需要将数据中的logo信息设置为空
                        if(this.product.id){
                            this.$http.post("/product/product/updateLogo",{
                                id:this.product.id,
                                logo:null
                            }).then((res)=>{
                                if(res.data.success){
                                    this.$message({
                                        message: res.data.message,
                                        type: 'success'
                                    });
                                }else{
                                    this.$message({
                                        message: res.data.message,
                                        type: 'error'
                                    });
                                }
                            })
                        };
                    }else{
                        this.$message({
                            message: data.message,
                            type: 'error'
                        });
                        //返回false表示无法删除
                        return false;
                    }
                })
            },
            //获取商品列表
            getProducts() {
                this.listLoading = true;
                this.$http.post("/product/product/page",{
                    page:this.page,
                    size:this.size,
                    keyword:this.filters.keyword
                }).then((res)=>{
                    this.listLoading = false;
                    this.products = res.data.rows;
                    this.total = res.data.total;
                })
            },
            //回到第一页展示数据
            queryProducts() {
                this.page = 1;
                this.getProducts();
            },
            //获取商品类型
            getProductType() {
                this.$http.get("/product/productType/tree").then((res)=>{
                    this.productTypes = this.clearChildren(res.data);
                })
            },
            //取得商品品牌
            getBrands(){
                this.$http.get("/product/brand/list").then((res)=>{
                    this.brands = res.data;
                })
            },
            //递归清除children为[]的情况
            clearChildren(arr){
                for(let i = 0;i<arr.length;i++){
                    if(!arr[i].children.length){
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
                this.queryProducts();
            },
            clearProduct(){
                this.product = {
                    id:null,
                    name:'',
                    subName:'',
                    productTypeId:null,
                    brandId:null,
                    maxPrice:null,
                    minPrice:null,
                    description:'',
                    content:'',
                    mediasArr:[]
                };
            },
            //清空表单
            clearForm(){
                this.clearProduct();
                if(!this.product.id){//添加才删除
                    this.mediasList=[];//清空文件列表
                }
                //对话框关闭清空表单验证
                this.$nextTick(() => {
                    this.$refs.product.clearValidate();
                });
            },
            //取消表单
            cancelForm() {
                if(!this.product.id && this.mediasList.length){//添加才删除
                    this.handleLogoRemove(this.mediasList[0]);//清空logo
                }
                this.clearForm();
                this.productVisible = false;
                this.queryProducts();
            },
            //显示编辑界面
            handleEdit: function (index, row) {
                this.productVisible = true;
                this.product = Object.assign({}, row);
                this.product.productTypeId = this.cascaderSelect(this.product.productTypeId,this.productTypes);
                if(row.logo){
                    //图片回填
                    this.logoFile.url = "http://192.168.43.127"+row.logo;
                    this.logoFile.fileId = row.logo;
                    this.mediasList.push(this.logoFile);
                }
            },
            //显示新增界面
            handleAdd: function () {
                this.productVisible = true;
                this.clearProduct();
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
            //手动拼接需要保存的媒体文件信息
            arrToString(arr){
                let result = '[';
                for(let index=0;index<arr.length;index++){
                    result += "\""+arr[index]+"\""
                    if(index!=arr.length-1){
                        result+=","
                    }
                }
                result += ']';
                return result;
            },
            //新增/编辑
            submitForm: function () {
                this.$refs.product.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.submitLoading = true;
                            let para = Object.assign({}, this.product);
                            para.medias = this.arrToString(this.product.mediasArr);
                            para.productTypeId = para.productTypeId[para.productTypeId.length - 1];
                            this.$http.post("/product/product", para)
                                .then((result) => {
                                    //成功后的回调
                                    let data = result.data;
                                    if (data.success) {
                                        //成功
                                        this.$message({
                                            message: data.message,
                                            type: 'success'
                                        });
                                        this.queryProducts();
                                    } else {
                                        //失败
                                        this.$message.error(data.message);
                                    }
                                    this.productVisible = false;
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
                    //删除对象
                    this.$http.delete("/product/product/"+row.id).then((res)=>{
                        let data = res.data;
                        if(data.success){
                            this.$message({
                                type: 'success',
                                message: data.message
                            });
                            //删除logo
                            this.$http.delete("/common/file/delete",{
                                params:{
                                    fileId:row.logo
                                }
                            }).then(res=>{
                                let data = res.data;
                                if(data.success){
                                    this.$message({
                                        message: data.message,
                                        type: 'success'
                                    });
                                }else{
                                    this.$message({
                                        message: data.message,
                                        type: 'error'
                                    });
                                }
                            })
                        }else{
                            this.$message({
                                message: data.message,
                                type: 'error'
                            });
                        }
                        this.queryProducts();
                    });
                })
            },
            //批量删除
            batchRemove: function () {
                var ids = this.sels.map(item => item.id);
                var fileIds = this.sels.map(item => item.logo);
                this.$confirm('确认删除选中记录吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.$http.delete("/product/product/batch/"+ids).then((res)=>{
                        let data = res.data;
                        if(data.success){
                            this.$message({
                                type: 'success',
                                message: data.message
                            });
                            this.$http.delete("/common/file/batchDelete",{
                                params:{
                                    fileIds:fileIds.join(",")
                                }
                            }).then(()=>{
                                if(res.data.success){
                                    this.$message({
                                        type: 'success',
                                        message: res.data.message
                                    });
                                }else{
                                    this.$message({
                                        message: res.data.message,
                                        type: 'error'
                                    });
                                }
                            });
                        }else{
                            this.$message({
                                message: data.message,
                                type: 'error'
                            });
                        }
                        this.queryProducts();
                    })
                })
            }
        },
        mounted() {
            this.getProducts();
            this.getProductType();
            this.getBrands();
        }
    }

</script>

<style scoped>

</style>