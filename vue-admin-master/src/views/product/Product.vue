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
                <el-form-item>
                    <el-button type="primary" @click="handleViewProperties">显示属性</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleSkuProperties">SKU属性</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">上架</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">下架</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="products" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
            <el-table-column type="selection" width="55">
            </el-table-column>
            <el-table-column type="index" width="60">
            </el-table-column>
            <el-table-column prop="name" label="标题" width="250" sortable>
            </el-table-column>
            <el-table-column prop="subName" label="副标题" width="150" sortable>
            </el-table-column>
            <el-table-column prop="pt.name" label="商品类型" width="100" sortable>
            </el-table-column>
            <el-table-column prop="brand.name" label="商品品牌" width="100" sortable>
            </el-table-column>
            <el-table-column prop="onSaleTime" label="上架时间" width="150" :formatter="formatterOnSaleTime" sortable>
            </el-table-column>
            <el-table-column prop="offSaleTime" label="下架时间" width="150" :formatter="formatterOffSaleTime" sortable>
            </el-table-column>
            <el-table-column prop="state" label="状态" min-width="50" sortable>
                <template scope="scope">
                    <span v-if="scope.row.state==0" style="color:red">下架</span>
                    <span v-else style="color:green">上架</span>
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
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="size" :total="total" style="float:right;">
            </el-pagination>
        </el-col>

        <!--编辑界面-->
        <el-dialog title="编辑" v-model="editFormVisible" :close-on-click-modal="false">
            <el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
                <el-form-item label="姓名" prop="name">
                    <el-input v-model="editForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="性别">
                    <el-radio-group v-model="editForm.sex">
                        <el-radio class="radio" :label="1">男</el-radio>
                        <el-radio class="radio" :label="0">女</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="年龄">
                    <el-input-number v-model="editForm.age" :min="0" :max="200"></el-input-number>
                </el-form-item>
                <el-form-item label="生日">
                    <el-date-picker type="date" placeholder="选择日期" v-model="editForm.birth"></el-date-picker>
                </el-form-item>
                <el-form-item label="地址">
                    <el-input type="textarea" v-model="editForm.addr"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
            </div>
        </el-dialog>

        <!--新增界面-->
        <el-dialog title="新增" :visible.sync="addFormVisible" :close-on-click-modal="false">
            <el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
                <el-form-item label="标题" prop="name">
                    <el-input v-model="addForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="副标题" prop="subName">
                    <el-input v-model="addForm.subName" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="类型">
                    <el-cascader style="width:100%"
                                 :options="productTypes"
                                 v-model="addForm.productType"
                                 :props="productProps">
                    </el-cascader>
                </el-form-item>
                <el-form-item label="品牌">
                    <el-input v-model="addForm.brandId"></el-input>
                </el-form-item>
                <el-form-item label="媒体属性">
                    <el-upload
                            class="upload-demo"
                            action="http://localhost:9527/services/common/file/upload"
                            :file-list="mediaList"
                            :on-success="handleUploadSeccess"
                            list-type="picture">
                        <el-button size="small" type="primary">点击上传</el-button>
                    </el-upload>
                </el-form-item>
                <el-form-item label="商品描述">
                    <el-input v-model="addForm.description"></el-input>
                </el-form-item>
                <el-form-item label="商品详情">
                    <!--<quill-editor
                            v-model="addForm.content"
                            ref="myQuillEditor"
                            :options="editorOption">
                    </quill-editor>-->
                    <SquillEditorFastdfs host="http://192.168.1.3" v-model="addForm.content" uploadUrl='http://localhost:9527/services/common/file/upload'></SquillEditorFastdfs>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>



        <!-------------------------------------------显示属性维护的模态框-------------------------------------->
        <el-dialog
                title="显示属性管理"
                :visible.sync="viewDialogVisible"
                width="40%">

            <!--卡片-->
            <el-card class="box-card">
                <div v-for="index in viewProperties.length" :key="index" class="text item" style="margin-bottom: 5px">
                    <el-row>
                        <el-col :span="3">{{viewProperties[index-1].specName}}:</el-col>
                        <el-col :span="21">
                            <el-input v-model="viewProperties[index-1].value"></el-input>
                        </el-col>
                    </el-row>
                </div>
            </el-card>


            <span slot="footer" class="dialog-footer">
                <el-button @click="viewDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="subViewProperties">确 定</el-button>
          </span>
        </el-dialog>

        <!-------------------------------------------SKU属性维护的模态框-------------------------------------->
        <el-dialog
                title="SKU属性管理"
                :visible.sync="skuDialogVisible"
                width="60%">

            <!--卡片     外层的卡片代表者一个sku属性-->
            <el-card class="box-card" v-for="skuProperty in skuProperties" style="margin-bottom: 5px">
                <!--内层的div代表sku属性的选项-->
                <div slot="header" class="clearfix">
                    <span>{{skuProperty.specName}}</span>
                </div>
                <div v-for="index in skuProperty.options.length+1" :key="index" class="text item">
                    <el-input v-model="skuProperty.options[index-1]" style="width:80%"></el-input>
                    <el-button icon="el-icon-delete" @click="skuProperty.options.splice(index-1,1)" style="width:10%"></el-button>
                </div>
            </el-card>

            <el-table
                    :data="skus"
                    border
                    style="width: 100%">
                <el-table-column
                        type="index"
                        width="50">
                </el-table-column>
                <template v-for="(value,key) in skus[0]">
                    <!--更改价格和库存单元格和标题-->
                    <el-table-column v-if="key=='price'"
                                     :prop="key"
                                     label="价格">
                        <template scope="scope">
                            <el-input v-model="scope.row.price"></el-input>
                        </template>
                    </el-table-column>
                    <el-table-column v-else-if="key=='availableStock'"
                                     :prop="key"
                                     label="库存">
                        <template scope="scope">
                            <el-input v-model="scope.row.availableStock"></el-input>
                        </template>
                    </el-table-column>
                    <!--隐藏sku_index属性-->
                    <el-table-column v-else-if="key!='sku_index'"
                                     :prop="key"
                                     :label="key">
                    </el-table-column>
                </template>
            </el-table>


            <span slot="footer" class="dialog-footer">
                <el-button @click="skuDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="subSkuProperties">确 定</el-button>
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
            return {
                skus:[],
                skuProperties:[],
                skuDialogVisible:false,//sku属性模态框
                viewProperties:[],
                viewDialogVisible:false,//显示属性模态框
                editorOption:{},
                //商品类型
                productTypes: [],
                //级联选择器的属性配置
                productProps: {
                    label: "name",
                    value: "id"
                },
                mediaList:[],
                filters: {
                    keyword: ''
                },
                products: [],
                total: 0,
                page: 1,
                size:10,
                listLoading: false,
                sels: [],//列表选中行

                editFormVisible: false,//编辑界面是否显示
                editLoading: false,
                editFormRules: {
                    name: [
                        { required: true, message: '请输入姓名', trigger: 'blur' }
                    ]
                },
                //编辑界面数据
                editForm: {
                    id: 0,
                    name: '',
                    sex: -1,
                    age: 0,
                    birth: '',
                    addr: ''
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
                    name:'',
                    subName:'',
                    productType:null,
                    brandId:null,
                    maxPrice:null,
                    minPrice:null,
                    description:'',
                    content:'',
                    mediasArr:[]
                }

            }
        },
        methods: {
            //保存sku
            subSkuProperties(){
                //准备请求参数
                let para = {}
                para.skuProperties = this.skuProperties;//保存到商品表
                para.productId = this.sels[0].id;
                para.skus = this.skus;//添加到sku表中
                //发送请求
                this.$confirm('确认提交吗？', '提示', {}).then(() => {
                    this.$http.post("/product/product/skuProperties",para).then(res=>{
                        let data = res.data;
                        if(data.success){
                            this.$message({
                                message: '保存成功!',
                                type: 'success'
                            });
                            //关闭模态框
                            this.skuDialogVisible = false;
                        }else{
                            this.$message({
                                message: data.message,
                                type: 'error'
                            });
                        }
                    })
                })
            },
            handleSkuProperties(){
                //判断是否选中一行属性
                if(this.sels.length!=1){
                    this.$message({
                        message: '只能选中一行商品进行操作!',
                        type: 'warning'
                    });
                    return;
                }
                //查询sku属性以及属性列表
                let productId = this.sels[0].id;
                this.$http.get("/product/product/skuProperties?productId="+productId).then(res=>{
                    this.skuProperties = res.data;
                })
                this.skuDialogVisible = true;
            },
            //保存显示属性
            subViewProperties(){
                //准备请求参数
                let para = {}
                para.viewProperties = this.viewProperties;
                para.productId = this.sels[0].id;
                //发送请求
                this.$confirm('确认提交吗？', '提示', {}).then(() => {
                    this.$http.post("/product/product/viewProperties",para).then(res=>{
                        let data = res.data;
                        if(data.success){
                            this.$message({
                                message: '保存成功!',
                                type: 'success'
                            });
                            //关闭模态框
                            this.viewDialogVisible = false;
                        }else{
                            this.$message({
                                message: data.message,
                                type: 'error'
                            });
                        }
                    })
                })
            },
            //点击显示属性
            handleViewProperties(){
                //判断是否选中一行属性
                if(this.sels.length!=1){
                    this.$message({
                        message: '只能选中一行商品进行操作!',
                        type: 'warning'
                    });
                    return;
                }


              //发送请求请求viewProperties
              let productId = this.sels[0].id;
              this.$http.get("/product/product/viewProperties?productId="+productId).then(res=>{
                  this.viewProperties = res.data;//后台要返回一个[{"id":2,"name":"产品名称","value":"越南新娘"},{"id":4,"name":"样式","value":"肥胖"}]格式的数组
              })
              this.viewDialogVisible = true;
            },
            //文件上传成功钩子函数
            handleUploadSeccess(response){
                this.addForm.mediasArr.push(response.data)
            },
            //加载商品类型
            getProductTypes() {
                this.$http.get("/product/productType/tree").then((res) => {
                    this.productTypes = this.getTreeData(res.data);
                })
            },
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
            //上架时间格式化
            formatterOnSaleTime:function(row, column){
                return row.onSaleTime?this.formatDate(row.onSaleTime):""
            },
            //下架时间格式化
            formatterOffSaleTime:function(row, column){
                return row.offSaleTime?this.formatDate(row.offSaleTime):""
            },
            //时间显示转换
            formatDate(longTime){
                let date = new Date(longTime);
                let year = date.getFullYear();//2019
                let month = date.getMonth()+1;//月份从0开始
                let day = date.getDate();//日
                let hour = date.getHours();
                let minute = date.getMinutes();
                let second = date.getSeconds();
                return year+"-"+this.numberFormatter(month)+"-"+this.numberFormatter(day)+" "
                    +this.numberFormatter(hour)+":"+this.numberFormatter(minute)+":"+this.numberFormatter(second);
            },
            numberFormatter(num){
                if(num<10){
                    return "0"+num;
                }
                return num;
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
                    size:this.size,
                    keyword: this.filters.keyword
                };
                this.listLoading = true;
                this.$http.post("/product/product/page",para).then(res=>{
                    let data = res.data;
                    this.total = data.total;
                    this.products = data.rows;
                    this.listLoading = false;
                });
            },
            //删除
            handleDel: function (index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    let para = { id: row.id };
                    removeUser(para).then((res) => {
                        this.listLoading = false;
                        //NProgress.done();
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                        this.getProducts();
                    });
                }).catch(() => {

                });
            },
            //显示编辑界面
            handleEdit: function (index, row) {
                this.editFormVisible = true;
                this.editForm = Object.assign({}, row);
            },
            //显示新增界面
            handleAdd: function () {
                this.addFormVisible = true;
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
            //新增
            addSubmit: function () {
                this.$refs.addForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.addLoading = true;
                            //NProgress.start();
                            let para = Object.assign({}, this.addForm);
                            //medias
                            para.medias = this.arrToString(this.addForm.mediasArr);
                            para.productType = this.addForm.productType[this.addForm.productType.length-1];
                            //发送请求
                            this.$http.post("/product/product",para).then(res=>{
                                this.addLoading = false;
                                let data  = res.data;
                                if(data.success){
                                    this.$message({
                                        message: '保存成功',
                                        type: 'success'
                                    });
                                    this.$refs['addForm'].resetFields();
                                    this.addFormVisible = false;
                                    this.getProducts();
                                }else{
                                    this.$message({
                                        message: '保存失败',
                                        type: 'error'
                                    });
                                }

                            })

                        });
                    }
                });
            },
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
            //当table的复选框发送选中或者取消选中时的回调
            selsChange: function (sels) {
                console.log("sels",sels)
                this.sels = sels;
            },
            //批量删除
            batchRemove: function () {
                var ids = this.sels.map(item => item.id).toString();
                this.$confirm('确认删除选中记录吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    let para = { ids: ids };
                    batchRemoveUser(para).then((res) => {
                        this.listLoading = false;
                        //NProgress.done();
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                        this.getProducts();
                    });
                }).catch(() => {

                });
            }
        },
        mounted() {
            this.getProducts();
            this.getProductTypes();
        },
        watch:{
            skuProperties:{//深度监听，可监听到对象、数组的变化
                handler(val, oldVal){
                    //filter es6中数组的一个方法 过滤返回一个新的数组
                    //去掉没有值的属性
                    //let tempSkuProperties = this.skuProperties.filter(o=>o.options.length>0)
                    //动态生成skus值
                    //TODO  某个属性没有现象的时候前端bug
                    let res = this.skuProperties.reduce((pre,cur)=>{
                        let result = [];
                        pre.forEach(e1=>{
                            e1.sku_index = (e1.sku_index||'')+"_";
                            for(let i=0;i<cur.options.length;i++){
                                let e2 = cur.options[i];
                                let temp = {}
                                Object.assign(temp,e1);
                                temp[cur.specName] = e2;
                                temp.sku_index += i;
                                result.push(temp);
                            }
                        })
                        return result;
                    },[{}])

                    //添加价格
                    res.forEach(e1=>{
                        e1.price = 0;
                        e1.availableStock = 0;
                        e1.sku_index = e1.sku_index.substring(1);
                    })

                    this.skus = res;
                },
                deep:true
            }
        }
    }

</script>

<style scoped>

</style>