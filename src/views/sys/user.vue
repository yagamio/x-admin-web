<template>
    <div>
        <!-- Search 搜索栏 -->
        <el-card id="search">
            <el-row>
                <el-col :span="20">
                    <el-input v-model="searchModel.username" placeholder="Username" clearable></el-input>
                    <el-input v-model="searchModel.phone" placeholder="Phone number" clearable></el-input>
                    <el-button @click="getUserList" type="primary" round icon="el-icon-search">Search</el-button>
                </el-col>
                <el-col :span="4" align="right">
                    <el-button @click="openEditUI" type="primary" icon="el-icon-plus" circle></el-button>
                </el-col>
            </el-row>
        </el-card>

        <!-- Result List 结果列表 -->
        <el-card>
            <el-table :data="userList" stripe style="width: 100%">
                <el-table-column label="#" width="180">
                    <template slot-scope="scope">
                        <!-- (pageNo - 1) * pageSize + index + 1 -->
                        {{ (searchModel.pageNo - 1) * searchModel.pageSize + scope.$index + 1 }}
                    </template>
                </el-table-column>
                <el-table-column prop="id" label="User ID" width="180">
                </el-table-column>
                <el-table-column prop="username" label="Username" width="180">
                </el-table-column>
                <el-table-column prop="phone" label="Phone Number" width="180">
                </el-table-column>
                <el-table-column prop="status" label="Status" width="180">
                    <template slot-scope="scope">
                        <el-tag v-if="scope.row.status == 1">In use</el-tag>
                        <el-tag v-else type="danger">Frozen</el-tag>
                    </template>
                </el-table-column>
                <el-table-column prop="email" label="email">
                </el-table-column>
                <el-table-column label="Operate">
                </el-table-column>
            </el-table>
        </el-card>

        <!-- Pagination -->
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
            :current-page="searchModel.pageNo" :page-sizes="[5, 10, 20, 50]" :page-size="searchModel.pageSize"
            layout="total, sizes, prev, pager, next, jumper" :total="total">
        </el-pagination>

        <!-- Edit user information 用户信息编辑对话框 -->
        <el-dialog @close="clearForm" :title="title" :visible.sync="dialogFormVisible">
            <el-form :model="userForm" ref="userFormRef" :rules="rules">
                <el-form-item label="Username" prop="username" :label-width="formLabelWidth">
                    <el-input v-model="userForm.username" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="Password" prop="password" :label-width="formLabelWidth">
                    <el-input type="password" v-model="userForm.password" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="Phone" :label-width="formLabelWidth">
                    <el-input v-model="userForm.phone" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="Status" :label-width="formLabelWidth">
                    <el-switch v-model="userForm.status" :active-value="1" :inactive-value="0">
                    </el-switch>
                </el-form-item>
                <el-form-item label="Email" prop="email" :label-width="formLabelWidth">
                    <el-input v-model="userForm.email" autocomplete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisible = false">Cancel</el-button>
                <el-button type="primary" @click="saveUser">Confirm</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
import userApi from "@/api/userManage";
export default {
    data() {
        var checkEmail = (rule, value, callback) => {
            var reg =
                /^[a-zA-Z0-9]+([-_.][a-zA-Z0-9]+)*@[a-zA-Z0-9]+([-_.][a-zA-Z0-9]+)*\.[a-z]{2,}$/;
            if (!reg.test(value)) {
                return callback(new Error("Invalid email format"));
            }
            callback();
        };
        return {
            formLabelWidth: '130px',
            userForm: {},
            dialogFormVisible: false,
            title: "",
            total: 0,
            searchModel: {
                pageNo: 1,
                pageSize: 10
            },
            userList: [],
            rules: {
                username: [
                    { required: true, message: 'Please set up your username', trigger: 'blur' },
                    { min: 3, max: 50, message: 'Length should be 3 to 50 characters', trigger: 'blur' }
                ],
                password: [
                    { required: true, message: 'Please set up your password', trigger: 'blur' },
                    { min: 6, max: 16, message: 'Length should be 6 to 16 characters', trigger: 'blur' }
                ],
                email: [
                    { required: true, message: 'Please enter your Email', trigger: 'blur' },
                    { validator: checkEmail, trigger: "blur" },
                ],
            }
        };
    },
    methods: {
        saveUser() {
            // 触发表单验证
            this.$refs.userFormRef.validate((valid) => {
                if (valid) {
                    // 提交请求给后台
                    userApi.addUser(this.userForm).then(response => {
                        // 成功提示
                        this.$message({
                            title: 'Success',
                            message: response.message,
                            type: 'success'
                        });
                        // 关闭对话框
                        this.dialogFormVisible = false;
                        // 刷新表格
                        this.getUserList();
                    });
                } else {
                    console.log('error submit!!');
                    return false;
                }
            });

        },
        clearForm() {
            this.userForm = {};
            this.$refs.userFormRef.clearValidate();
        },
        openEditUI() {
            this.title = 'Add new user';
            this.dialogFormVisible = true;
        },
        handleSizeChange(pageSize) {
            this.searchModel.pageSize = pageSize;
            this.getUserList();
        },
        handleCurrentChange(pageNo) {
            this.searchModel.pageNo = pageNo;
            this.getUserList();
        },
        getUserList() {
            userApi.getUserList(this.searchModel).then((response) => {
                this.userList = response.data.rows;
                this.total = response.data.total;
            });
        },
    },
    created() {
        this.getUserList();
    },
};
</script>

<style>
#search .el-input {
    width: 200px;
    margin-right: 10px;
}

.el-dialog .el-input {
    width: 85%;
}
</style>