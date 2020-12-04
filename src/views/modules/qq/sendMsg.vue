<template>
  <div v-loading="loading">
    <el-row :gutter="20">
      <el-col :span="12"><div class="grid-content bg-purple">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>发送内容</span>
          </div>
          <el-input
            type="textarea"
            :rows="2"
            placeholder="请输入发送内容"
            v-model="textarea">
          </el-input>
          <br/><br/>
          <el-upload
            class="upload-demo"
            :action="this.$http.adornUrl(`/sys/oss/upload?token=${this.$cookie.get('token')}`)"
            :on-remove="handleRemove"
            :on-success="handleSuccess"
            :file-list="fileList"
            list-type="picture">
            <el-button size="small" type="primary">点击上传</el-button>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
          </el-upload>
        </el-card>
      </div></el-col>

      <el-col :span="12"><div class="grid-content bg-purple">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>选择群发对象</span>
          </div>
          <el-select v-model="qqBot" @change="getFriendsAndGroups" placeholder="请选择QQ机器人">
            <el-option
              v-for="item in qqBots"
              :key="item.qqId"
              :label="item.qqId"
              :value="item.qqId"
              :disabled="item.enable != 1">
            </el-option>
          </el-select>
          <br /><br />
          <el-table
            ref="multipleTable"
            :data="list"
            tooltip-effect="dark"
            height="500px"
            border
            style="width: 100%"
            @selection-change="handleSelectionChange">
            <el-table-column
              type="selection"
              width="55">
            </el-table-column>
            <el-table-column
              prop="number"
              label="号码"
              show-overflow-tooltip>
            </el-table-column>
            <el-table-column
              prop="name"
              label="名称"
              show-overflow-tooltip>
            </el-table-column>
          </el-table>
        </el-card>
      </div></el-col>
    </el-row>
    <el-row>
      <el-col :span="24"><div class="grid-content bg-purple-dark"><el-button type="primary" @click="sendMsg">发送</el-button></div></el-col>
    </el-row>
  </div>
</template>
<script>
export default {
  data () {
    return {
      loading: false,
      textarea: '',
      fileList: [],
      qqBot: '',
      qqBots: [],
      friends: [],
      groups: [],
      list: [],
      multipleSelection: []
    }
  },
  activated () {
    this.getBotList()
  },
  methods: {
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
    getFriendsAndGroups (qqId) {
      this.$http({
        url: this.$http.adornUrl('/generator/qqbot/allGroupsAndFriends'),
        method: 'get',
        params: this.$http.adornParams({
          qqId: qqId
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.groups = []
          this.friends = []
          // eslint-disable-next-line no-unused-vars
          for (var k in data.groups) {
            this.groups.push({id: data.groups[k].id, number: '【群】' + data.groups[k].id, name: data.groups[k].name, type: 0})
          }
          // eslint-disable-next-line no-redeclare
          for (var k in data.friends) {
            this.friends.push({id: data.friends[k].id, number: data.friends[k].id, name: data.friends[k].nickname, type: 1})
          }
          this.list = [...this.groups, ...this.friends]
        } else {
          this.groups = []
          this.friends = []
          this.list = []
        }
      })
    },
    sendMsg () {
      let images = []
      // eslint-disable-next-line no-unused-vars
      for (var k in this.fileList) {
        images.push(this.fileList[k].id)
      }
      let m = {
        qqId: this.qqBot,
        friends: this.multipleSelection.filter(o => o.type === 1),
        groups: this.multipleSelection.filter(o => o.type === 0),
        content: this.textarea,
        images: images
      }
      if (m.qqId.length > 0 && (m.friends.length > 0 || m.groups.length > 0)) {
        this.loading = true
        this.$http({
          url: this.$http.adornUrl(`/generator/qqbot/sendMsg`),
          method: 'post',
          data: this.$http.adornData(m)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '发送成功',
              type: 'success'
            })
          } else {
            this.$message.error(data.msg)
          }
          this.loading = false
        })
      } else {
        this.$message({
          message: '请先选择发送对象',
          type: 'warning'
        })
      }
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
    },
    handleSelectionChange (val) {
      this.multipleSelection = val
    }
  }
}
</script>
<style>
</style>
