<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="用户名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="注册时间">
      </el-table-column>
      <el-table-column
        prop="username"
        header-align="center"
        align="center"
        label="用户名">
      </el-table-column>
      <!-- <el-table-column
        prop="password"
        header-align="center"
        align="center"
        label="密码">
      </el-table-column>
      <el-table-column
        prop="nickname"
        header-align="center"
        align="center"
        label="昵称">
      </el-table-column> -->
      <el-table-column
        prop="mobile"
        header-align="center"
        align="center"
        label="手机号码">
      </el-table-column>
      <!-- <el-table-column
        prop="email"
        header-align="center"
        align="center"
        label="邮箱">
      </el-table-column>
      <el-table-column
        prop="header"
        header-align="center"
        align="center"
        label="头像">
      </el-table-column> -->
      <el-table-column
        prop="gender"
        header-align="center"
        align="center"
        label="性别">
        <template slot-scope="scope">
          <p v-if="scope.row.gender === 1">
            男
          </p>
          <p v-if="scope.row.gender === 0">
            女
          </p>
        </template>
      </el-table-column>
      <!-- <el-table-column
        prop="birth"
        header-align="center"
        align="center"
        label="生日">
      </el-table-column>
      <el-table-column
        prop="city"
        header-align="center"
        align="center"
        label="所在城市">
      </el-table-column>
      <el-table-column
        prop="job"
        header-align="center"
        align="center"
        label="职业">
      </el-table-column>
      <el-table-column
        prop="sign"
        header-align="center"
        align="center"
        label="个性签名">
      </el-table-column>
      <el-table-column
        prop="sourceType"
        header-align="center"
        align="center"
        label="用户来源">
      </el-table-column> -->
      <!-- <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="此用户是否有效">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="注册时间">
      </el-table-column> -->
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
  </div>
</template>

<script>
    export default {
        data() {
            return {
                dataForm: {
                    key: ''
                },
                dataList: [],
                pageIndex: 1,
                pageSize: 10,
                totalPage: 0,
                dataListLoading: false,
                dataListSelections: [],
                addOrUpdateVisible: false
            }
        },

        activated() {
            this.getDataList()
        },
        methods: {
            // 获取数据列表
            getDataList() {
                this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/ums/ums/list'),
                    method: 'get',
                    params: this.$http.adornParams({
                        'page': this.pageIndex,
                        'limit': this.pageSize,
                        'username': this.dataForm.key
                    })
                }).then(({data}) => {
                    if (data && data.code === 0) {
                        this.dataList = data.page.list
                        this.totalPage = data.page.totalCount
                    } else {
                        this.dataList = []
                        this.totalPage = 0
                    }
                    this.dataListLoading = false
                })
            },
            // 每页数
            sizeChangeHandle(val) {
                this.pageSize = val
                this.pageIndex = 1
                this.getDataList()
            },
            // 当前页
            currentChangeHandle(val) {
                this.pageIndex = val
                this.getDataList()
            },
            // 多选
            selectionChangeHandle(val) {
                this.dataListSelections = val
            },
            // 新增 / 修改
            addOrUpdateHandle(id) {
                this.addOrUpdateVisible = true
                this.$nextTick(() => {
                    this.$refs.addOrUpdate.init(id)
                })
            },
            // 删除
            deleteHandle(id) {
                var ids = id ? [id] : this.dataListSelections.map(item => {
                    return item.id
                })
                this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.$http({
                        url: this.$http.adornUrl('/ums/ums/del'),
                        method: 'post',
                        data: this.$http.adornData(ids, false)
                    }).then(({data}) => {
                        if (data && data.code === 0) {
                            this.$message({
                                message: '操作成功',
                                type: 'success',
                                duration: 1500,
                                onClose: () => {
                                    this.getDataList()
                                }
                            })
                        } else {
                            this.$message.error(data.msg)
                        }
                    })
                })
            }
        }
    }
</script>
