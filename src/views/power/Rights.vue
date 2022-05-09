<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片背景 -->
    <el-card class="box-card">
      <template>
        <el-table :data="rightsList" stripe style="width: 100%" border>
          <el-table-column type="index" label="序号" >
          </el-table-column>
          <el-table-column prop="authName" label="权限说明" >
          </el-table-column>
          <el-table-column prop="path" label="权限路径" >
          </el-table-column>
          <el-table-column prop="level" label="权限等级" >
            <template v-slot="scope">
              <el-tag v-if="scope.row.level == '0'">一级</el-tag>
              <el-tag type="success" v-else-if="scope.row.level == '1'">二级</el-tag>
              <el-tag type="danger" v-else>三级</el-tag>
            </template>
          </el-table-column>
        </el-table>
      </template>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rightsList: []
    }
  },
  created () {
    this.getRightsList()
  },
  methods: {
    // 得到权限列表信息
    async getRightsList () {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) return this.$message.error('获取权限列表失败')
      this.$message.success('获取权限列表成功')
      this.rightsList = res.data
      // console.log(res.data)
    }
  }
}
</script>

<style lang="less" scoped>

</style>
