<template>
    <div style="height: 630px">
        <el-row style="height: 100%">
            <el-col class="container-left" :span="5">
                <el-tree >

                </el-tree>
            </el-col>
            <el-col class="container-right" :span="19">

            </el-col>
        </el-row>
    </div>
</template>

<script>
    export default {
        name: "product-type.vue"
    }
</script>

<style scoped>
    .container-left{
        height: 100%;
        border:1px solid #ccc;
        border-right:none;
    }
    .container-right{
        height: 100%;
        border:1px solid #ccc
    }
</style>