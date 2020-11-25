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
    <el-card >
      <el-steps finish-status="success">
        <el-step v-for="step in stepList" :title="step.stepType" :key="step.id"></el-step>
      </el-steps>
      <!-- <span class="demonstration" >Click 指示器触发</span> -->
      <el-carousel trigger="click" arrow="always" :autoplay=false :indicator-position="outside" :loop=false @change="carouselChange" height="400px">
        <el-carousel-item v-for="(step,index) in stepList" :key="index" >
          <!-- <div>
            <span>请完成: {{step.stepTitle}}</span>
          </div> -->
          <!-- <span >请完成：{{step.stepType}}</span> -->
          <!-- 视频观看题目 -->
          <video-player v-if="step.stepType ==='视频观看'"  class="video-player vjs-custom-skin" :playsinline="true" ref="videoPlayer" :options="step.contentObject"></video-player>
          <!-- 简答题目 -->
          <el-form
          ref="simpleAnswerFormRef"
          label-width="100px"
          v-if="step.stepType ==='简答'"
          :data="step.contentObject"
          >
            <el-form-item label="标题" >
              <span>{{step.contentObject.title}}</span>
            </el-form-item>
            <el-form-item label="回答" prop="answerFromStudent">
              <el-input v-model="step.contentObject.answerFromStudent" type="textarea" :autosize="{ minRows: 4, maxRows: 10}" style="width: 80%; height: 50%;">{{step.contentObject.answerFromStudent}}</el-input>
            </el-form-item>
            <el-form-item label="参考答案" >
              <span>{{step.contentObject.answerFromTeacher}}</span>
            </el-form-item>
            <el-form-item>
              <el-button type="primary">提交</el-button>
              <!-- <el-button >重置</el-button> -->
            </el-form-item>
          </el-form>
          <!-- 单选题目 -->
          <el-form
          label-width="100px"
          v-if="step.stepType ==='单选'"
          :data="step.contentObject"
          ref="simpleSelectFormRef"
          >
            <el-form-item label="标题" >
              <span>{{step.contentObject.title}}</span>
            </el-form-item>
            <el-form-item label="请选择">
              <el-radio-group v-model="step.contentObject.selected">
                <el-radio v-for="select in step.contentObject.selectList" :label="select" :key="select">{{select}}</el-radio>
                <!-- <el-radio label="2">不会</el-radio> -->
              </el-radio-group>
            </el-form-item>
            <el-form-item label="参考答案" >
              <span>{{step.contentObject.answerFromTeacher}}</span>
            </el-form-item>
            <el-form-item>
              <el-button type="primary">提交</el-button>
              <!-- <el-button >重置</el-button> -->
            </el-form-item>
        </el-form>
        <!-- 阅读题 -->
        </el-carousel-item>
      </el-carousel>
      <!-- <a href="../../assets/pdf/机器学习.pdf"></a> -->
      <!-- <el-link href="../../assets/pdf/机器学习.pdf" >22</el-link> -->
      <!-- <el-link href="https://element.eleme.io" target="_blank">默认链接</el-link> -->
      <!-- <pdf src="../../assets/pdf/机器学习.pdf"></pdf> -->
      <!-- <iframe src="../../assets/pdf/机器学习.pdf" id='iframebox' ref='pdfiframe' frameborder="0" scrolling="no"></iframe> -->
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
      selectRoleId: '',
      playerOptions: {
        // playbackRates: [0.7, 1.0, 1.5, 2.0], // 播放速度
        autoplay: false, // 如果true,浏览器准备好时开始回放。
        muted: false, // 默认情况下将会消除任何音频。
        loop: false, // 导致视频一结束就重新开始。
        preload: 'auto', // 建议浏览器在<video>加载元素后是否应该开始下载视频数据。auto浏览器选择最佳行为,立即开始加载视频（如果浏览器支持）
        language: 'zh-CN',
        aspectRatio: '16:9', // 将播放器置于流畅模式，并在计算播放器的动态大小时使用该值。值应该代表一个比例 - 用冒号分隔的两个数字（例如"16:9"或"4:3"）
        fluid: true, // 当true时，Video.js player将拥有流体大小。换句话说，它将按比例缩放以适应其容器。
        sources: [{
          type: 'video/mp4', // 这里的种类支持很多种：基本视频格式、直播、流媒体等，具体可以参看git网址项目
          src: 'https://api.dogecloud.com/player/get.mp4?vcode=5ac682e6f8231991&userId=17&ext=.mp4' // url地址
        }],
        // poster: 'https://i.loli.net/2019/06/06/5cf8c5d9c57b510947.png', // 你的封面地址
        // width: document.documentElement.clientWidth, // 播放器宽度
        notSupportedMessage: '此视频暂无法播放，请稍后再试'// 允许覆盖Video.js无法播放媒体源时显示的默认信息。
      },
      simpleAnswerForm: {
        title: '下雨吗',
        answerFromStudent: '',
        answerFromTeacher: '不会的'
      },
      simpleSelectForm: {
        title: '下雨吗',
        selected: '',
        selectList: ['会', '不知道', '不会'],
        answerFromTeacher: '不会'
      }
    }
  },
  created () {
    this.getStepList()
  },
  methods: {
    async getStepList () {
      const { data: res } = await this.$http.get('step/lesson/' + this.$route.query.lessonId, {
        params: { pageSize: this.queryInfo.pagesize,
          pageNum: 1,
          query: 100 }
      })
      if (res.code !== 200) {
        return this.$message.error('获取学习内容失败！')
      }
      // console.log(res)
      this.stepList = res.data.content
      this.totle = res.data.totalCount

      this.stepList.filter((item, i) => {
        // console.log(tem.content)
        // console.log(i)
        // this.carouselData[i] = JSON.parse(item.content)
        this.carouselData[i] = JSON.parse(item.content)
      })
      console.log(JSON.stringify(this.playerOptions))
      console.log(this.playerOptions)
      console.log(this.stepList[0].contentObject)
    },
    // 监听 pagesize改变的事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getStepList()
    },
    // 监听 页码值 改变事件
    handleCurrentChange (newSize) {
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
    carouselChange () {
    }
  }
}
</script>

<style lang="less" scoped>
   .el-carousel__item h3 {
    color: #475669;
    font-size: 14px;
    opacity: 0.75;
    line-height: 250px;
    margin: 0;
  }
  .el-carousel__item:nth-child(2n) {
     background-color: #bec0c2;
  }
  .el-carousel__item:nth-child(2n+1) {
     background-color: #c3d7ec;
  }
  .demonstration{
    padding-bottom: 20px;
  }
  .video-js .vjs-icon-placeholder {
    width: 80%;
    height: 80%;
    display: block;
  }
  .video-player {
    width: 60%;
    margin-left:auto;
    margin-right:auto;
    margin-top: auto;
    margin-bottom: auto;
    margin-bottom: 20px;
  }
</style>
