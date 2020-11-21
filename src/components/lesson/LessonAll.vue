<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <!-- <el-breadcrumb-item :to="{ path: '/student/camp' }">已选课程管理</el-breadcrumb-item> -->
      <el-breadcrumb-item>全部章节列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 搜索 添加 -->
      <el-row :gutter="20">
        <el-col :span="6">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getLessonList">
            <el-button slot="append" icon="el-icon-search" @click="getLessonList"></el-button>
          </el-input>
        </el-col>
        <!-- <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col> -->
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="campList" border stripe>
        <!-- stripe: 斑马条纹
        border：边框-->
        <el-table-column prop="scheduleDay" label="序号"></el-table-column>
        <el-table-column prop="lessonTitle" label="章节名称"></el-table-column>
        <el-table-column prop="description" label="章节描述"></el-table-column>
        <el-table-column prop="lastUpdatedTime" label="课程更新时间"></el-table-column>
        <el-table-column prop="" label="星星"></el-table-column>
        <!-- <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch>
          </template>
        </el-table-column> -->
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-tooltip
              class="item"
              effect="dark"
              content="开始学习"
              :enterable="false"
              placement="top"
            >
              <router-link :to="{path: '/step', query: {lessonId: scope.row.id}}">
                  <el-button
                  type="primary"
                  icon="el-icon-folder-opened"
                  size="small"
                  circle
                  ></el-button>
              </router-link>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[2, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="totle"
      ></el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      campId: this.$route.query.campId,
      // 获取用户列表查询参数对象
      queryInfo: {
        query: '',
        // 当前页数
        pagenum: 1,
        // 每页显示多少数据
        pagesize: 5
      },
      campList: [],
      totle: 0,
      // 添加用户对话框
      addDialogVisible: false,
      // 用户添加
      addUserForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      }
    }
  },
  created () {
    this.getLessonList()
  },
  methods: {
    async getLessonList () {
      const { data: res } = await this.$http.get('lesson/list', {
        params: { pageSize: this.queryInfo.pagesize,
          pageNum: this.queryInfo.pagenum,
          query: this.queryInfo.query }
      })
      // console.log(this.campId)
      if (res.code !== 200) {
        return this.$message.error('获取用户列表失败！')
      }
      this.campList = res.data.content
      this.totle = res.data.totalCount
    },
    // 监听 pagesize改变的事件
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getLessonList()
    },
    // 监听 页码值 改变事件
    handleCurrentChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagenum = newSize
      this.getLessonList()
    },
    // 监听 switch开关 状态改变
    async userStateChanged (userInfo) {
      // console.log(userInfo)
      const { data: res } = await this.$http.put(
        `users/${userInfo.id}/state/${userInfo.mg_state}`
      )
      if (res.meta.status !== 200) {
        userInfo.mg_state = !userInfo.mg_state
        return this.$message.error('更新用户状态失败')
      }
      this.$message.success('更新用户状态成功！')
    },
    // 监听 添加用户对话框的关闭事件
    addDialogClosed () {
      this.$refs.addUserFormRef.resetFields()
    },
    // 添加用户
    addUser () {
      // 提交请求前，表单预验证
      this.$refs.addUserFormRef.validate(async valid => {
        // console.log(valid)
        // 表单预校验失败
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addUserForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败！')
        }
        this.$message.success('添加用户成功！')
        // 隐藏添加用户对话框
        this.addDialogVisible = false
        this.getLessonList()
      })
    }
  }
}
</script>

<style lang="less" scoped>
</style>
