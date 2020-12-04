<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="计划名称" prop="name">
      <el-input v-model="dataForm.name" placeholder="计划名称"></el-input>
    </el-form-item>
    <el-form-item label="消息类型" prop="type">
      <el-radio v-model="dataForm.type" label="0">文本</el-radio>
      <el-radio v-model="dataForm.type" label="1">图文</el-radio>
      <el-radio v-model="dataForm.type" label="2">自定义接口</el-radio>
    </el-form-item>
    <el-form-item label="请选择机器人" prop="botId">
      <el-select v-model="dataForm.botId" placeholder="请选择QQ机器人">
        <el-option
          v-for="item in qqBots"
          :key="item.id"
          :label="item.qqId"
          :value="item.id"
          :disabled="item.enable != 1">
        </el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="发送消息对象" prop="target">
      <el-input v-model="dataForm.target" placeholder="发送消息对象"></el-input>
    </el-form-item>
    <el-form-item label="消息文本" prop="text"  v-if="dataForm.type == 0 || dataForm.type == 1">
      <el-input type="textarea" v-model="dataForm.text" placeholder="消息文本"></el-input>
    </el-form-item>
    <el-form-item label="自定义消息接口" prop="api" v-if="dataForm.type == 2">
      <el-input v-model="dataForm.api" placeholder="自定义消息接口"></el-input>
    </el-form-item>
    <el-form-item label="自定义消息接口参数" prop="json" v-if="dataForm.type == 2">
      <el-input v-model="dataForm.json" placeholder="自定义消息接口参数"></el-input>
    </el-form-item>
    <el-form-item label="cron表达式" prop="cron">
      <el-input v-model="dataForm.cron" placeholder="cron表达式"></el-input>
    </el-form-item>
    </el-form>
    <el-upload
      v-if="dataForm.type == 1"
      class="upload-demo"
      :action="this.$http.adornUrl(`/sys/oss/upload?token=${this.$cookie.get('token')}`)"
      :file-list="fileList"
      :on-remove="handleRemove"
      :on-success="handleSuccess"
      list-type="picture">
      <el-button size="small" type="primary">点击上传</el-button>
      <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
    </el-upload>
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
        qqBots: [],
        dataForm: {
          id: 0,
          name: '',
          type: '0',
          botId: '',
          target: '',
          text: '',
          api: '',
          json: '',
          cron: ''
        },
        dataRule: {
          name: [
            { required: true, message: '计划名称不能为空', trigger: 'blur' }
          ],
          type: [
            { required: true, message: '消息类型不能为空', trigger: 'blur' }
          ],
          target: [
            { required: true, message: '发送消息对象不能为空', trigger: 'blur' }
          ],
          text: [
            { required: true, message: '消息文本不能为空', trigger: 'blur' }
          ],
          api: [
            { required: true, message: '自定义消息接口不能为空', trigger: 'blur' }
          ],
          json: [
            { required: true, message: '自定义消息接口参数不能为空', trigger: 'blur' }
          ],
          cron: [
            { required: true, message: 'cron表达式不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.fileList = []
          this.$refs['dataForm'].resetFields()
          this.getBotList()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/generator/qqschedulemsg/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.qqScheduleMsg.name
                this.dataForm.type = data.qqScheduleMsg.type + ''
                this.dataForm.botId = data.qqScheduleMsg.botId
                this.dataForm.target = data.qqScheduleMsg.target
                this.dataForm.text = data.qqScheduleMsg.text
                this.dataForm.api = data.qqScheduleMsg.api
                this.dataForm.json = data.qqScheduleMsg.json
                this.dataForm.cron = data.qqScheduleMsg.cron
                if (data.qqScheduleMsg.images != null) {
                  for (var k in data.qqScheduleMsg.images) {
                    this.fileList.push({name: data.qqScheduleMsg.images[k].id, id: data.qqScheduleMsg.images[k].id, url: data.qqScheduleMsg.images[k].url})
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
              url: this.$http.adornUrl(`/generator/qqschedulemsg/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'type': this.dataForm.type,
                'botId': this.dataForm.botId,
                'target': this.dataForm.target,
                'text': this.dataForm.text,
                'api': this.dataForm.api,
                'json': this.dataForm.json,
                'cron': this.dataForm.cron,
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
      getBotList () {
        this.$http({
          url: this.$http.adornUrl('/generator/qqbot/all'),
          method: 'get',
          params: this.$http.adornParams({
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.qqBots = data.list
          } else {
            this.qqBots = []
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
