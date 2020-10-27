<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="食谱标题" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
        <el-button type="success" @click="tonbu()">同步食谱数据</el-button>
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
        label="创建时间">
      </el-table-column>
      <el-table-column
        prop="title"
        header-align="center"
        align="center"
        label="标题">
      </el-table-column>
      <el-table-column
        prop="images"
        header-align="center"
        align="center"
        label="默认图片">
        <template slot-scope="scope">
          <img :src="scope.row.images" alt="加载失败" width="150px" height="100px">
        </template>
      </el-table-column>
      <el-table-column
        prop="userName"
        header-align="center"
        align="center"
        label="来自用户">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">查看</el-button>
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
    <!-- 弹窗, 查看详情 -->
    <Infomation v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></Infomation>
  </div>
</template>

<script>
    import Infomation from './infomation'
    export default {
        data () {
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
        components:{

            Infomation
        },

        activated () {
            this.getDataList()
        },
        methods: {
            // 同步食谱信息到elasticsearch的foods索引
            tonbu() {
                this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/food/info/tongbu/foods'),
                    method: 'post',
                }).then(({data})=>{

                    if (data && data.code === 0) {
                        this.$alert(`一共${data.num}条数据`, '同步食谱信息成功', {
                            confirmButtonText: '确定',
                            callback: action => {
                                this.getDataList()
                            }
                        });
                    }
                    this.dataListLoading = false

                })
            },

            // 获取数据列表
            getDataList () {
                this.dataListLoading = true
                this.$http({
                    url: this.$http.adornUrl('/ums/auditi/ums/0'),
                    method: 'get',
                    params: this.$http.adornParams({
                        'page': this.pageIndex,
                        'limit': this.pageSize,
                        'title': this.dataForm.key
                    })
                }).then(({data}) => {
                    console.log(data)
                    if (data && data.code === 0) {
                        console.log("fdata",data)
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
            sizeChangeHandle (val) {
                this.pageSize = val
                this.pageIndex = 1
                this.getDataList()
            },
            // 当前页
            currentChangeHandle (val) {
                this.pageIndex = val
                this.getDataList()
            },
            // 多选
            selectionChangeHandle (val) {
                this.dataListSelections = val
            },
            // 新增 / 修改
            addOrUpdateHandle (id) {
                this.addOrUpdateVisible = true
                this.$nextTick(() => {
                    this.$refs.addOrUpdate.init(id)
                })
            },
            // 删除
            deleteHandle (id) {
                var ids = id ? [id] : this.dataListSelections.map(item => {
                    return item.id
                })
                this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.$http({
                        url: this.$http.adornUrl('/food/info/del/food'),
                        method: 'delete',
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
