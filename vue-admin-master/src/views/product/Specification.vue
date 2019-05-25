<template>
    <el-container style="height: 600px">
        <el-aside width="300px">
            <el-tree :data="showProductTypes" :props="defaultProps"
                     @node-click="handleNodeClick"
                     :highlight-current="true"
                     node-key="id" :default-expanded-keys="[0]"
                     :expand-on-click-node = "false">
                <span class="custom-tree-node" slot-scope="{ node, data }">
                    <span>
                        <i :class="data.icon"></i>{{ data.name }}
                    </span>
                </span>
            </el-tree>
        </el-aside>
        <el-main>
            <section v-if="specifications.length>0">
                <!--------------------------------工具条----------------------------------------------------->
                <el-col :span="24" style="padding: 0px;margin: 0px">
                    <el-form :inline="true">
                        <el-form-item>
                            <el-button type="primary" v-on:click="" icon="el-icon-search" size="mini" round>查询</el-button>
                        </el-form-item>
                        <el-form-item>
                            <el-button type="success" v-on:click="" icon="el-icon-zoom-out" size="mini" round>清空查询</el-button>
                        </el-form-item>
                        <el-form-item>
                            <el-button type="warning" @click="handleAdd" icon="el-icon-circle-plus-outline" size="mini" round>新增</el-button>
                        </el-form-item>
                    </el-form>
                </el-col>
                <!---------------------------------列表----------------------------------->
                <el-table :data="specifications" border :header-cell-style="cellStyle" :cell-style="cellStyle"
                          highlight-current-row v-loading="listLoading" style="width: 100%;"
                          :row-class-name="tableRowClassName" height="555"  @selection-change="selsChange">
                    <el-table-column type="selection" width="50">
                    </el-table-column>
                    <el-table-column type="index" width="50">
                    </el-table-column>
                    <el-table-column prop="specName" label="属性名称">
                    </el-table-column>
                    <el-table-column prop="productType.name" label="商品类型">
                    </el-table-column>
                    <el-table-column prop="isSku" label="是否是sku属性" width="120">
                        <template scope="scope">
                            <span v-if="scope.row.isSku==0" style="background-color: #ff1a39;padding: 2px 10px;border-radius: 3px;color: #fff">否</span>
                            <span v-else style="background-color: #29ce57;padding: 2px 10px;border-radius: 3px;color: #fff">是</span>
                        </template>
                    </el-table-column>
                    <el-table-column label="操作" width="200">
                        <template scope="scope">
                            <el-button size="mini" icon="el-icon-edit" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                            <el-button type="danger" size="mini" icon="el-icon-circle-close" @click="handleDel(scope.$index, scope.row)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </section>
        </el-main>

        <!-------------------------------新增/编辑--------------------------------->
        <!--<el-dialog title="新增/编辑" :visible.sync="specificationVisible" width="50%" center :close-on-click-modal="false" :before-close="clearForm">
            <el-form :model="specification" label-width="100px" :rules="specificationRules" ref="specification">
                <el-form-item label="名称" prop="name">
                    <el-col :span="22">
                        <el-input v-model="productType.name" auto-complete="off" clearable></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="父级类型">
                    <el-col :span="22">
                        &lt;!&ndash;<el-select style="width: 100%" v-model="productType.pid" clearable filterable placeholder="请选择父级类型">
                            <el-option v-for="item in chooseProductTypes" :label="item.name" :value="item.id">
                            </el-option>
                        </el-select>&ndash;&gt;
                        <el-input v-model="productType.pname" auto-complete="off" :disabled="true"></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="排序索引" prop="sortIndex">
                    <el-col :span="22">
                        <el-input v-model="productType.sortIndex" auto-complete="off" clearable></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="商品数量" prop="totalCount">
                    <el-col :span="22">
                        <el-input v-model="productType.totalCount" auto-complete="off" clearable></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="分类标题" prop="seoTitle">
                    <el-col :span="22">
                        <el-input v-model="productType.seoTitle" auto-complete="off" clearable></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="分类关键字" prop="seoKeywords">
                    <el-col :span="22">
                        <el-input v-model="productType.seoKeywords" auto-complete="off" clearable></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="商品描述">
                    <el-col :span="22">
                        <el-input type="textarea" v-model="productType.description" clearable></el-input>
                    </el-col>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="cancelForm">取消</el-button>
                <el-button type="primary" @click.native="submitForm" :loading="submitLoading">提交</el-button>
            </div>
        </el-dialog>-->

    </el-container>
</template>

