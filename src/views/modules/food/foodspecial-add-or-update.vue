<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="80px">
      <el-form-item label="专题名字" prop="name">
        <el-input maxlength="50" show-word-limit v-model="dataForm.name" placeholder="专题名字"></el-input>
      </el-form-item>
      <el-form-item label="专题图片" prop="imgurl">
<!--        <el-input v-model="dataForm.imgurl" placeholder="专题图片"></el-input>-->
        <SingleUpload v-model="dataForm.imgurl"></SingleUpload>
      </el-form-item>
      <el-form-item label="详情图片" prop="imgurlBig">
<!--        <el-input v-model="dataForm.imgurlBig" placeholder="专题详情图"></el-input>-->
        <SingleUpload v-model="dataForm.imgurlBig"></SingleUpload>
      </el-form-item>
      <el-form-item label="详情内容" prop="content">
        <el-input type="textarea" :rows="5" maxlength="300" show-word-limit v-model="dataForm.content" placeholder="专题介绍"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
    import SingleUpload from '@/components/upload/singleUpload.vue'

    export default {

        data() {
            return {
                visible: false,
                dataForm: {
                    id: 0,
                    name: '',
                    imgurl: '',
                    imgurlBig: '',
                    content: ''
                },
                dataRule: {
                    name: [
                        {required: true, message: '专题名字不能为空', trigger: 'blur'}
                    ],
                    imgurl: [
                        {required: true, message: '专题图片不能为空', trigger: 'blur'}
                    ],
                    imgurlBig: [
                        {required: true, message: '专题详情图不能为空', trigger: 'blur'}
                    ],
                    content: [
                        {required: true, message: '专题介绍不能为空', trigger: 'blur'}
                    ],
                }
            }
        },
        components: {
            SingleUpload
        },
        methods: {
            init(id) {
                this.dataForm.id = id || 0
                this.visible = true
                this.$nextTick(() => {
                    this.$refs['dataForm'].resetFields()
                    if (this.dataForm.id) {
                        this.$http({
                            url: this.$http.adornUrl(`/food/special/info/${this.dataForm.id}`),
                            method: 'get',
                            params: this.$http.adornParams()
                        }).then(({data}) => {
                            console.log("info",data)
                            if (data && data.code === 0) {
                                this.dataForm.name = data.foodSpecial.name
                                this.dataForm.imgurl = data.foodSpecial.imgurl
                                this.dataForm.imgurlBig = data.foodSpecial.imgurlBig
                                this.dataForm.content = data.foodSpecial.content
                            }
                        })
                    }
                })
            },
            // 表单提交
            dataFormSubmit() {
                this.$refs['dataForm'].validate((valid) => {
                    if (valid) {
                        this.$http({
                            url: this.$http.adornUrl(`/food/special/${!this.dataForm.id ? 'add' : 'update'}`),
                            method: 'post',
                            data: this.$http.adornData({
                                'id': this.dataForm.id || undefined,
                                'name': this.dataForm.name,
                                'imgurl': this.dataForm.imgurl,
                                'imgurlBig': this.dataForm.imgurlBig,
                                'content': this.dataForm.content
                            })
                        }).then(({data}) => {
                            if (data && data.code === 0) {
                                this.$message({
                                    message: '操作成功',
                                    type: 'success',
                                    duration: 1500,
                                    onClose: () => {
                                        this.visible = false
                                        this.$emit('refreshDataList')
                                    }
                                })
                            } else {
                                this.$message.error(data.msg)
                            }
                        })
                    }
                })
            }
        }
    }
</script>
