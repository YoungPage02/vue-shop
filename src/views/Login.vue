<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 标题logo -->
      <!-- <div class="avatar_box">
        <img src="../assets/logo.png" alt="">
      </div> -->
      <!-- 表单区域-->
      <el-form class="login_form" ref="loginFormRef" :model="loginForm" :rules="loginFormRules">
        <!-- 用户 -->
        <el-form-item prop="username">
          <el-input prefix-icon="el-icon-user" v-model="loginForm.username"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input prefix-icon="el-icon-goods" v-model="loginForm.password" type="password"></el-input>
        </el-form-item >
        <!-- 按钮区域-->
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetLogin">重置</el-button>
        </el-form-item >
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 用户、密码值
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      loginFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, min: 6, max: 15, message: '密码长度需在在 6 到 15 位', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 重置表单
    resetLogin () {
      this.$refs.loginFormRef.resetFields()
    },
    // 登录校验
    login () {
      this.$refs.loginFormRef.validate(async valid => {
        // console.log(valid)
        if (valid) {
          const { data: res } = await this.$http.post('/login', this.loginForm)
          // console.log(res)
          if (res.meta.status === 200) {
            // console.log('登陆成功')
            this.$message({
              message: '登录成功',
              type: 'success'
            })
            window.sessionStorage.setItem('token', res.data.token)
            this.$router.push('/home')
          } else {
            // console.log('登陆失败')
            this.$message.error('登录失败')
          }
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .login_container {
    background: url(@/assets/img/90724581_p0_master1200.jpg) no-repeat;
    background-size: cover;
    height: 100%;
    .login_form {
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      width: 100%;
      box-sizing: border-box;
      padding: 0 20px;
      .btns {
        display: flex;
        justify-content: flex-end;
        margin: 0;
      }
    }
  }
  .login_box {
    height: 300px;
    width: 450px;
    background-color: white;
    opacity: .9;
    border-radius: 5px;
    position: absolute;
    left: 30%;
    top: 50%;
    transform: translate(-50%,-50%);
    /* .avatar_box {
      width: 130px;
      height: 130px;
      border: 1px solid #ddd;
      box-shadow: 0 0 10px #ddd;
      border-radius: 50%;
      padding: 15px;
      position: absolute;
      left: 50%;
      right: 50%;
      transform: translate(-50%,-50%);
      background-color: #fff;
      img {
        width: 100%;
        height: 100%;
        border-radius: 50%;
        // background-color: #eee;
      }
    } */
  }
</style>
