<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="目标号码" prop="target">
      <el-input v-model="dataForm.target" placeholder="目标号码"></el-input>
    </el-form-item>
    <el-form-item label="消息类型： 1好友 2群" prop="type">
      <el-input v-model="dataForm.type" placeholder="消息类型： 1好友 2群"></el-input>
    </el-form-item>
    <el-form-item label="消息内容" prop="content">
      <el-input v-model="dataForm.content" placeholder="消息内容"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">关闭</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          target: '',
          type: '',
          content: ''
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
              url: this.$http.adornUrl(`/generator/qqmsglog/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.target = data.qqMsgLog.target
                this.dataForm.type = data.qqMsgLog.type
                this.dataForm.content = data.qqMsgLog.content
              }
            })
          }
        })
      }
    }
  }
</script>
