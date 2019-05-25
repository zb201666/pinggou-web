<template>
    <section>
        <!-------------------------------------工具条----------------------------------------->
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
                    <el-button type="primary" @click="handleViewProperties" icon="el-icon-view" size="mini" round :disabled="this.sels.length!==1">显示属性</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="success" @click="handleSkuProperties" icon="el-icon-more" size="mini" round :disabled="this.sels.length!==1">SKU属性</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="warning" @click="handleOnSale(1)" icon="el-icon-upload2" size="mini" round :disabled="this.sels.length<=0">上架</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="danger" @click="handleOnSale(0)" icon="el-icon-download" size="mini" round :disabled="this.sels.length<=0">下架</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--------------------------------------------列表------------------------------------------->
        <el-table :data="products" border :header-cell-style="cellStyle" :cell-style="cellStyle" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
            <el-table-column type="selection" width="50">
            </el-table-column>
            <el-table-column type="index" width="50">
            </el-table-column>
            <el-table-column prop="name" label="标题" width="250">
            </el-table-column>
            <el-table-column prop="subName" label="副标题" width="130">
            </el-table-column>
            <el-table-column prop="productType.name" label="商品类型">
            </el-table-column>
            <el-table-column prop="brand.name" label="商品品牌">
            </el-table-column>
            <el-table-column prop="onSaleTime" label="上架时间" sortable width="160" :formatter="formatterOnSaleTime">
            </el-table-column>
            <el-table-column prop="offSaleTime" label="下架时间" sortable width="160" :formatter="formatterOffSaleTime">
            </el-table-column>
            <el-table-column prop="state" label="状态" sortable width="80">
                <template scope="scope">
                    <span v-if="scope.row.state==0" style="background-color: #ff1a39;padding: 2px 10px;border-radius: 3px;color: #fff">下架</span>
                    <span v-else style="background-color: #29ce57;padding: 2px 10px;border-radius: 3px;color: #fff">上架</span>
                </template>
            </el-table-column>
            <el-table-column label="操作" width="200">
                <template scope="scope">
                    <el-button size="mini" icon="el-icon-edit" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button type="danger" size="mini" icon="el-icon-circle-close" @click="handleDel(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>

        <!------------------------------------------工具条-------------------------------------------->
        <el-col :span="24" class="toolbar">
            <el-button type="danger" size="mini" icon='el-icon-error' @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
            <el-pagination
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="page"
                    :page-sizes="[5, 10, 15, 22]"
                    :page-size="size"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="total"
                    style="float:right;">
            </el-pagination>
        </el-col>

        <!----------------------------------------新增/编辑界面------------------------------------------>
        <el-dialog title="新增/编辑" :visible.sync="productVisible" width="50%" center :close-on-click-modal="false" @close="clearForm">
            <el-form :model="product" label-width="80px" :rules="productRules" ref="product">
                <el-form-item label="标题" prop="name">
                    <el-col :span="22">
                        <el-input v-model="product.name" auto-complete="off"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="副标题" prop="subName">
                    <el-col :span="22">
                        <el-input v-model="product.subName" auto-complete="off"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="品牌" prop="brandId">
                    <el-col :span="22">
                        <el-select style="width: 100%" v-model="product.brandId" clearable filterable placeholder="请选择品牌">
                            <el-option v-for="item in brands" :label="item.name" :value="item.id">
                            </el-option>
                        </el-select>
                    </el-col>
                </el-form-item>
                <el-form-item label="类型" prop="productTypeId">
                    <el-col :span="22">
                        <el-cascader style="width: 100%" v-model="product.productTypeId" placeholder="请选择类型" :props="defaultProps"
                                     :options="productTypes" clearable filterable change-on-select>
                        </el-cascader>
                    </el-col>
                </el-form-item>
                <el-form-item label="媒体属性">
                    <el-col :span="22">
                        <el-upload
                                class="upload-demo"
                                action="http://localhost:9527/services/common/file/upload"
                                :before-remove="handleMediasRemove"
                                :file-list="mediasList"
                                list-type="picture"
                                :on-success="handleUploadSeccess"
                                :on-remove="handleRemoveSeccess"
                                :accept="'image/*'">
                            <el-button size="small" type="primary">点击上传</el-button>
                        </el-upload>
                    </el-col>
                </el-form-item>
                <el-form-item label="商品描述">
                    <el-col :span="22">
                        <el-input type="textarea" v-model="product.description"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="商品详情">
                    <el-col :span="22">
                        <SquillEditorFastdfs
                                host="http://192.168.79.188"
                                v-model="product.content"
                                uploadUrl='http://localhost:9527/services/common/file/upload'
                                >
                        </SquillEditorFastdfs>
                    </el-col>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="cancelForm">取消</el-button>
                <el-button type="primary" @click.native="submitForm" :loading="submitLoading">提交</el-button>
            </div>
        </el-dialog>



        <!--------------------------------------------显示属性------------------------------------->
        <el-dialog title="显示属性管理" :visible.sync="viewDialogVisible" center width="40%" @close="cancelViewDialog">
            <!--卡片-->
            <el-card class="box-card" shadow="hover">
                <div v-for="index in viewProperties.length" :key="index" class="text item" style="margin-bottom: 5px">
                    <el-row>
                        <el-col :span="8" style="color: #20a0ff;padding-top: 10px;padding-right:20px;text-align: right">{{viewProperties[index-1].specName}}:</el-col>
                        <el-col :span="16">
                            <el-input v-model="viewProperties[index-1].value"></el-input>
                        </el-col>
                    </el-row>
                </div>
            </el-card>
            <span slot="footer" class="dialog-footer">
                <el-button @click.native="cancelViewDialog">取 消</el-button>
                <el-button type="primary" @click.native="submitViewProperties">确 定</el-button>
            </span>
        </el-dialog>

        <!-------------------------------------------sku属性------------------------------------------>
        <el-dialog title="sku属性管理" :visible.sync="skuDialogVisible" center width="60%" @close="cancelSkuDialog">
            <!--表示一个sku属性-->
            <el-card class="box-card" shadow="hover" v-for="property in skuProperties" >
                <div slot="header" class="clearfix">
                    <span style="color: #20a0ff">{{property.specName}}</span>
                </div>
                <!--表示一个属性选项-->
                <div v-for="index in property.options.length+1" :key="index" class="text item" style="margin-bottom: 5px">
                    <el-row>
                        <el-input v-model="property.options[index-1]" style="width:80%"></el-input>
                        <el-button icon="el-icon-remove" type="text" size="small" plain round @click="property.options.splice(index-1,1)" style="width:10%;margin-left: 10px">删除</el-button>
                    </el-row>
                </div>
            </el-card>
            <el-card class="box-card" shadow="hover" v-if="skus.length>0">
                <div slot="header" class="clearfix">
                    <span style="color: #20a0ff">库存和价格</span>
                </div>
                <el-table :data="skus" border stripe style="width: 100%" :header-cell-style="cellStyle" :cell-style="cellStyle">
                    <el-table-column type="index" width="50"></el-table-column>
                    <template v-for="(value,key) in skus[0]">
                        <!--更改价格和库存单元格和标题-->
                        <el-table-column v-if="key=='price'" :prop="key" label="价格">
                            <template scope="scope">
                                <el-input v-model="scope.row.price"></el-input>
                            </template>
                        </el-table-column>
                        <el-table-column v-else-if="key=='availableStock'" :prop="key" label="库存">
                            <template scope="scope">
                                <el-input v-model="scope.row.availableStock"></el-input>
                            </template>
                        </el-table-column>
                        <!--隐藏skuIndex属性-->
                        <el-table-column v-else-if="key!='skuIndex'" :prop="key" :label="key">
                        </el-table-column>
                    </template>
                </el-table>
            </el-card>
            <span slot="footer" class="dialog-footer">
                <el-button @click.native="cancelSkuDialog">取 消</el-button>
                <el-button type="primary" @click.native="submitSkuProperties">确 定</el-button>
            </span>
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
                    callback(new Error('请输入标题'));
                } else {
                    if(!this.product.id){//添加的时候发请求验证名称是否存在
                        this.$http.post("/product/product/page",{
                            keyword:value
                        }).then((res)=>{
                            let data = res.data;
                            if(data.rows.length>0){
                                callback(new Error('该标题已存在'));
                            }else{
                                callback();
                            }
                        })
                    }else{
                        callback();
                    }
                }
            };
            return {
                filters: {
                    keyword: ''
                },
                viewProperties:[],
                viewDialogVisible:false,
                skuProperties:[],
                skuDialogVisible:false,
                products: [],
                productTypes:[],
                brands:[],
                skus:[],
                mediasList: [],
                mediasFile:{
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
                        { required:true, message: "请选择类型", trigger: 'blur' }
                    ],
                    brandId: [
                        { required:true, message: "请选择品牌", trigger: 'blur' }
                    ]
                },
                //新增界面数据
                product: {
                    id:null,
                    name:null,
                    subName:null,
                    productTypeId:null,
                    brandId:null,
                    maxPrice:null,
                    minPrice:null,
                    description:null,
                    content:null,
                    mediasArr:[]
                },
                defaultProps: {
                    value: 'id',
                    label: 'name'
                }
            }
        },
        methods: {
            /*===================================单元格样式和分页处理==============================================*/
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

            /*=====================================显示属性===============================================*/
            handleViewProperties(){
                this.viewDialogVisible = true;
                let id = this.sels[0].id;
                this.$http.get("/product/product/viewProperties/"+id).then((res)=>{
                    this.viewProperties = res.data;
                })
            },
            cancelViewDialog(){
                this.viewProperties = [];
                this.viewDialogVisible = false;
            },
            submitViewProperties(){
                let params = {};
                params.id = this.sels[0].id;
                params.viewProperties = JSON.stringify(this.viewProperties);
                this.$confirm('确认提交吗？', '提示', {}).then(() => {
                    this.$http.post("/product/product/viewProperties",params).then((res)=>{
                        let data = res.data;
                        if(data.success){
                            this.$message({
                                message:data.message,
                                type:"success"
                            });
                            this.viewDialogVisible = false;
                        }else{
                            this.$message({
                                message:data.message,
                                type:"error"
                            })
                        }
                    })
                })
            },


            /*=========================================sku属性============================================*/
            cancelSkuDialog(){
                this.skuDialogVisible = false;
            },
            submitSkuProperties(){
                let params = {};
                params.productId = this.sels[0].id;
                params.skuProperties = JSON.stringify(this.skuProperties);
                params.skus = this.skus;
                this.$confirm('确认提交吗？', '提示', {}).then(() => {
                    this.$http.post("/product/product/skuProperties",params).then((res)=>{
                        let data = res.data;
                        if(data.success){
                            this.$message({
                                message:data.message,
                                type:"success"
                            });
                            this.skuDialogVisible = false;
                        }else{
                            this.$message({
                                message:data.message,
                                type:"error"
                            })
                        }
                    })
                })
            },
            handleSkuProperties(){
                this.skuDialogVisible = true;
                let id = this.sels[0].id;
                this.skuProperties = [];
                this.skus = [];
                this.$http.get("/product/product/skuProperties/"+id).then((res)=>{
                    this.skuProperties = res.data;
                });
                this.$http.get("/product/product/skus/"+id).then((res)=>{
                    let data = res.data;
                    for(let i in data){
                        for(let j in this.skus){
                            if(data[i].skuIndex==this.skus[j].skuIndex){
                                this.skus[j].price = data[i].price+"";//注意将整型转为字符串
                                this.skus[j].availableStock = data[i].availableStock+"";
                                break;
                            }
                        }
                    }
                });
            },

            /*======================================商品上下架========================================================*/
            handleOnSale(state){
                let flag = false;
                this.sels.forEach(s=>{
                    if(s.state==state){
                        flag = true;
                    }
                });
                if(flag){
                    this.$message({
                        message: "请不要重复"+(state==1?"上":"下")+"架",
                        type: 'warning'
                    });
                    return;
                }
                this.$confirm("确认"+(state==1?"上":"下")+"架吗？", '提示', {}).then(() => {
                    let ids = this.sels.map(item=>item.id).join();
                    let url = state==1?"/product/product/onSale":"/product/product/offSale";
                    this.$http.get(url, {
                        params:{
                            ids:ids
                        }
                    }).then((result) => {
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
                        });
                });
            },







            /*==============================商品CRUD=============================================================*/

            //文件上传成功钩子函数
            handleUploadSeccess(file,response,fileList){
                this.mediasList = fileList;
            },
            //文件删除成功钩子函数
            handleRemoveSeccess(file, fileList){
                this.mediasList = fileList;
            },

            formatterOnSaleTime(row, column) {
                return row.onSaleTime ? this.formatDate(row.onSaleTime) : ""
            },
            formatterOffSaleTime(row, column) {
                return row.offSaleTime ? this.formatDate(row.offSaleTime) : ""
            },

            //时间显示转换
            formatDate(longTime) {
                let date = new Date(longTime);
                let year = date.getFullYear();//2019
                let month = date.getMonth() + 1;//月份从0开始
                let day = date.getDate();//日
                let hour = date.getHours();
                let minute = date.getMinutes();
                let second = date.getSeconds();
                return year + "-" + this.numberFormatter(month) + "-" + this.numberFormatter(day) + " "
                    + this.numberFormatter(hour) + ":" + this.numberFormatter(minute) + ":" + this.numberFormatter(second);
            },
            numberFormatter(num) {
                if (num < 10) {
                    return "0" + num;
                }
                return num;
            },


            //取得需要保存的媒体数据
            getMediasArr(){
                if(this.mediasList.length) {
                    for (let index in this.mediasList) {
                        if (this.mediasList[index].response && this.mediasList[index].response.data) {
                            this.product.mediasArr.push(this.mediasList[index].response.data);
                        } else if (this.mediasList[index].fileId) {
                            this.product.mediasArr.push(this.mediasList[index].fileId);
                        }
                    }
                }
                return this.product.mediasArr;
            },
            //删除文件
            handleMediasRemove(fileList){
                let fileIds = [];
                if(fileList.length){
                        for(let index in fileList){
                            if(fileList[index].response && fileList[index].response.data){
                                fileIds.push(fileList[index].response.data);
                            }else if(fileList[index].fileId){
                                fileIds.push(fileList[index].fileId);
                            }
                        }
                    this.$http.delete("/common/file/batchDelete",{
                        params:{
                            fileIds:fileIds.join()
                        }
                    }).then(res=>{
                        let data = res.data;
                        if(data.success){
                            this.$message({
                                message: data.message,
                                type: 'success'
                            });
                            //如果是修改，删除文件后需要将数据中的媒体信息设置为空
                            if(this.product.id){
                                this.updateMedias(this.product.id);
                            }
                        }else{
                            this.$message({
                                message: data.message,
                                type: 'error'
                            });
                            //返回false表示无法删除
                            return false;
                        }
                    })
                }
            },

            //将数据库中的媒体信息设置为空
            updateMedias(productId){
                this.$http.post("/product/product/updateMedias",{
                    id:productId,
                    medias:null
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
            },

            //批量删除媒体图片
            removeBatchMedias(medias){
                this.$http.delete("/common/file/batchDelete",{
                    params:{
                        fileIds:medias.join()
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
                    name:null,
                    subName:null,
                    productTypeId:[],
                    brandId:null,
                    maxPrice:null,
                    minPrice:null,
                    description:null,
                    content:null,
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
                    this.handleMediasRemove(this.mediasList);//清空媒体属性
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
                if(row.medias){
                    let arr = JSON.parse(row.medias);
                    //图片回填
                    for(let index in arr){
                        let mediasFile = {
                            url :"http://192.168.79.188"+arr[index],
                            fileId : arr[index]
                        };
                        this.mediasList.push(mediasFile);
                    }
                }
                this.product.mediasArr = [];
                this.$http.get("/product/productExt/loadProductExtByProductId?productId="+row.id).then((res)=>{
                    let data = res.data;
                    this.product.description = data.description;
                    this.product.content = data.richContent;
                })
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
                            para.medias = this.arrToString(this.getMediasArr());
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

                            if(row.medias){
                                let medias = JSON.parse(row.medias);
                                this.removeBatchMedias(medias);
                            }
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
                let ids = this.sels.map(item => item.id);
                let medias = this.sels.map(item => item.medias);
                //将medias中的值全部放在一个数组中
                let fileIds = medias.reduce((pre,cur)=>{
                    if(cur){
                        JSON.parse(cur).forEach(e=>{
                            pre.push(e);
                        });
                    }
                    return pre;
                },[]);
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

                            //删除媒体文件
                            if(fileIds.length){
                                this.removeBatchMedias(fileIds);
                            }
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
        },
        watch:{
            skuProperties:{//深度监听，可监听到对象、数组的变化
                handler(val, oldVal){
                    if(this.skuProperties.length){
                        //动态生成skus值
                        let res = this.skuProperties.reduce((pre,cur)=>{
                            let result = [];
                            pre.forEach(o1=>{
                                o1.skuIndex = (o1.skuIndex||'')+"_";//skuIndex存在就拼接原值+"_"，没有就""+"_"
                                for(let i=0;i<cur.options.length;i++){
                                    let o2 = cur.options[i];
                                    let temp = {};
                                    Object.assign(temp,o1);//将ol复制给temp
                                    temp[cur.specName] = o2;//将o2赋值给temp中的cur.specName
                                    temp.skuIndex += i;//sku索引拼接当前选项值的索引，最终拼接成形如_X_X_X的结果
                                    result.push(temp);//将循环结果加入最终结果中
                                }
                            });
                            return result;
                        },[{}]);

                        //添加价格和库存
                        res.forEach(o1=>{
                            o1.price = "0";
                            o1.availableStock = "0";
                            o1.skuIndex = o1.skuIndex.substring(1);
                        });
                        this.skus = res;
                    }
                },
                deep:true
            }
        }
    }

</script>

<style scoped>

</style>