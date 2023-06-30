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
                    <el-button type="primary" icon="el-icon-plus" circle></el-button>
                </el-col>
            </el-row>
        </el-card>

        <!-- Result List 结果列表 -->
        <el-card>
            <el-table :data="userList" stripe style="width: 100%">
                <el-table-column label="#" width="180">
                    <template slot-scope="scope">
                        <!-- (pageNo - 1) * pageSize + index + 1 -->
                        {{(searchModel.pageNo - 1) * searchModel.pageSize + scope.$index + 1}}
                    </template>
                </el-table-column>
                <el-table-column prop="id" label="User ID" width="180">
                </el-table-column>
                <el-table-column prop="username" label="Username" width="180">
                </el-table-column>
                <el-table-column prop="phone" label="Phone Number" width="180">
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
    </div>
</template>

<script>
import userApi from "@/api/userManage";
export default {
    data() {
        return {
            total: 0,
            searchModel: {
                pageNo: 1,
                pageSize: 10
            },
            userList: []
        }
    },
    methods: {
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
</style>