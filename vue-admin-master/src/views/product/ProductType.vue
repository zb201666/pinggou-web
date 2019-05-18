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
                    <li class="menu__item" @click="addProductType">
                        <i class="el-icon-circle-plus-outline"></i>添加
                    </li>
                    <li class="menu__item" @click="editProductType">
                        <i class="el-icon-edit"></i>编辑
                    </li>
                    <li class="menu__item" @click="deleteProductType">
                        <i class="el-icon-remove-outline"></i>删除
                    </li>
                </ul>
            </div>
        </el-aside>
        <el-main>

        </el-main>
    </el-container>


</template>

<script>
    export default {
        data() {
            return {
                menuVisible: false,
                productTypes: [],
                defaultProps: {
                    children: 'children',
                    label: 'name',
                }
            }
        },
        methods: {
            //加载树形结构
            loadTree() {
                this.$http.get("/product/productType/tree").then((res) => {
                    this.productTypes = this.setIcon(res.data);
                })
            },
            //递归的设置图标
            setIcon(arr) {
                for (let i = 0; i < arr.length; i++) {
                    if (arr[i].children.length) {
                        arr[i].icon = "el-icon-location-information";
                    } else {
                        arr[i].icon = "el-icon-add-location";
                        this.setIcon(arr[i].children);
                    }
                }
                return arr;
            },
            handleNodeClick() {
                this.foo();
            },
            //右键事件
            rightClick(MouseEvent, object, Node, element) {
                // 鼠标右击触发事件
                this.menuVisible = false;
                // 先把模态框关死，目的是 第二次或者第n次右键鼠标的时候 它默认的是true
                this.menuVisible = true;
                // 显示模态窗口，跳出自定义菜单栏
                var menu = document.querySelector('#menu');
                menu.style.left = MouseEvent.clientX +30 + 'px';
                // 给整个document添加监听鼠标事件，点击任何位置执行foo方法
                document.addEventListener('click', this.foo);
                menu.style.top = MouseEvent.clientY - 70 + 'px';
                /*console.debug('右键被点击的event:', MouseEvent);
                console.debug('右键被点击的object:', object);
                console.debug('右键被点击的value:', Node);
                console.debug('右键被点击的element:', element);*/
            },
            foo() {
                // 取消鼠标监听事件 菜单栏
                this.menuVisible = false;
                //关闭监听
                document.removeEventListener('click', this.foo);
            },
            addProductType() {

            },
            editProductType() {

            },
            deleteProductType() {

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

    .menu__item {
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
    }

    li:hover {
        background-color: #1790ff;
        color: white;
    }

</style>