<template>
    <el-container style="height: 600px">
        <el-aside width="300px">
            <el-tree :data="productTypes" :props="defaultProps"
                     @node-click="handleNodeClick"
                     @node-contextmenu="rightClick"
                     :highlight-current="true">
                <span class="custom-tree-node" slot-scope="{ node, data }">
                    <span>
                        <i :class="data.icon"></i>{{ node.label }}
                    </span>
                </span>
            </el-tree>
            <!--鼠标右键菜单栏，其实就是添加一个基于鼠标位置的模态框而已 -->
            <div v-show="menuVisible">
                <ul id="menu" class="menu">
                    <li class="menuItem" @click="addProductType">
                        <i class="el-icon-circle-plus-outline"></i>添加
                    </li>
                    <li class="menuItem" @click="editProductType">
                        <i class="el-icon-edit"></i>编辑
                    </li>
                    <li class="menuItem" @click="deleteProductType">
                        <i class="el-icon-remove-outline"></i>删除
                    </li>
                </ul>
            </div>
        </el-aside>
        <el-main>
            <!---------------------------------列表----------------------------------->
            <el-table :data="showProductType" border :header-cell-style="cellStyle" :cell-style="cellStyle"
                      highlight-current-row v-loading="listLoading" style="width: 100%;"
                      :row-class-name="tableRowClassName" height="570" v-show="showProductType.length>0">
                <!-- <el-table-column type="selection" width="50">
                 </el-table-column>-->
                <el-table-column type="index" width="50">
                </el-table-column>
                <el-table-column prop="name" label="名称" sortable>
                </el-table-column>
                <el-table-column prop="parent.name" label="父级菜单">
                </el-table-column>
                <el-table-column prop="createTime" label="创建时间" :formatter="formatterCreateTime">
                </el-table-column>
                <el-table-column prop="totalCount" label="商品数量" sortable>
                </el-table-column>
                <el-table-column prop="seoTitle" label="分类标题">
                </el-table-column>
                <el-table-column prop="seoKeywords" label="分类关键字">
                </el-table-column>
                <el-table-column prop="description" label="描述">
                </el-table-column>
            </el-table>
        </el-main>

        <!-------------------------------新增/编辑--------------------------------->
        <el-dialog title="新增/编辑" :visible.sync="productTypeVisible" width="50%" center :close-on-click-modal="false" :before-close="clearForm">
            <el-form :model="productType" label-width="100px" :rules="productTypeRules" ref="productType">
                <el-form-item label="名称" prop="name">
                    <el-col :span="22">
                        <el-input v-model="productType.name" auto-complete="off" clearable></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="父级类型" prop="pid">
                    <el-col :span="22">
                        <el-select style="width: 100%" v-model="productType.pid" clearable filterable placeholder="请选择父级类型">
                            <el-option v-for="item in chooseProductTypes" :label="item.name" :value="item.id">
                            </el-option>
                        </el-select>
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
                <el-form-item label="分类标题">
                    <el-col :span="22">
                        <el-input v-model="productType.seoTitle" auto-complete="off" clearable></el-input>
                    </el-col>
                </el-form-item>
                <el-form-item label="分类关键字">
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
        </el-dialog>

    </el-container>
</template>

<script>
    export default {
        data() {
            var validateNumber = (rule, value, callback) => {
                if (!value) {
                    callback(new Error('请输入值'));
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
                menuVisible: false,
                listLoading: false,
                productTypeVisible:false,
                submitLoading:false,
                productTypes: [],
                chooseProductTypes:[],
                defaultProps: {
                    children: 'children',
                    label: 'name',
                },
                showProductType: [],
                productType:{
                    id:null,
                    name:null,
                    pid:null,
                    logo:null,
                    description:null,
                    sortIndex:null,
                    totalCount:null,
                    seoTitle:null,
                    seoKeywords:null
                },
                productTypeRules:{
                    name: [
                        { required:true, message: "请输入名称", trigger: 'blur' }
                    ],
                    pid: [
                        { required:true, message: "请选择父级类型", trigger: 'blur' }
                    ],
                    sortIndex: [
                        { required:true, validator: validateNumber, trigger: 'blur' }
                    ],
                    totalCount: [
                        { required:true, validator: validateNumber, trigger: 'blur' }
                    ],
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
            //加载树形结构
            loadTree() {
                this.$http.get("/product/productType/tree").then((res) => {
                    this.productTypes = this.setIcon(res.data);
                })
            },
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
                this.foo();
                this.$http.get("/product/productType/getProductType/" + data.id).then((res) => {
                    this.showProductType = [];
                    //this.showProductType.push(res.data);
                    this.showProductType = res.data;
                })
            },
            //右键事件
            rightClick(MouseEvent, object, Node, element) {
                // 鼠标右击触发事件
                this.menuVisible = false;
                // 先把模态框关死，目的是 第二次或者第n次右键鼠标的时候 它默认的是true
                this.menuVisible = true;
                // 显示模态窗口，跳出自定义菜单栏
                var menu = document.querySelector('#menu');
                menu.style.left = MouseEvent.clientX + 30 + 'px';
                // 给整个document添加监听鼠标事件，点击任何位置执行foo方法
                document.addEventListener('click', this.foo);
                menu.style.top = MouseEvent.clientY - 70 + 'px';
            },
            foo() {
                // 取消鼠标监听事件 菜单栏
                this.menuVisible = false;
                //关闭监听
                document.removeEventListener('click', this.foo);
            },

            clearForm(){
                this.productType = {
                    id:null,
                    name:null,
                    pid:null,
                    logo:null,
                    description:null,
                    sortIndex:null,
                    totalCount:null,
                    seoTitle:null,
                    seoKeywords:null
                };
                this.$refs.productType.clearValidate();
                this.productTypeVisible = false;
            },
            cancelForm(){
                this.clearForm();
            },
            submitForm(){

            },
            addProductType() {
                this.productTypeVisible = true;
            },
            editProductType() {

            },
            deleteProductType() {

            },

            formatterCreateTime: function (row, column) {
                return row.createTime ? this.formatDate(row.createTime) : ""
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

</style>