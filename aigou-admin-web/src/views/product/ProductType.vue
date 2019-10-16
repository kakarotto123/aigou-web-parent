<template>
    <el-row>
        <el-col class="col col-left" span="6">
            <el-tree :data="productType" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
        </el-col>
        <el-col class="col col-right" span="17"></el-col>
    </el-row>
</template>

<script>
    export default {
        data(){
            return{
                productType:[],
                defaultProps: {
                    children: 'children',
                    label: 'name'
                }
            }
        },methods:{
            loadTypeTree(){
                this.$http.get("/product/productType/loadTypeTree").then(res=>{
                    this.productType = res.data;
                }).catch({});
            }
        },mounted(){
            this.loadTypeTree();
        }
    }
</script>

<style scoped>
    .col{
        border: 1px solid #ccc;
        height: 600px;
        margin-top: 20px;
    }
    .col-left{
        border-right: none;
        /*设置数据显示不超出界限*/
        overflow-x: hidden;
        overflow-y: scroll;
    }
    /*取出下拉条*/
    .col-left::-webkit-scrollbar {
        display: none;
    }
    .el-tree{
        border: none;
    }
</style>
