<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- card区域 -->
    <el-card class="box-card">
      <!-- 添加按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 添加用户弹出的对话框 -->
      <el-dialog
        title="添加角色信息"
        :visible.sync="addDialogVisible" width="50%" @close="addDialogReset">
        <!-- 内容主体区域 -->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" class="demo-addForm">
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addDialogInfo">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 角色信息区域 -->
      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
          <template v-slot="scope">
            <el-row :class="['bd-bottom',index1 === 0 ? 'bd-top' : '','vcenter']" v-for="(item1,index1) in scope.row.children" :key="item1.id">
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightByIde(scope.row,item1.id)">{{ item1.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二、三级权限 -->
              <el-col :span="19">
                <el-row :class="[index2 === 0 ? '' : 'bd-top','vcenter']" v-for="(item2,index2) in item1.children" :key="item2.id">
                  <!-- 二级权限 -->
                  <el-col :span="9">
                    <el-tag type="success" closable @close="removeRightByIde(scope.row,item2.id)"> {{ item2.authName }} </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="15">
                    <el-tag type="danger" v-for="item3 in item2.children" :key="item3.id" closable @close="removeRightByIde(scope.row,item3.id)">
                      {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="序号"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作">
          <template v-slot="scope">
            <el-button type="primary" icon="el-icon-search" size="mini" @click="getInfoById(scope.row.id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteUserInfo(scope.row.id)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 编辑弹出dialog -->
      <el-dialog
        title="编辑角色信息"
        :visible.sync="editDialogVisible" width="50%" @close="editDialogReset">
        <!-- 内容主体区域 -->
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" class="demo-editForm">
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="editForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="editForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editDialogInfo">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 分配权限弹出消息框 -->
      <el-dialog
        title="分配权限"
        :visible.sync="setRightDialogVisible"
        width="30%">
        <!-- 树形控件内容 -->
        <el-tree :data="rightsList" :props="rightsListProps" ref="treeRef" node-key="id" :default-checked-keys="defaultKey" show-checkbox default-expand-all></el-tree>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRightDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="allotRight">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      // 权限列表数据
      rightsList: [],
      // 树形键控默认展开的节点的 key 的数组
      defaultKey: [],
      // 角色id数据
      roleId: '',
      // 树形控件所需属性
      rightsListProps: {
        children: 'children',
        label: 'authName'
      },
      // 添加用户dialog所需布尔值
      addDialogVisible: false,
      // 编辑用户dialog所需布尔值
      editDialogVisible: false,
      // 分配权限dialog所需布尔值
      setRightDialogVisible: false,
      // 添加用户表单绑定的数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      // 编辑用户表单绑定的数据
      editForm: {},
      // 添加用户表单绑定的数据验证规则
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 3, max: 12, message: '长度在 3 到 12 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ]
      },
      // 编辑用户表单绑定的数据验证规则
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 3, max: 12, message: '长度在 3 到 12 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    // 得到角色列表信息
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取失败')
      // this.$message.success('获取成功')
      // console.log(res.data)
      this.rolesList = res.data
    },
    // 重置addForm表单
    addDialogReset () {
      // console.log(this.$refs)
      this.$refs.addFormRef.resetFields()
    },
    // 重置editform表单
    editDialogReset () {
      this.$refs.editFormRef.resetFields()
    },
    // 校验信息并添加用户
    addDialogInfo () {
      this.$refs.addFormRef.validate(async value => {
        if (!value) return this.$message.error('校验失败')
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) return this.$message.error('添加用户失败')
        this.$message.success('添加用户成功')
        this.getRolesList()
        this.addDialogVisible = false
      })
    },
    // 根据id查询角色信息
    async getInfoById (id) {
      // console.log(id)
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('查询信息失败')
      this.$message.success('查询信息成功')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 校验信息并编辑角色信息
    editDialogInfo () {
      this.$refs.editFormRef.validate(async value => {
        if (!value) return this.$message.error('校验失败')
        const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, {
          roleName: this.editForm.roleName,
          roleDesc: this.editForm.roleDesc
        })
        console.log(res)
        if (res.meta.status !== 200) return this.$message.error('编辑角色失败')
        this.$message.success('编辑角色成功')
        this.editDialogVisible = false
        this.getRolesList()
      })
    },
    // 删除角色信息
    async deleteUserInfo (id) {
      const value = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // console.log(value)
      if (value !== 'confirm') return this.$message.error('删除角色信息失败')
      // console.log(id)
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除角色失败')
      this.$message.success('删除角色成功')
      this.getRolesList()
    },
    // 根据id删除权限
    async removeRightByIde (role, rightId) {
      const value = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (value !== 'confirm') return this.$message.info('取消了删除')
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除权限失败')
      this.$message.success('删除权限成功')
      role.children = res.data
    },
    // 展示分配权限dialog表单
    async showSetRightDialog (role) {
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限列表失败')
      this.rightsList = res.data
      // console.log(res)
      this.defaultKey = []
      this.roleId = role.id
      this.getTreeId(role, this.defaultKey)
      this.setRightDialogVisible = true
    },
    // 通过递归得到树形键控的id
    getTreeId (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getTreeId(item, arr)
      })
    },
    // 分配权限
    async allotRight () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, {
        rids: idStr
      })
      if (res.meta.status !== 200) return this.$message.error('分配权限失败')
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-table {
  margin-top: 20px;
}
.el-tag {
  margin: 7px;
}
.bd-bottom {
  border-bottom: 1px solid #eee;
}
.bd-top {
  border-top: 1px solid #eee;
}
</style>
