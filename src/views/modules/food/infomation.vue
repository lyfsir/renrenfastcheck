<template>
  <div>

    <el-dialog
      v-if="status"
      :visible.sync="dialogVisible"
      top="10%"
      width="50%"
      :before-close="handleClose"
    >
      <el-table
        v-loading="loading">

      </el-table>
    </el-dialog>
    <el-dialog
      v-if="!status"
      :title="dataList.userName+'的食谱信息'"
      :visible.sync="dialogVisible"
      width="50%"
      :before-close="handleClose"
    >
      <div class="div">
        <div><p class="tname">所属专题：</p><p class="tvalue">{{dataList.specialName}}</p></div>
        <div>
          <p class="tname">所属专题：</p>
          <el-tag type="success" style="margin-left: 10px;margin-top: 10px" v-for="(cate, indexs) in dataList.cateName" :key="indexs">
            <p class="tvalue">{{cate}}</p></el-tag>
        </div>
        <div><p class="tname">标题：</p><p class="tvalue">{{dataList.title}}</p></div>
        <div><p class="tname">说明：</p><p class="tvalue">{{dataList.descrit}}</p></div>
        <div><p class="tname">成品图：</p></div>
        <span class="imglist" v-for="(item, index) in dataList.selfoodimages" :key="index">
        <img :src="item.imagesUrl" alt="img" style="width:100%;height:100%"/>
      </span>

        <div><p class="tname">配料信息：</p></div>
        <div
          class="peiliao"
          v-for="(item, index) in dataList.foodAttrs"
          :key="'_'+index"
        ><p class="tvalue">食材名：{{item.attrName}} = 用量：{{item.attrValue}}</p>
        </div>

        <div><p class="tname">步骤信息：</p></div>
        <div
          class="peiliao"
          v-for="(item, index) in dataList.selfoodSteps"
          :key="'__'+index"
        ><p class="tvalue">第{{index+1}}步 : {{item.content}}</p>
        </div>
      </div>

    </el-dialog>

  </div>
</template>

<script>
    export default {
        data() {
            return {
                status: true,
                loading: true,
                options: [{}],
                value: '',
                fid: 0,
                dataList: [],
                dialogVisible: false,

                cids: [],

                // cid: [],

                cname: [],

                category: [],

                categorys: [],
                props: {
                    value: "id",
                    label: "name",
                    children: "children"
                },

            };
        },
        created() {
            this.getDataList();
        },
        methods: {

            getDataList() {
                this.dataListLoading = true;
                this.$http({
                    url: this.$http.adornUrl("/food/sort/doget/sort"),
                    method: "get"
                }).then(({data}) => {
                    console.log("tree", data)
                    this.categorys = data.data.data;
                });
            },
            handleClose(done) {
                this.$confirm("确认关闭？")
                    .then(_ => {
                        done();
                        this.cname = []
                        this.categorys = []
                    })
                    .catch(_ => {
                    });
            },
            //根据过来的id进行查询
            init(id) {
                this.fid = id;
                this.dialogVisible = true;
                this.$http({
                    url: this.$http.adornUrl(
                        `/food/info/doget/info/${this.fid}`
                    ),
                    method: "get"
                })
                    .then(({data}) => {
                        console.log("info", data)
                        if (data && data.code === 0) {
                            this.dataList = data.data;
                            this.status = false
                        } else {
                            //请求失败处理
                        }
                    })
                    .catch(err => {
                        console.log(err);
                    });
            },

        }
    };
</script>

<style>
  .div div {
    padding: 10px;
  }
  .tname{
    display: inline;
    color: #67C23A;
    font-family: "宋体";
    font-size: 15px;
    font-weight: bold;
  }
  .tvalue{
    display: inline;
    color: #909399;
    font-family: "宋体";
    font-size: 14px;
  }

  .imglist {
    display: inline-block;
    width: 150px;
    height: 150px;
    margin-left: 10px;
  }

  .peiliao {
    margin-left: 50px;
  }
</style>
