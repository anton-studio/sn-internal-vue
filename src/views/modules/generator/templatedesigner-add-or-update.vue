<template>
  <el-dialog
    :title="!dataForm.designerId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="设计师名字" prop="name" label-width="100px" >
      <el-input v-model="dataForm.name" placeholder="设计师名字"></el-input>
    </el-form-item>
    <el-form-item label="设计师类型" prop="type" label-width="100px">
      <el-select v-model="dataForm.type" placeholder="设计师类型">
        <el-option v-for="item in ['内部设计师', '外部设计师']" :key="item.index" :value="item" :label="item"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="质量得分" prop="ratings" label-width="100px">
      <el-rate style="margin-top: 8px;" show-text v-model="dataForm.ratings" :texts="['极差', '失望', '一般', '满意', '惊喜']"></el-rate>
    </el-form-item>
    <el-form-item label="联系方式" prop="contact" label-width="100px">
      <el-input v-model="dataForm.contact" placeholder="联系方式"></el-input>
    </el-form-item>
    <el-form-item label="备注" prop="notes" label-width="100px">
      <el-input v-model="dataForm.notes" placeholder="备注" type="textarea"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          designerId: 0,
          name: '',
          type: '',
          ratings: '',
          contact: '',
          notes: ''
        },
        dataRule: {
          name: [
            { required: true, message: '设计师名字不能为空', trigger: 'blur' }
          ],
          type: [
            { required: true, message: '设计师类型 (内部设计师/外部设计师)不能为空', trigger: 'blur' }
          ],
          ratings: [
            { required: false, message: '质量得分 (0-5)不能为空', trigger: 'blur' }
          ],
          contact: [
            { required: false, message: '联系方式不能为空', trigger: 'blur' }
          ],
          notes: [
            { required: false, message: '备注不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.designerId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.designerId) {
            this.$http({
              url: this.$http.adornUrl(`/generator/templatedesigner/info/${this.dataForm.designerId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.templateDesigner.name
                this.dataForm.type = data.templateDesigner.type
                this.dataForm.ratings = data.templateDesigner.ratings
                this.dataForm.contact = data.templateDesigner.contact
                this.dataForm.notes = data.templateDesigner.notes
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/templatedesigner/${!this.dataForm.designerId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'designerId': this.dataForm.designerId || undefined,
                'name': this.dataForm.name,
                'type': this.dataForm.type,
                'ratings': this.dataForm.ratings,
                'contact': this.dataForm.contact,
                'notes': this.dataForm.notes
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