<script>
    export default {
        data() {
            return {
                sels:[],
                listLoading: false,
                specificationVisible:false,
                submitLoading:false,
                showProductTypes: [{
                    id:0,
                    name:"商品类型",
                    icon:"el-icon-menu",
                    children:null
                }],
                chooseProductTypes:[],
                defaultProps: {
                    children: 'children',
                    label: 'name',
                },
                specifications: [],
                deleteObject:null,
                specification:{
                    id:null,
                    specName:null,
                    productTypeId:null,
                    isSku:null
                },
                specificationRules:{
                    specName: [
                        { required:true, message: "请输入属性名称", trigger: 'blur' }
                    ]
                }
            }
        },
        methods: {
            //列表单元格样式
            cellStyle({row, column, rowIndex, columnIndex}) {
                return "text-align:center;color:#303133;font-size:14px;padding:7px 0px";
            },
            tableRowClassName({row, rowIndex}) {
                if (rowIndex === 0) {
                    return 'warning-row';
                }
                return '';
            },
            //选中项
            selsChange(sels) {
                this.sels = sels;
            },
            //加载树形结构
            loadTree() {
                this.$http.get("/product/productType/tree").then((res) => {
                    this.showProductTypes[0].children = this.setIcon(res.data);
                })
            },
            //加载所有的类型
            //递归的设置图标
            setIcon(arr) {
                for (let i = 0; i < arr.length; i++) {
                    if (!arr[i].children.length) {
                        arr[i].icon = "el-icon-location-information";
                    } else {
                        arr[i].icon = "el-icon-add-location";
                        this.setIcon(arr[i].children);
                    }
                }
                return arr;
            },
            //点击事件
            handleNodeClick(data, node, element) {
                this.$http.get("/product/specification/productType?productTypeId=" + data.id).then((res) => {
                    this.specifications = res.data;
                })
            },

            clearForm(){
                this.specification = {
                    id:null,
                    specName:null,
                    productTypeId:null,
                    isSku:null
                };
                this.$refs.specification.clearValidate();
                this.specificationVisible = false;
            },
            cancelForm(){
                this.clearForm();
            },
            submitForm(){
                this.$refs.specification.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.submitLoading = true;
                            let para = Object.assign({}, this.productType);
                            this.$http.post("/product/productType", para)
                                .then((result) => {
                                    //成功后的回调
                                    let data = result.data;
                                    if (data.success) {
                                        //成功
                                        this.$message({
                                            message: data.message,
                                            type: 'success'
                                        });
                                    } else {
                                        //失败
                                        this.$message.error(data.message);
                                    }
                                    this.specificationVisible = false;
                                    this.submitLoading = false;
                                    this.loadTree();
                                });
                        });
                    }
                });
            },
            handleAdd() {
                this.$confirm('添加需要选中父级类型','确认添加吗？', {}).then(() => {
                    this.specificationVisible = true;
                })
            },
            handleEdit(index,row) {
                this.$confirm('编辑需要选中当前类型', '确认编辑吗？',{}).then(() => {
                    this.specificationVisible = true;
                    this.$http.get("/product/productType/"+this.productType.id).then((res)=>{
                        this.productType = res.data;
                        if(!this.productType.pid){
                            this.productType.pname = "商品类型";
                        }
                    })
                })
            },

            //递归找出需要删除的id
            findDeleteIds(ids,obj){
                ids.push(obj.id);
                if(obj.children.length){
                    for(let i in obj.children){
                        this.findDeleteIds(ids,obj.children[i]);
                    }
                }
                return ids;
            },
            handleDelete(index,row) {
                this.$confirm('删除操作会删除当前类型及其子孙类型', '确认删除吗？',{}).then(() => {
                    let idss = [];
                    let ids = this.findDeleteIds(idss,this.deleteObject);
                    this.$http.delete("/product/productType/batch/"+ids).then((res)=>{
                        let data = res.data;
                        if(data.success){
                            this.$message({
                                type: 'success',
                                message: data.message
                            });
                        }else{
                            this.$message({
                                message: data.message,
                                type: 'error'
                            });
                        }
                        this.loadTree();
                    })
                })
            },

        },
        mounted() {
            this.loadTree();
        }
    }
</script>

<style scoped>
    .el-aside {
        border: 1px solid #ccc;
        border-right: none;
    }

    .el-main {
        border: 1px solid #ccc;
        padding: 0px;
    }

    .el-table .warning-row {
        background: oldlace;
    }

    .menuItem {
        display: block;
        line-height: 20px;
        text-align: center;
        margin-top: 10px;
    }

    .menu {
        height: 100px;
        width: 100px;
        position: absolute;
        border-radius: 10px;
        border: 1px solid #999999;
        background-color: #f4f4f4;
        padding-left: 0px;
    }

    li:hover {
        background-color: #1790ff;
        color: white;
    }
    .el-form-item {
        margin-bottom: 0px;
    }
</style>