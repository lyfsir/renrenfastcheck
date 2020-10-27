<template>
  <div>
    <el-row>
      <el-button type="danger" round @click="alldel">批量删除</el-button>
    </el-row>
    <el-tree
      :data="data"
      :props="defaultProps"
      @node-click="handleNodeClick"
      node-key="id"
      :expand-on-click-node="false"
      show-checkbox
      :default-expanded-keys="dek"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.data.level==1"
            type="text"
            size="mini"
            @click="() => append(data)"
          >Append</el-button>
          <el-button type="text" size="mini" @click="edit(data)">edit</el-button>
          <el-button
            v-if="node.data.children==null"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >Delete</el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="title" :visible.sync="dialogFormVisible">
      <el-form :model="category">
        <el-form-item label="菜单名称">
          <el-input v-model="name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submit">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  components: {},
  props: {},
  data() {
    return {
      data: [],
      dek: [],
      dialogFormVisible: false,
      category: {
        id: 0,
        name: "",
        parentId: 0,
        level: 0,
        showStatus: 1,
        sort: 0
      },
      name: "",
      title: "",
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  created() {
    this.getDataList();
  },
  methods: {
    edit(data) {
      this.category = {};
      this.dialogFormVisible = true;
      this.title = "修改菜单";
      this.name = data.name;

      this.category.id = data.id;
      this.category.parentId = data.parentId;
    },
    submit() {
      if (this.title === "新增菜单") {
        this.add();
      } else {
        this.subedit();
      }
    },
    append(data) {
      this.name = "";
      console.log("pi", data);
      this.title = "新增菜单";
      this.category = {};
      this.category.parentId = data.id;
      this.category.level = data.level * 1 + 1;
      console.log("data", data);
      this.dialogFormVisible = true;
    },

    remove(node, data) {
      var ids = [node.data.id];
      this.$confirm(`是否确认删除 ${data.name} 菜单`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/food/sort/del/ids"),
            method: "delete",
            data: this.$http.adornData(ids, false)
          })
            .then(({ data }) => {
              this.$message({
                message: "恭喜你，删除成功",
                type: "success"
              });
              this.getDataList();
              this.dek = [node.data.parentId];
              console.log("dek", this.dek);
            })
            .catch(() => {});
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });

      console.log("remove", node, data);
    },

    alldel(){
      let all = [];
      let allid = [];
      let names = [];
      let pid = []
      all = this.$refs.menuTree.getCheckedNodes();
      console.log(all);
      if(all.length===0){
        this.$message({
            type: "info",
            message: "请选择要删除的菜单"
          });
        return;
      }
      for(var i = 0;i<all.length;i++){
          allid.push(all[i].id);
          names.push(all[i].name);
          pid.push(all[i].parentId);
      }
      this.$confirm(`是否确认批量删除 ${names} 菜单`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/food/sort/del/ids"),
            method: "delete",
            data: this.$http.adornData(allid, false)
          })
            .then(({ data }) => {
              this.$message({
                message: "恭喜你，删除成功",
                type: "success"
              });
              this.getDataList();
              console.log(pid);
              this.dek = pid;
              console.log("dek", this.dek);
            })
            .catch(() => {});
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });

    },

    handleNodeClick(data) {
      console.log(data);
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/food/sort/doget/sort"),
        method: "get"
      }).then(({ data }) => {
          console.log("data",data)
        this.data = data.data.data;

      });
    },

    add() {
      this.category.name = this.name;
      var data = this.category;

      if (data.name.trim() == "") {
        this.$message({
          message: "菜单名不能为空",
          type: "warning"
        });
        return;
      }
      this.$http({
        url: this.$http.adornUrl("/food/sort/add/sort"),
        method: "post",
        data: this.$http.adornData(data, false)
      }).then(({ data }) => {
        this.getDataList();
        console.log("id", data);
        this.dek = [this.category.parentId];
        this.dialogFormVisible = false;
        this.category = {};
      });
    },
    subedit() {
      this.category.name = this.name;
      var data = this.category;
      if (data.name.trim() == "") {
        this.$message({
          message: "菜单名不能为空",
          type: "warning"
        });
        return;
      }
      this.$http({
        url: this.$http.adornUrl("/food/sort/edit/sort"),
        method: "put",
        data: this.$http.adornData(data, false)
      }).then(({ data }) => {
        this.getDataList();
        console.log("id", data);
        this.dek = [this.category.id];
        this.dialogFormVisible = false;
        this.category = {};
      });
    }
  }
};
</script>

<style>
</style>
