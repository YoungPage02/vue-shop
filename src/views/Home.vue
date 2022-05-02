<template>
  <div class="home_container">
    <h1>欢迎来到天道宫</h1>
    <el-button @click="loginOut">退出登录</el-button>
  </div>
</template>

<script>
export default {
  methods: {
    loginOut () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script>

<style lang="less" scoped>

</style>
