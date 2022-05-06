<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片区域 -->
    <el-card class="box-card">
        <!-- 搜索栏 -->
        <el-row :gutter="20">
          <el-col :span="8">
            <div class="grid-content bg-purple">
              <!-- 搜索区域 -->
              <el-input placeholder="请输入内容" class="input-with-select" v-model="queryInfo.query" :clearable="true" @clear="getUsersInfo">
                <el-button slot="append" icon="el-icon-search" @click="getUsersInfoA"></el-button>
              </el-input>
            </div>
          </el-col>
          <!-- 添加用户按钮 -->
          <el-col :span="4"><div class="grid-content bg-purple"><el-button type="primary" @click="addDialogVisible = true">添加用户</el-button></div></el-col>
        </el-row>
        <!-- 用户列表区域 -->
        <el-table
          :data="userList"
          style="width: 100%" stripe border>
          <el-table-column type="index" label="序号"></el-table-column>
          <el-table-column
            prop="username"
            label="姓名">
          </el-table-column>
          <el-table-column
            prop="email"
            label="邮件">
          </el-table-column>
          <el-table-column
            prop="mobile"
            label="电话">
          </el-table-column>
          <el-table-column
            prop="role_name"
            label="角色">
          </el-table-column>
          <!-- 状态栏 -->
          <el-table-column label="状态">
            <template v-slot="scope">
              <el-switch v-model="scope.row.mg_state" @change="changeState(scope.row)">
              </el-switch>
            </template>
          </el-table-column>
          <!-- 操作区域 ---修改、删除、设置-->
          <el-table-column label="操作">
              <template v-slot="scope">
                <el-tooltip class="item" effect="dark" content="修改" placement="top" :enterable="false">
                  <el-button type="primary" icon="el-icon-edit" circle @click="showModifyDialog(scope.row.id)"></el-button>
                </el-tooltip>
                <el-tooltip class="item" effect="dark" content="删除" placement="top" :enterable="false">
                  <el-button type="danger" icon="el-icon-delete" circle @click="removeUserById(scope.row.id)"></el-button>
                </el-tooltip>
                <el-tooltip class="item" effect="dark" content="设置" placement="top" :enterable="false">
                  <el-button type="warning" icon="el-icon-setting" circle></el-button>
                </el-tooltip>
              </template>
          </el-table-column>
        </el-table>
        <!-- 修改弹出对话框 -->
        <el-dialog
          title="修改用户信息"
          :visible.sync="modifyDialogVisible"
          width="50%" @close="modifyFormReset">
          <!-- 主体内容区域 -->
          <el-form :model="modifyForm" :rules="modifyFormRules" ref="modifyFormRef" label-width="100px" class="demo-ruleForm">
            <el-form-item label="用户" prop="username">
              <el-input v-model="modifyForm.username" disabled></el-input>
            </el-form-item>
            <el-form-item label="邮件" prop="email">
              <el-input v-model="modifyForm.email"></el-input>
            </el-form-item>
            <el-form-item label="电话" prop="mobile">
              <el-input v-model="modifyForm.mobile"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="modifyDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="modifyUsersInfo">确 定</el-button>
          </span>
        </el-dialog>

        <!-- 分页区域 -->
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum"
          :page-sizes="[1, 2, 3, 4]"
          :page-size="queryInfo.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>

        <!-- 添加用户的提示框 -->
        <el-dialog
          title="提示"
          :visible.sync="addDialogVisible"
          width="50%" @close="addReset">
          <!-- 主体内容区域 -->
          <el-form :model="addForm" :rules="addFormRuls" ref="addFormRef" label-width="70px" class="demo-addForm">
            <el-form-item label="姓名" prop="username">
              <el-input v-model="addForm.username"></el-input>
            </el-form-item>
            <el-form-item label="密码" prop="password">
              <el-input v-model="addForm.password"></el-input>
            </el-form-item>
            <el-form-item label="邮件" prop="email">
              <el-input v-model="addForm.email"></el-input>
            </el-form-item>
            <el-form-item label="电话" prop="mobile">
              <el-input v-model="addForm.mobile"></el-input>
            </el-form-item>
          </el-form>
          <!-- 底部按钮区域 -->
          <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addUser">确 定</el-button>
          </span>
        </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    const checkEamil = (rule, value, callback) => {
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) {
        callback()
      } else {
        callback(new Error('请输入正确的邮箱地址'))
      }
    }
    const checkMobile = (rule, value, callback) => {
      const regMobile = /^(13[0-9]|14[01456879]|15[0-35-9]|16[2567]|17[0-8]|18[0-9]|19[0-35-9])\d{8}$/
      if (regMobile.test(value)) {
        callback()
      } else {
        callback(new Error('请输入正确的电话号码'))
      }
    }

    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userList: [],
      total: 0,
      // 添加用户的提示所需布尔值
      addDialogVisible: false,
      // 修改用户信息所需布尔值
      modifyDialogVisible: false,
      // 添加用户双向绑定的数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加表单验证规则
      addFormRuls: {
        username: [
          { required: true, message: '请输入用户名称', trigger: 'blur' },
          { min: 3, max: 12, message: '长度需 3 到 12 位', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱地址', trigger: 'blur' },
          { validator: checkEamil, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入电话号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 21, message: '密码长度需在 6 到 21 位', trigger: 'blur' }
        ]
      },
      // 修改用户信息验证规则
      modifyFormRules: {
        username: [
          { required: true, message: '请输入用户名称', trigger: 'blur' },
          { min: 3, max: 12, message: '长度需 3 到 12 位', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱地址', trigger: 'blur' },
          { validator: checkEamil, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入电话号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 修改用户信息数据
      modifyForm: {}
    }
  },
  methods: {
    // 得到用户数据
    async getUsersInfo () {
      const { data: res } = await this.$http.get('/users', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('请求数据失败')
      }
      // console.log(res)
      this.userList = res.data.users
      this.total = res.data.total
    },
    getUsersInfoA () {
      this.queryInfo.pagenum = 1
      this.getUsersInfo()
    },
    // 监听pagesize改变事件
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUsersInfo()
    },
    // 监听页码值改变事件
    handleCurrentChange (newPage) {
      // console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getUsersInfo()
    },
    // 改变用户状态
    async changeState (val) {
      const { data: res } = await this.$http.put(`/users/${val.id}/state/${val.mg_state}`)
      // console.log(res)
      if (res.meta.status !== 200) {
        val.mg_state = !val.mg_state
        return this.$message.error('设置状态失败')
      }
      this.$message.success('设置状态成功')
    },
    // 添加表单重置
    addReset () {
      this.$refs.addFormRef.resetFields()
    },
    // 为添加用户表单进行校验并发起请求
    addUser () {
      this.$refs.addFormRef.validate(async value => {
        if (!value) {
          this.$message.error('添加用户校验失败')
        } else {
          const { data: res } = await this.$http.post('users', this.addForm)
          if (res.meta.status !== 201) {
            this.$message.error('添加用户失败')
          } else {
            this.$message.success('添加用户成功')
            this.addDialogVisible = false
            this.getUsersInfo()
          }
        }
      })
    },
    // 查询用户信息
    async showModifyDialog (id) {
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        this.$message.error('查询失败')
      } else {
        this.$message.success('查询成功')
        this.modifyForm = res.data
        this.modifyDialogVisible = true
      }
    },
    // 修改信息表单重置
    modifyFormReset () {
      this.$refs.modifyFormRef.resetFields()
    },
    // 修改用户信息
    modifyUsersInfo () {
      this.$refs.modifyFormRef.validate(async value => {
        if (!value) return this.$message.error('校验用户信息失败')
        const { data: res } = await this.$http.put('users/' + this.modifyForm.id, {
          email: this.modifyForm.email,
          mobile: this.modifyForm.mobile
        })
        if (res.meta.status !== 200) return this.$message.error('修改用户信息失败')
        this.$message.success('修改用户成功')
        this.modifyDialogVisible = false
        this.getUsersInfo()
      })
    },
    // 删除用户信息
    async removeUserById (id) {
      // console.log(id)
      const value = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // console.log(value)
      if (value !== 'confirm') return this.$message.info('取消操作成功')
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除用户失败')
      this.$message.success('删除信息成功')
      this.getUsersInfo()
    }
  },
  created () {
    this.getUsersInfo()
  }
}
</script>

<style lang="less" scoped>
.el-row {
  margin-bottom: 20px
}
</style>
