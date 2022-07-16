<template>
    <el-container class="home_container">
      <!-- header栏 -->
      <el-header>
        <div class="left">
          <img src="@/assets/大V认证.png" alt="">
          <span>iStudy</span>
        </div>
        <!-- <el-button type="info" @click="loginOut">退出登录</el-button> -->
        <div class="right">
          <img class="ic" src="@/assets/3.1铃铛.png" alt="">
          <img class="head" src="@/assets/99435249_p0_master1200.jpg" alt="" @click="isLogoutChange">
          <div :class="isLogout? 'logout' : 'active' ">
            <div class="item" @click="loginOut">退出登录</div>
          </div>
        </div>
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
            text-color="#000" :default-active="navState"
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
      navState: '',
      isLogout: false
    }
  },
  methods: {
    // 显示退出按钮
    isLogoutChange () {
      this.isLogout = !this.isLogout
    },
    // 退出登录
    loginOut () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      // console.log(res)
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
    position: relative;
    background-color: white;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-left: 5px;
    padding-right: 71px;
    .left {
      cursor: pointer;
      display: flex;
      align-items: center;
      font-size: 24px;
      font-weight: 900;
      color: black;
      img {
        height: 47px;
        margin-right: 2px;
      }
    }
    .right {
      display: flex;
      align-items: center;
      .ic {
        height: 24px;
        margin-right: 15px;
      }
      .head {
        cursor: pointer;
        width: 36px;
        height: 36px;
        border-radius: 50%;
      }
      .logout {
        display: block;
        cursor: pointer;
        position: absolute;
        width: 100px;
        background-color: white;
        right: 29px;
        bottom: -52px;
        border-radius: 10px;
        text-align: center;
        box-shadow: 1px 1px 1px;
        padding: 12px;
        z-index: 2;
      }
      .active {
        display: none;
      }
    }
  }
  // 侧边栏样式
  .el-aside {
    transition: 0.5s;
    background-color: #fff;
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
    background-color: #0c192c;
    overflow: hidden;
  }
</style>
