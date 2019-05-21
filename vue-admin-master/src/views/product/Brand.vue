<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input v-model="filters.keyword" placeholder="关键字"></el-input>
                </el-form-item>
                <el-form-item label="商品类型" prop="productTypeId">
                    <el-cascader style="width:300px"
                                 :change-on-select="true"
                                 :options="productTypes"
                                 v-model="filters.pType"
                                 :props="productProps">
                    </el-cascader>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" round v-on:click="getBrands">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="info" round v-on:click="handleClearn">重置</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="success" round v-on:click="handleAdd">新增</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="brands" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
            <el-table-column type="selection" width="55">
            </el-table-column>
            <el-table-column type="index" width="60">
            </el-table-column>
            <el-table-column prop="name" label="名称" sortable>
            </el-table-column>
            <el-table-column prop="englishName" label="英文名" sortable>
            </el-table-column>
            <el-table-column prop="firstLetter" label="首字母" sortable>
            </el-table-column>
            <el-table-column prop="sortIndex" label="序号" sortable>
            </el-table-column>
            <el-table-column prop="logo" label="品牌logo"  sortable>
            </el-table-column>
            <el-table-column prop="productType.name" label="商品类型" sortable>
            </el-table-column>
            <el-table-column prop="description" label="描述" sortable>
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
        <el-dialog title="编辑" :visible.sync="editFormVisible" :close-on-click-modal="false">
            <el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
                <el-form-item label="商品名称" prop="name">
                    <el-input v-model="editForm.name"></el-input>
                </el-form-item>
                <el-form-item label="英文名" prop="englishName">
                    <el-input v-model="editForm.englishName"></el-input>
                </el-form-item>
                <el-form-item label="商品类型" prop="productTypeId">
                    <el-cascader style="width:100%"
                                 :change-on-select="true"
                                 :options="productTypes"
                                 v-model="addForm.productTypeId"
                                 :props="productProps">
                    </el-cascader>
                </el-form-item>
                <el-form-item label="排序" prop="sortIndex">
                    <el-input  v-model="editForm.sortIndex" ></el-input>
                </el-form-item>
                <el-form-item label="logo">
                    <el-upload
                            class="upload-demo"
                            action="http://localhost:9527/services/common/file/upload"
                            :before-remove="handleLogoRemove"
                            :file-list="logoList"
                            list-type="picture"
                            :limit="1"
                            :on-change="handelChange"
                            :on-success="handleUploadSeccess"
                            :auto-upload="true"
                            :on-exceed="handleOutOfRange">
                        <el-button size="small" type="primary">点击上传</el-button>
                    </el-upload>
                </el-form-item>
                <el-form-item label="品牌描述">
                    <el-input type="textarea" v-model="editForm.description"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="clearEditFrom">取消</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>

        <!--新增界面-->
        <el-dialog title="新增" :visible.sync="addFormVisible" :close-on-click-modal="false">
            <el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
                <el-form-item label="商品名称" prop="name">
                    <el-input v-model="addForm.name"></el-input>
                </el-form-item>
                <el-form-item label="英文名" prop="englishName">
                    <el-input v-model="addForm.englishName"></el-input>
                </el-form-item>
                <el-form-item label="商品类型" prop="productTypeId">
                    <el-cascader style="width:100%"
                                 :change-on-select="true"
                                 :options="productTypes"
                                 v-model="addForm.productTypeId"
                                 :props="productProps">
                    </el-cascader>
                </el-form-item>
                <el-form-item label="排序" prop="sortIndex">
                    <el-input  v-model="addForm.sortIndex" ></el-input>
                </el-form-item>
                <el-form-item label="logo">
                    <el-upload
                            class="upload-demo"
                            action="http://localhost:9527/services/common/file/upload"
                            :before-remove="handleLogoRemove"
                            :file-list="logoList"
                            list-type="picture"
                            :limit="1"
                            :on-change="handelChange"
                            :on-success="handleUploadSeccess"
                            :auto-upload="true"
                            :on-exceed="handleOutOfRange">
                        <el-button size="small" type="primary">点击上传</el-button>
                    </el-upload>
                </el-form-item>
                <el-form-item label="品牌描述">
                    <el-input type="textarea" v-model="addForm.description"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="clearAddFrom">取消</el-button>
                <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>

    import qs from 'qs'

    export default {
        data() {
            return {
                //文件是否立即上传
                // autoupload:false,
                //级联选择器的属性配置
                productProps: {
                    label: "name",//就是显示的内容 显示名字
                    value: "id" //绑定的字段 绑定到 id字段
                },
                filters: {
                    keyword: '',
                    pType:null
                },
                brands: [],
                total: 0,
                page: 1,
                size:10,
                listLoading: false,
                sels: [],//列表选中列
                //logo 图片
                logoList: [],
                productTypes:[],//获取到所有的商品类型

                editFormVisible: false,//编辑界面是否显示
                editLoading: false,
                editFormRules: {
                    name: [
                        { required: true, message: '请输入姓名', trigger: 'blur' }
                    ]
                },
                //编辑界面数据
                editForm: {
                    id:"",
                    createTime:"",
                    updateTime:"",
                    name:"",
                    englishName:"",
                    firstLetter:"",
                    description:"",
                    productTypeId:"",
                    sortIndex:"",
                    logo:""
                },

                addFormVisible: false,//新增界面是否显示
                addLoading: false,
                addFormRules: {
                    name: [
                        { required: true, message: '请输入姓名', trigger: 'blur' }
                    ],
                    englishName: [
                        { required: true, message: '请输入英文名', trigger: 'blur' }
                    ],
                    englishName: [
                        { required: true, message: '请输入英文名', trigger: 'blur' }
                    ],
                    sortIndex: [
                        {required: true, message: '请输入序号', trigger: 'blur'}
                    ],
                    productTypeId: [
                        {required: true, message: '请选择商品类型', trigger: 'blur'}
                    ]
                },
                //新增界面数据
                addForm: {
                    id:"",
                    createTime:"",
                    updateTime:"",
                    name:"",
                    englishName:"",
                    firstLetter:"",
                    description:"",
                    productTypeId:"",
                    sortIndex:"",
                    logo:""
                }

            }
        },
        methods: {
            //清空高级查询条件
            handleClearn(){
                this.filters={};
                //重新刷新页面
                this.getBrands();
            },
            //文件上传成功钩子函数
            handleUploadSeccess(response){
                this.addForm.logo = response.data;
            },
            //文件超出个数
            handleOutOfRange(){
                this.$message({
                    message: '只能上传一张图片',
                    type: 'warning'
                });
            },
            //文件上传时。把文件保存到 logoList中
            handelChange(file, fileList){
                this.logoList = fileList;
            },
            // handleBeforeUpload(file){
            //     console.debug(file)
            //     return false;//阻止上传
            // },
            changeDetSelect(key,treeData){
                let arr = []; // 在递归时操作的数组
                let returnArr = []; // 存放结果的数组
                let depth = 0; // 定义全局层级
                // 定义递归函数
                function childrenEach(childrenData, depthN) {
                    for (var j = 0; j < childrenData.length; j++) {
                        depth = depthN; // 将执行的层级赋值 到 全局层级
                        arr[depthN] = (childrenData[j].id);
                        if (childrenData[j].id == key) {
                            returnArr = arr.slice(0, depthN+1); //将目前匹配的数组，截断并保存到结果数组，
                            break
                        } else {
                            if (childrenData[j].nodes) {
                                depth ++;
                                childrenEach(childrenData[j].nodes, depth);
                            }
                        }
                    }
                    return returnArr;
                }
                return childrenEach(treeData, depth);
            },
            //显示编辑界面
            handleEdit: function (index, row) {
                this.editFormVisible = true;
                //回填表单信息
                this.editForm = Object.assign({}, row);
                //回填 类型
                console.debug(row.id);
                //通过id查询当前 行数据的类型.主要输查询类型。
                this.$http.get("/product/brand/"+row.id).then(res=>{

                    console.debug(this.productTypeId+"   res");
                    this.productProps.id = this.productTypeId;
                    if(res){
                        this.$message({
                            message: '查询成功',
                            type: 'success'
                        });
                    }else{
                        this.$message({
                            message: '查询失败',
                            type: 'error'
                        });
                    }
                })
                // this.productProps.name = this.

                // console.debug(this.productTypes.path,"000000");
                //回填 图片

            },
            clearEditFrom(){
                console.debug(this.logoList,"12351351");
                if(this.logoList.length>0){//当有时才删除
                    this.handleLogoRemove(this.logoList[0]);
                }
                //清空表单
                this.addForm = {};
                this.logoList=[];//清空 logo
                this.$refs['editForm'].resetFields();//重置验证规则 有bug
                this.editFormVisible= false;//关闭模态框
            },
            //点击取消按钮时【删除上传的文件】
            clearAddFrom(){
                // console.debug(this.logoList,"12351351");
                if(this.logoList.length>0){//当有时才删除
                    // console.debug("12351351");
                    this.handleLogoRemove(this.logoList[0]);
                }
                //清空表单
                this.addForm = {};
                this.logoList=[];//清空 logo
                this.$refs['addForm'].resetFields();//重置验证规则 有bug
                this.addFormVisible= false;//关闭模态框
            },
            //显示新增界面
            handleAdd: function () {
                this.addFormVisible = true;

            },
            //删除文件
            handleLogoRemove(file, fileList){
                this.$http.get("/common/file/delete",{
                    params:{
                        fileId:file.response.data
                    }
                }).then(res=>{
                    let data = res.data;
                    if(data.success){
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                    }else{
                        this.$message({
                            message: '删除失败',
                            type: 'error'
                        });
                        return false;//阻止删除
                    }
                })
            },
            //分页操作
            handleCurrentChange(val) {
                this.page = val;
                this.getBrands();
            },
            //获取商品品牌列表
            getBrands() {
                this.listLoading = true;
                console.debug(this.filters.pType);
                //NProgress.start();
                var objectpType = Object.assign({},this.filters.pType);
                console.debug(objectpType);
                this.$http.post("/product/brand/page",{
                        page:this.page,
                        size:this.size,
                        keyword:this.filters.keyword,
                        pType:objectpType
                }).then((res)=>{
                    this.listLoading = false;
                    let data = res.data;//PageList

                    this.brands = data.rows;
                    this.total = data.total;
                })
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
            //删除
            handleDel: function (index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    // let id =  row.id ;
                    // console.debug(para);
                    this.$http.delete("/product/brand/"+row.id).then(res=>{
                        this.listLoading = false;
                        //NProgress.done();
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                        this.getBrands();
                    });
                }).catch(() => {
                    this.$message({
                        message: '删除失败成功',
                        type: 'error'
                    });
                    this.getBrands();
                });
            },
            //编辑
            editSubmit: function () {
                this.$refs.editForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.editLoading = true;
                            this.$http.post("/product/brand",this.editForm).then(res=>{
                                this.addLoading = false;
                                this.$message({
                                    message: '编辑成功',
                                    type: 'success'
                                });
                                this.$refs['editForm'].resetFields();//刷新验证规则
                                this.editFormVisible = false;//关闭模态框
                                this.getBrands();
                            });
                        });
                    }
                });
            },
            //新增
            addSubmit: function () {
                this.$refs.addForm.validate((valid) => {//验证通过
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.addLoading = true;
                            // this.autoupload=true;
                            //格式化部分参数  productTypeId=[1,2,3]  ===> productTypeId=1
                            let para = Object.assign({}, this.addForm);
                            //只获取最后 一级 的id
                            para.productTypeId = para.productTypeId[para.productTypeId.length - 1];
                            this.$http.post("/product/brand",para).then(res=>{
                                this.addLoading = false;
                                    this.$message({
                                        message: '新增成功',
                                        type: 'success'
                                    });
                                    this.$refs['addForm'].resetFields();//刷新验证规则
                                    this.addFormVisible = false;//关闭模态框
                                    this.getBrands();
                            });
                        });
                    }
                });
            },
            selsChange: function (sels) {
                this.sels = sels;
            },
            //批量删除
            batchRemove: function (index, row) {
                var ids = this.sels.map(item => item.id).toString();
                let para = ids;
                // this.listLoading = true;
                this.$confirm('确认删除选中记录吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    // this.listLoading = false;
                    //发送删除请求
                    this.$http.delete("/product/brand/ids", {
                        params: {
                            ids: para,
                            type: 1
                        },
                        paramsSerializer: params => {
                            return qs.stringify(params, { indices: false })
                        }})
                    this.getBrands();
                }).catch(() => {
                    this.$message({
                        message: '删除失败',
                        type: 'error'
                    });
                });
            }
        },
        //$(function(){})
        mounted() {
            //查询商品品牌
            this.getBrands();
            //查询所有的商品类型
            this.getProductTypes();
        }
    }

</script>

<style scoped>

</style>