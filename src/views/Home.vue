<template>
    <el-container class="home_container">
      <!-- header栏 -->
      <el-header>
        <div>
          <img src="@/assets/Elaina.png" alt="">
          <span>欢 迎 来 到 天 道 宫</span>
        </div>
        <el-button type="info" @click="loginOut">退出登录</el-button>
      </el-header>
      <!-- 主体部分 -->
      <el-container>
        <!-- 侧边栏 -->
        <el-aside :width="collapseStatus ? '64px' : '200px'">
          <!-- 折叠键 -->
          <div @click="toggle"><i :class="collapseStatus ? 'el-icon-s-unfold' : 'el-icon-s-fold'"></i></div>
          <!-- 侧边栏菜单部分 -->
          <el-menu
            class="el-menu-vertical-demo"
            background-color="#515568"
            text-color="#fff" :default-active="navState"
            active-text-color="#409EFF" unique-opened :collapse="collapseStatus" :collapse-transition="false" :router="true">
            <!-- 一级菜单 -->
            <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
              <template slot="title">
                <i :class="iconList[item.id]"></i>
                <span>{{item.authName}}</span>
              </template>
              <!-- 二级菜单 -->
              <el-menu-item :index=" '/' + subItem.path + ''" v-for="subItem in item.children" :key="subItem.id" @click="saveNavState('/' + subItem.path)">
                <template slot="title">
                  <i class="el-icon-menu"></i>
                  <span>{{subItem.authName}}</span>
                </template>
              </el-menu-item>
            </el-submenu>
          </el-menu>
        </el-aside>
        <!-- 内容 -->
        <el-main>
          <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>
</template>

<script>
export default {
  data () {
    return {
      menuList: [],
      iconList: {
        125: 'el-icon-user-solid',
        103: 'el-icon-s-grid',
        101: 'el-icon-s-goods',
        102: 'el-icon-s-order',
        145: 'el-icon-s-marketing'
      },
      collapseStatus: false,
      navState: ''
    }
  },
  methods: {
    // 退出登录
    loginOut () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      console.log(res)
      if (res.meta.status !== 200) return console.log(res.meta.msg)
      this.menuList = res.data
    },
    // 折叠键
    toggle () {
      this.collapseStatus = !this.collapseStatus
    },
    saveNavState (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.navState = window.sessionStorage.getItem('activePath')
    }
  },
  created () {
    this.getMenuList()
    this.navState = window.sessionStorage.getItem('activePath')
  }
}
</script>

<style lang="less" scoped>
  .home_container {
    height: 100%;
  }
  // 头部样式
  .el-header {
    background-color: #556165;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-left: 0;
    div {
      display: flex;
      align-items: center;
      font-size: 20px;
      color: #fff;
      img {
        height: 60px;
        margin-right: 20px;
      }
    }
  }
  // 侧边栏样式
  .el-aside {
    transition: 0.5s;
    background-color: #515568;
    >div {
      height: 24px;
      padding: 5px;
      display: flex;
      justify-content: center;
      font-size: 24px;
      cursor: pointer;
    }
    .el-menu {
      border-right: none;
    }
  }
  .el-main {
    background-color: rgb(234,237,241);
  }
</style>
