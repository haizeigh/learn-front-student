<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item :to="{ path: '/student/camp' }">已选课程管理</el-breadcrumb-item>
      <el-breadcrumb-item :to="{ path: '/lesson' }">章节列表</el-breadcrumb-item>
      <el-breadcrumb-item>学习内容</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-steps  finish-status="success">
        <el-step title="步骤 1"></el-step>
        <el-step title="步骤 2"></el-step>
        <el-step title="步骤 3"></el-step>
        <el-step title="步骤 4"></el-step>
      </el-steps>
      <!-- <div > -->
        <!-- <span class="demonstration" >Click 指示器触发</span> -->
        <el-carousel trigger="click" arrow="always" :autoplay=false indicator-position="outside" :loop=false @change="carouselChange">
          <el-carousel-item v-for="step in stepList" :key="step.id" :name="step.id" >
            <!-- <h3 class="normal">{{ item }}</h3> -->
                  <!-- 搜索 添加 -->
            <el-row :gutter="20" v-if="step.stepType=='阅读题'">
              <el-col :span="6">
                <span>{{step.stepType}}</span>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="6">
                <span>{{step.stepTitle}}</span>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="6">
                <span>{{step.content}}</span>
              </el-col>
            </el-row>
          </el-carousel-item>
        </el-carousel>
      <!-- </div> -->
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    // 自定义邮箱规则
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^\w+@\w+(\.\w+)+$/
      if (regEmail.test(value)) {
        // 合法邮箱
        return callback()
      }
      callback(new Error('请输入合法邮箱'))
    }
    // 自定义手机号规则
    var checkMobile = (rule, value, callback) => {
      const regMobile = /^1[34578]\d{9}$/
      if (regMobile.test(value)) {
        return callback()
      }
      // 返回一个错误提示
      callback(new Error('请输入合法的手机号码'))
    }
    return {
      // 获取用户列表查询参数对象
      queryInfo: {
        query: '',
        // 当前页数
        pagenum: 1,
        // 每页显示多少数据
        pagesize: 5
      },
      stepList: [],
      totle: 0,
      // 添加用户对话框
      addDialogVisible: false,
      // 用户添加
      addUserForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 用户添加表单验证规则
      addUserFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 2,
            max: 10,
            message: '用户名的长度在2～10个字',
            trigger: 'blur'
          }
        ],
        password: [
          { required: true, message: '请输入用户密码', trigger: 'blur' },
          {
            min: 6,
            max: 18,
            message: '用户密码的长度在6～18个字',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 修改用户
      editDialogVisible: false,
      editUserForm: {},
      // 编辑用户表单验证
      editUserFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 分配角色对话框
      setRoleDialogVisible: false,
      // 当前需要被分配角色的用户
      userInfo: {},
      // 所有角色数据列表
      rolesLsit: [],
      // 已选中的角色Id值
      selectRoleId: ''
    }
  },
  created () {
    this.getStepList()
  },
  methods: {
    async getStepList () {
      const { data: res } = await this.$http.get('step/lesson/' + this.$route.query.lessonId, {
        params: { pageSize: this.queryInfo.pagesize,
          pageNum: this.queryInfo.pagenum,
          query: this.queryInfo.query }
      })
      console.log(res)
      if (res.code !== 200) {
        return this.$message.error('获取学习内容失败！')
      }
      this.stepList = res.data.content
      this.totle = res.data.totalCount
    },
    // 监听 pagesize改变的事件
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getStepList()
    },
    // 监听 页码值 改变事件
    handleCurrentChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagenum = newSize
      this.getStepList()
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
        this.getStepList()
      })
    },
    // 编辑用户信息
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败！')
      }
      this.editUserForm = res.data
      this.editDialogVisible = true
    },
    // 占展示可选的班级
    async showClassDialog (id) {
      const { data: res } = await this.$http.get('class/camp/' + id, {
        params: { pageSize: 1, pageNum: 20 }
      })
      if (res.code !== 200) {
        return this.$message.error('查询班级列表失败！')
      }
      this.editUserForm = res.data.content
      this.editDialogVisible = true
    },
    // 监听修改用户对话框的关闭事件
    editDialogClosed () {
      this.$refs.editUserFormRef.resetFields()
    },
    // 修改用户信息
    editUser () {
      // 提交请求前，表单预验证
      this.$refs.editUserFormRef.validate(async valid => {
        // console.log(valid)
        // 表单预校验失败
        if (!valid) return
        const { data: res } = await this.$http.put(
          'users/' + this.editUserForm.id,
          {
            email: this.editUserForm.email,
            mobile: this.editUserForm.mobile
          }
        )
        if (res.meta.status !== 200) {
          this.$message.error('更新用户信息失败！')
        }
        // 隐藏添加用户对话框
        this.editDialogVisible = false
        this.$message.success('更新用户信息成功！')
        this.getStepList()
      })
    },
    // 删除用户
    async removeUserById (id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 点击确定 返回值为：confirm
      // 点击取消 返回值为： cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除用户失败！')
      this.$message.success('删除用户成功！')
      this.getStepList()
    },

    // 删除用户所选的课程
    async removeStudentFromCampById (id) {
      const confirmResult = await this.$confirm(
        '确定退课吗?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 点击确定 返回值为：confirm
      // 点击取消 返回值为： cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消退课')
      }
      const { data: res } = await this.$http.delete('student/camp/' + id)
      if (res.code !== 200) return this.$message.error('退课失败！')
      this.$message.success('退课成功！')
      this.getStepList()
    },
    // 展示分配角色的对话框
    async showSetRole (role) {
      this.userInfo = role
      // 展示对话框之前，获取所有角色列表
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      this.rolesLsit = res.data
      this.setRoleDialogVisible = true
    },
    // 分配角色
    async saveRoleInfo () {
      if (!this.selectRoleId) {
        return this.$message.error('请选择要分配的角色')
      }
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, { rid: this.selectRoleId })
      if (res.meta.status !== 200) {
        return this.$message.error('更新用户角色失败！')
      }
      this.$message.success('更新角色成功！')
      this.getStepList()
      this.setRoleDialogVisible = false
    },
    // 分配角色对话框关闭事件
    setRoleDialogClosed () {
      this.selectRoleId = ''
      this.userInfo = {}
    },
    carouselChange () {
      console.log('22')
    }
  }
}
</script>

<style lang="less" scoped>
   .el-carousel__item h3 {
    color: #475669;
    font-size: 14px;
    opacity: 0.75;
    /* line-height: 150px; */
    margin: 0;
  }
  .el-carousel__item:nth-child(2n) {
     background-color: #6b84a7;
  }
  .el-carousel__item:nth-child(2n+1) {
     background-color: #9ab2ca;
  }
  .demonstration{
    padding-bottom: 20px;
  }
</style>
