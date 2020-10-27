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
    :title="dataList.userName+'的食谱审核'"
    :visible.sync="dialogVisible"
    width="50%"
    :before-close="handleClose"
  >
    <div class="div">
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
    <span slot="footer" class="dialog-footer">
      <el-button @click="del">审核不通过</el-button>
      <el-button type="primary" @click="audsuccess">通过审核</el-button>
    </span>

    <!-- 通过级联选择器选择所属菜单，把选择的菜单名绑定到一个指定的文本中 -->
    <el-tag
      v-for="(item, index) in category"
      :key="item.categoryId"
      closable
      style="margin-top: 20px;margin-left: 10px"
      @close="handledel(item.cateName)"
      type="success">
      {{item.cateName}}
    </el-tag>

<!--    <el-tag type="success" v-for="(item, index) in category" :key=index style="margin-top: 20px;margin-left: 10px">{{item.cateName}}</el-tag>-->
<!--    <el-button type="danger" icon="el-icon-delete" circle @click="del" v-if="cname.length>0" style="margin-left: 10px"></el-button>-->
    <div style="margin-top: 20px">
      <div class="block">
        <span class="demonstration"><p class="tname">请选择分类(必须)</p></span>
        <el-cascader v-model="cids" :options="categorys" :props="props" @change="handleChange"
                     ref="cascaderAddr"></el-cascader>
      </div>
    </div>

    <div style="margin-top: 20px">
      <div class="block">
        <span class="demonstration"><p class="tname">请选择所属专题(可选)</p></span>
        <el-select v-model="value" placeholder="请选择" @change="handleChange2">
          <el-option
            v-for="item in options"
            :key="item.id"
            :label="item.name"
            :value="item.id">
          </el-option>
        </el-select>
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
                options: [{

                }],
                value: '',
                fid: 0,
                dataList: [],
                dialogVisible: false,

                cids: [],

                // cid: [],

                cname : [],

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
            this.getSpecila();
        },
        methods: {
            del(){
              this.cname.pop();
              this.category.pop();
            },
            getSpecila(){
                this.$http({
                    url: this.$http.adornUrl("/food/special/info"),
                    method: "get"
                }).then(({data}) => {
                    console.log("spl", data)
                    this.options = data.data;
                });
            },
            handleChange(value) {
                console.log(this.cid);
                this.category.push({
                    categoryId: this.cids[this.cids.length - 1],
                    cateName: this.$refs['cascaderAddr'].currentLabels[this.$refs['cascaderAddr'].currentLabels.length - 1]
                })
                // this.cid.push(this.cids[this.cids.length-1]);
                // this.cname.push(this.$refs['cascaderAddr'].currentLabels[this.$refs['cascaderAddr'].currentLabels.length-1])
                // console.log(this.$refs['cascaderAddr'].currentLabels)
                console.log("testdata", this.category)
                this.cname.push(this.$refs['cascaderAddr'].currentLabels[this.$refs['cascaderAddr'].currentLabels.length - 1])
            },
            handleChange2(value){
                this.dataList.sid = value;
            },
            handledel(tag) {
                this.category.splice(this.category.indexOf(tag), 1);
                console.log("test",this.category)
            },
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
                        this.category = []
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
            //保存审核通过信息
            audsuccess() {
                if (this.category.length == 0) {
                    this.$message.error("请选择分类信息");
                    return;
                }
                console.log("post", this.dataList);
                this.dataList.category = this.category;
                //需要id，name，cids
                const {id, title, category,sid} = this.dataList
                const data = {id, title, category,sid};
                console.log("post", data);
                this.$http({
                    url: this.$http.adornUrl("/food/info/save/food"),
                    method: "post",
                    data: data
                }).then(({data}) => {
                    if (data && data.code === 0) {
                        this.$message({
                            message: "操作成功",
                            type: "success"
                        });

                        //关闭窗口
                        this.dialogVisible = false;
                        //重新刷新数据
                        this.$emit("refreshDataList");
                        this.cname = []
                        this.category = []
                    } else {
                        this.$message.error(data.msg);
                    }
                }).catch(err => {
                    this.$message.error("请稍后再试");
                });
                //this.dialogVisible=false;
            },
            del(){
                var id = [this.dataList.id]
                console.log("id",id)
                this.$http({
                    url: this.$http.adornUrl('/food/info/delete/food'),
                    method: 'delete',
                    data: this.$http.adornData(id, false)
                }).then(({data})=>{
                    if (data && data.code === 0){
                        this.$message({
                            message: "操作成功",
                            type: "success"
                        });
                        this.dialogVisible = false
                        this.getSpecila();
                    }else {
                        this.$message.error(data.msg);
                    }
                }).catch(err => {
                    this.$message.error("请稍后再试");
                });
            }
        }
    };
</script>

<style>
  .div div {
    padding: 10px;
  }

  .imglist {
    display: inline-block;
    width: 150px;
    height: 150px;
    margin-left: 10px;
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

  .peiliao {
    margin-left: 50px;
  }
</style>
