<template>
  <el-dialog
    :title="!dataForm.industryId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="行业名" prop="name">
      <el-input v-model="dataForm.name" placeholder="行业名"></el-input>
    </el-form-item>
    <el-form-item label="备注" prop="notes">
      <el-input type="textarea" v-model="dataForm.notes" placeholder="备注"></el-input>
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
          industryId: 0,
          name: '',
          notes: ''
        },
        dataRule: {
          name: [
            { required: true, message: '行业名不能为空', trigger: 'blur' }
          ],
          notes: [
            { required: false, message: '备注不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.industryId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.industryId) {
            this.$http({
              url: this.$http.adornUrl(`/generator/templateindustry/info/${this.dataForm.industryId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.templateIndustry.name
                this.dataForm.notes = data.templateIndustry.notes
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
              url: this.$http.adornUrl(`/generator/templateindustry/${!this.dataForm.industryId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'industryId': this.dataForm.industryId || undefined,
                'name': this.dataForm.name,
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
