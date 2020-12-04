<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="回复名称" prop="replyName">
      <el-input v-model="dataForm.replyName" placeholder="回复名称"></el-input>
    </el-form-item>
    <el-form-item label="回复对象">
        <el-checkbox v-model="dataForm.replyTargetPersonal" true-label=1 false-label=0>个人私聊回复</el-checkbox>
        <el-checkbox v-model="dataForm.replyTargetGroup" true-label=1 false-label=0>群消息回复</el-checkbox>
    </el-form-item>
    <el-form-item label="回复类型" prop="replyType">
      <el-radio v-model="dataForm.replyType" label="0">文本</el-radio>
      <el-radio v-model="dataForm.replyType" label="1">图文</el-radio>
      <el-radio v-model="dataForm.replyType" label="2">自定义接口</el-radio>
    </el-form-item>
    <el-form-item label="回复关键词" prop="replyKeys">
      <el-input v-model="dataForm.replyKeys" placeholder="回复关键词"></el-input>
    </el-form-item>
    <el-form-item label="关键词完整匹配" prop="replyMatch">
      <el-radio v-model="dataForm.replyMatch" label="0">否</el-radio>
      <el-radio v-model="dataForm.replyMatch" label="1">是</el-radio>
    </el-form-item>
    <el-form-item label="回复文本" prop="replyText" v-if="dataForm.replyType == 0 || dataForm.replyType == 1">
      <el-input type="textarea" v-model="dataForm.replyText" placeholder="回复文本"></el-input>
    </el-form-item>
      <el-upload
        v-if="dataForm.replyType == 1"
        class="upload-demo"
        :action="this.$http.adornUrl(`/sys/oss/upload?token=${this.$cookie.get('token')}`)"
        :file-list="fileList"
        :on-remove="handleRemove"
        :on-success="handleSuccess"
        list-type="picture">
        <el-button size="small" type="primary">点击上传</el-button>
        <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
      </el-upload>
    <el-form-item label="接口url" prop="replyApi" v-if="dataForm.replyType == 2">
      <el-input v-model="dataForm.replyApi" placeholder="自定义接口回复,收到消息转发到此API接口"></el-input>
    </el-form-item>
    <el-form-item label="json参数" prop="replyApiJson" v-if="dataForm.replyType == 2">
      <el-input v-model="dataForm.replyApiJson" placeholder="转发消息自定义json参数"></el-input>
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
        fileList: [],
        target: [],
        dataForm: {
          replyId: 0,
          replyName: '',
          replyTargetPersonal: 0,
          replyTargetGroup: 0,
          replyType: '0',
          replyKeys: '',
          replyMatch: '0',
          replyText: '',
          replyApi: '',
          replyApiJson: ''
        },
        dataRule: {
          replyType: [
            { required: true, message: '回复类型: 0文本, 1图文, 2自定义接口不能为空', trigger: 'blur' }
          ],
          replyName: [
            { required: true, message: '回复名称', trigger: 'blur' }
          ],
          replyKeys: [
            { required: true, message: '回复关键词不能为空', trigger: 'blur' }
          ],
          replyMatch: [
            { required: true, message: '关键词完整匹配: 0否, 1是不能为空', trigger: 'blur' }
          ],
          replyText: [
            { required: true, message: '回复文本不能为空', trigger: 'blur' }
          ],
          replyApi: [
            { required: true, message: '自定义接口回复,收到消息转发到此API接口不能为空', trigger: 'blur' }
          ],
          replyApiJson: [
            { required: true, message: '转发消息自定义json参数不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.replyId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          this.fileList = []
          if (this.dataForm.replyId) {
            this.$http({
              url: this.$http.adornUrl(`/generator/qqreply/info/${this.dataForm.replyId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.replyName = data.qqReply.replyName
                this.dataForm.replyType = data.qqReply.replyType + ''
                this.dataForm.replyTargetPersonal = data.qqReply.replyTargetPersonal + ''
                this.dataForm.replyTargetGroup = data.qqReply.replyTargetGroup + ''
                this.dataForm.replyKeys = data.qqReply.replyKeys
                this.dataForm.replyMatch = data.qqReply.replyMatch + ''
                this.dataForm.replyText = data.qqReply.replyText
                this.dataForm.replyApi = data.qqReply.replyApi
                this.dataForm.replyApiJson = data.qqReply.replyApiJson
                if (data.qqReply.images != null) {
                  for (var k in data.qqReply.images) {
                    this.fileList.push({name: data.qqReply.images[k].id, id: data.qqReply.images[k].id, url: data.qqReply.images[k].url})
                  }
                }
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
              url: this.$http.adornUrl(`/generator/qqreply/${!this.dataForm.replyId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'replyId': this.dataForm.replyId || undefined,
                'replyName': this.dataForm.replyName,
                'replyTargetPersonal': this.dataForm.replyTargetPersonal,
                'replyTargetGroup': this.dataForm.replyTargetGroup,
                'replyType': this.dataForm.replyType,
                'replyKeys': this.dataForm.replyKeys,
                'replyMatch': this.dataForm.replyMatch,
                'replyText': this.dataForm.replyText,
                'replyApi': this.dataForm.replyApi,
                'replyApiJson': this.dataForm.replyApiJson,
                'images': this.fileList.filter(o => { return {id: o.id} })
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
      },
      handleRemove (file, fileList) {
        this.fileList = this.fileList.filter(o => { return o.id !== file.id })
      },
      handleSuccess (response, file, fileList) {
        this.fileList.push({
          name: file.name,
          id: file.response.id,
          url: file.response.url
        })
      }
    }
  }
</script>
