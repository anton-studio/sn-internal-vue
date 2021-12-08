<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="模版名称" prop="name" label-width="120px">
      <el-input v-model="dataForm.name" placeholder="模版名称"></el-input>
    </el-form-item>
    <el-form-item label="模版行业" prop="industryId" label-width="120px">
      <el-select v-model="dataForm.industryId" placeholder="模版行业">
        <el-option v-for="item in industryList" :key="item.industryId" :value="item.industryId" :label="item.name"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="设计类型" prop="type" label-width="120px">
      <el-select v-model="dataForm.type" placeholder="设计类型">
        <el-option v-for="item in ['新模版设计', '旧模版复现']" :key="item.index" :value="item" :label="item"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="责任设计师" prop="designerId" label-width="120px">
      <el-select v-model="dataForm.designerId" placeholder="责任设计师">
        <el-option v-for="item in designerList" :key="item.industryId" :value="item.designerId" :label="item.name"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="模版登录地址" prop="loginDetail" label-width="120px">
      <el-input type="textarea" v-model="dataForm.loginDetail" placeholder="模版登录地址"></el-input>
    </el-form-item>
    <el-form-item label="开始日期" prop="startedDate" label-width="120px">
      <el-date-picker type="date"  value-format="timestamp" v-model="dataForm.startedDate" placeholder="开始日期"></el-date-picker>
    </el-form-item>
    <el-form-item label="完成日期" prop="finishedDate" label-width="120px">
      <el-date-picker type="date"  value-format="timestamp" v-model="dataForm.finishedDate" placeholder="完成日期"></el-date-picker>
    </el-form-item>
    <el-form-item label="状态更新" prop="status" label-width="120px">
      <el-radio-group v-model="dataForm.status">
        <el-radio-button label="计划中"></el-radio-button>
        <el-radio-button label="设计中"></el-radio-button>
        <el-radio-button label="已完成"></el-radio-button>
      </el-radio-group>
    </el-form-item>
    <el-form-item label="是否能上 Prod" prop="prodReady" label-width="120px">
      <el-switch
        :active-value="1"
        :inactive-value="0"
        v-model="dataForm.prodReady"
        active-text="可以"
        inactive-text="不可以">
      </el-switch>
    </el-form-item>
    <el-form-item label="模版上线链接" prop="prodLink" label-width="120px">
      <el-input v-model="dataForm.prodLink" placeholder="模版上线链接"></el-input>
    </el-form-item>
    <el-form-item label="备注" prop="notes" label-width="120px">
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
    props: ['designerList', 'industryList'],
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          name: '',
          industryId: '',
          type: '',
          loginDetail: '',
          designerId: '',
          startedDate: '',
          finishedDate: '',
          status: '计划中',
          prodReady: 0,
          prodLink: '',
          notes: ''
        },
        dataRule: {
          name: [
            { required: true, message: '模版名称不能为空', trigger: 'blur' }
          ],
          industryId: [
            { required: true, message: '模版行业不能为空', trigger: 'blur' }
          ],
          type: [
            { required: true, message: '设计类型 (新模版设计/旧模版复现)不能为空', trigger: 'blur' }
          ],
          loginDetail: [
            { required: true, message: '模版登录地址不能为空', trigger: 'blur' }
          ],
          designerId: [
            { required: true, message: '责任设计师不能为空', trigger: 'blur' }
          ],
          startedDate: [
            { required: false, message: '开始日期不能为空', trigger: 'blur' }
          ],
          finishedDate: [
            { required: false, message: '完成日期不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '状态更新不能为空', trigger: 'blur' }
          ],
          prodReady: [
            { required: true, message: '是否能上 Production (0,1)不能为空', trigger: 'blur' }
          ],
          prodLink: [
            { required: false, message: '模版上线链接不能为空', trigger: 'blur' }
          ],
          notes: [
            { required: false, message: '备注不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/generator/templates/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.templates.name
                this.dataForm.industryId = data.templates.industryId
                this.dataForm.type = data.templates.type
                this.dataForm.loginDetail = data.templates.loginDetail
                this.dataForm.designerId = data.templates.designerId
                this.dataForm.startedDate = new Date(data.templates.startedDate).getTime()
                this.dataForm.finishedDate = new Date(data.templates.finishedDate).getTime()
                this.dataForm.status = data.templates.status
                this.dataForm.prodReady = data.templates.prodReady
                this.dataForm.prodLink = data.templates.prodLink
                this.dataForm.notes = data.templates.notes
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
              url: this.$http.adornUrl(`/generator/templates/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'industryId': this.dataForm.industryId,
                'type': this.dataForm.type,
                'loginDetail': this.dataForm.loginDetail,
                'designerId': this.dataForm.designerId,
                'startedDate': this.dataForm.startedDate,
                'finishedDate': this.dataForm.finishedDate,
                'status': this.dataForm.status,
                'prodReady': this.dataForm.prodReady,
                'prodLink': this.dataForm.prodLink,
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
