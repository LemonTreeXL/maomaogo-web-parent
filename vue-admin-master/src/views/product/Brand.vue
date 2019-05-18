<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input v-model="filters.keyword" placeholder="关键字"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getBrands">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="handleAdd">新增</el-button>
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
                    <el-input v-model="editForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="性别">
                    <el-radio-group v-model="editForm.sex">
                        <el-radio class="radio" :label="1">男</el-radio>
                        <el-radio class="radio" :label="0">女</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="英文名称">
                    <el-input-number v-model="editForm.englishName" :min="0" :max="200"></el-input-number>
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
                <el-form-item label="商品名称" prop="name">
                    <el-input v-model="addForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <!--<el-form-item label="性别">-->
                    <!--<el-radio-group v-model="addForm.sex">-->
                        <!--<el-radio class="radio" :label="1">男</el-radio>-->
                        <!--<el-radio class="radio" :label="0">女</el-radio>-->
                    <!--</el-radio-group>-->
                <!--</el-form-item>-->
                <el-form-item label="英文名称">
                    <el-input v-model="addForm.englishName"></el-input>
                </el-form-item>
                <el-form-item label="创建时间">
                    <el-date-picker type="date" placeholder="选择日期" v-model="addForm.createTime"></el-date-picker>
                </el-form-item>
                <el-form-item label="更新时间">
                    <el-date-picker type="date" placeholder="选择日期" v-model="addForm.updateTime"></el-date-picker>
                </el-form-item>
                <el-form-item label="首字母">
                    <el-input  v-model="addForm.firstLetter"></el-input>
                </el-form-item>
                <el-form-item label="品牌描述">
                    <el-input type="textarea" v-model="addForm.description"></el-input>
                </el-form-item>
                <el-form-item label="品牌类型">
                    <el-input  v-model="addForm.product_type_id"></el-input>
                </el-form-item>
                <el-form-item label="排序">
                    <el-input  v-model="addForm.sortIndex"></el-input>
                </el-form-item>
                <el-form-item label="LOGO">
                    <el-input v-model="addForm.logo"></el-input>
                </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addFormVisible = false">取消</el-button>
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
                filters: {
                    keyword: ''
                },
                brands: [],
                total: 0,
                page: 1,
                size:10,
                listLoading: false,
                sels: [],//列表选中列

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
                    product_type_id:"",
                    sortIndex:"",
                    logo:""
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
                    id:"",
                    createTime:"",
                    updateTime:"",
                    name:"",
                    englishName:"",
                    firstLetter:"",
                    description:"",
                    product_type_id:"",
                    sortIndex:"",
                    logo:""
                }

            }
        },
        methods: {
            //性别显示转换
            formatSex: function (row, column) {
                return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
            },
            //分页操作
            handleCurrentChange(val) {
                this.page = val;
                this.getBrands();
            },
            //获取商品品牌列表
            getBrands() {
                this.listLoading = true;
                //NProgress.start();

                this.$http.post("/product/brand/page",{
                        page:this.page,
                        size:this.size,
                        keyword:this.filters.keyword
                }).then((res)=>{
                    this.listLoading = false;
                    let data = res.data;//PageList

                    this.brands = data.rows;
                    this.total = data.total;
                })

                /*
                getUserListPage(para).then((res) => {
                    this.total = res.data.total;
                    this.users = res.data.users;
                    this.listLoading = false;
                    //NProgress.done();
                });*/
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
            //显示编辑界面
            handleEdit: function (index, row) {
                this.editFormVisible = true;
                //回填表单信息
                this.editForm = Object.assign({}, row);
            },
            //显示新增界面
            handleAdd: function () {
                this.addFormVisible = true;
                //清空表单
                this.addForm = {}; //有bug
                // this.$refs['addForm'].resetFields();//重置验证规则
                // this.addForm.id="",
                // this.addForm.createTime="",
                // this.addForm.updateTime="",
                // this.addForm.name="",
                // this.addForm.englishName="",
                // this.addForm.firstLetter="",
                // this.addForm.description="",
                // this.addForm.product_type_id="",
                // this.addForm.sortIndex="",
                // this.addForm.logo=""
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
                            this.$http.post("/product/brand",{
                                brand:this.addForm
                            }).then(res=>{
                                this.addLoading = false;
                                    this.$message({
                                        message: '提交成功',
                                        type: 'success'
                                    });
                                    this.$refs['addForm'].resetFields();//刷新验证规则
                                    this.addFormVisible = false;//关闭模态框
                                    this.getBrands();
                            });
                            //NProgress.start();
                            // let para = Object.assign({}, this.addForm);
                            // para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
                            // addUser(para).then((res) => {
                            //     this.addLoading = false;
                            //     //NProgress.done();
                            //     this.$message({
                            //         message: '提交成功',
                            //         type: 'success'
                            //     });
                            //     this.$refs['addForm'].resetFields();
                            //     this.addFormVisible = false;
                            //     this.getBrands();
                            // });
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
                this.$confirm('确认删除选中记录吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    //发送删除请求
                    this.$http.delete("/product/brand/ids", {
                        params: {
                            ids: para,
                            type: 1
                        },
                        paramsSerializer: params => {
                            return qs.stringify(params, { indices: false })
                        }})

/*
                    this.$http.post("/product/brand/ids",{para}).then(res=>{
                        this.$message({
                            message: '批量删除成功',
                            type: 'success'
                        });
                        this.getBrands();
                    });*/
                }).catch(() => {
                    this.$message({
                        message: '删除失败',
                        type: 'error'
                    });
                    this.getBrands();
                });
            }
        },
        //$(function(){})
        mounted() {
            //查询商品品牌
            this.getBrands();
            //查询所有的商品类型

        }
    }

</script>

<style scoped>

</style>