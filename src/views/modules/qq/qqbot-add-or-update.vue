<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="机器人号码" prop="qqId">
      <el-input v-model="dataForm.qqId" placeholder="机器人号码"></el-input>
    </el-form-item>
    <el-form-item label="机器人IP" prop="serverIp">
      <el-input v-model="dataForm.serverIp" placeholder="机器人IP"></el-input>
    </el-form-item>
    <el-form-item label="机器人端口" prop="serverPort">
      <el-input v-model="dataForm.serverPort" placeholder="机器人端口"></el-input>
    </el-form-item>
    <el-form-item label="认证Key" prop="authKey">
      <el-input v-model="dataForm.authKey" placeholder="认证Key"></el-input>
    </el-form-item>
    <el-form-item label="自动回复" prop="replyList">
      <el-checkbox-group v-model="dataForm.replyList">
        <el-checkbox :label="data" :indeterminate="(dataForm.replyList.filter(o=>o.replyId == data.replyId).length>0)" v-for="data in dataForm.allReplyList">{{data.replyName}}</el-checkbox>
      </el-checkbox-group>
    </el-form-item>
    <el-form-item label="是否启动" prop="enable">
      <el-switch v-model="dataForm.enable"></el-switch>
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
          id: 0,
          qqId: '',
          serverIp: '',
          serverPort: '',
          authKey: '',
          enable: true,
          allReplyList: [],
          replyList: []
        },
        dataRule: {
          qqId: [
            { required: true, message: '机器人号码不能为空', trigger: 'blur' }
          ],
          serverIp: [
            { required: true, message: '机器人IP不能为空', trigger: 'blur' }
          ],
          serverPort: [
            { required: true, message: '机器人端口不能为空', trigger: 'blur' }
          ],
          authKey: [
            { required: true, message: '认证Key不能为空', trigger: 'blur' }
          ],
          sessionKey: [
            { required: true, message: '会话Key不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/generator/qqbot/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.qqId = data.qqBot.qqId
                this.dataForm.serverIp = data.qqBot.serverIp
                this.dataForm.serverPort = data.qqBot.serverPort
                this.dataForm.authKey = data.qqBot.authKey
                this.dataForm.enable = data.qqBot.enable === 1
                this.dataForm.replyList = data.qqBot.replyList
                this.dataForm.allReplyList = data.replyList
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
              url: this.$http.adornUrl(`/generator/qqbot/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'qqId': this.dataForm.qqId,
                'serverIp': this.dataForm.serverIp,
                'serverPort': this.dataForm.serverPort,
                'authKey': this.dataForm.authKey,
                'enable': this.dataForm.enable === true ? 1 : 0,
                'replyList': this.dataForm.replyList
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
