<template>
  <div>
    <!-- 面包屑导航 -->
  <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>商品管理</el-breadcrumb-item>
    <el-breadcrumb-item>商品分类</el-breadcrumb-item>
  </el-breadcrumb>
  <!-- 卡片区域 -->
  <el-card>
    <!-- 添加分类按钮 -->
    <el-row>
      <el-col>
        <el-button type="primary" @click="showAddDialog">添加分类</el-button>
      </el-col>
    </el-row>
    <!-- 添加分类 打开dialog -->
    <el-dialog
      title="添加分类"
      :visible.sync="addDialogVisible" @close="addCateDialogReset"
      width="50%" >
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" class="demo-addForm">
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父类名称:" prop="cat_pid">
          <!-- Cascader 级联选择器 -->
          <el-cascader
            v-model="selectKeys" clearable
            :options="parentCateList"
            :props="{ expandTrigger: 'hover',value: 'cat_id',label: 'cat_name',children: 'children',checkStrictly: true }"
            @change="cascaterChange"></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addNewCate">确 定</el-button>
      </span>
    </el-dialog>
    <!-- table区域 -->
    <el-table
      :data="cateList"
      style="width: 100%;margin-bottom: 20px;"
      row-key="cat_id" border
      :tree-props="{children: 'children'}">
      <!-- <el-table-column type="index" label="序号"  width="180">
      </el-table-column> -->
      <el-table-column prop="cat_name" label="分类名称"  >
      </el-table-column>
      <el-table-column label="是否有效"  >
        <template v-slot="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color : lightgreen"></i>
          <i class="el-icon-error" v-else style="color : red"></i>
        </template>
      </el-table-column>
      <el-table-column label="排序"  >
        <template v-slot="scope">
          <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="danger" v-else>三级</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作"  >
        <template v-slot="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="getGoodsInfoById(scope.row.cat_id)">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteGoodsInfo(scope.row.cat_id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 打开编辑商品分类dialog -->
    <el-dialog
      title="提示" :visible.sync="editDialogVisible" width="50%">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" class="demo-editForm">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editGoodsInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分页区域 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[3, 5, 10, 15]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
  </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 商品分类的数据列表
      cateList: [],
      // 请求参数
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 一级商品分类总数
      total: 0,
      // 添加分类dialog所需布尔值
      addDialogVisible: false,
      // 编辑商品名称dialog所需布尔值
      editDialogVisible: false,
      // 添加分类的表单所需数据
      addForm: {
        cat_pid: 0,
        cat_name: '',
        cat_level: 0
      },
      // 双向绑定编辑的商品分类数据
      editForm: {},
      // 父级分类列表数据
      parentCateList: [],
      // 选中的父级分类的id数组
      selectKeys: [],
      // 添加分类的表单的校验规则
      addFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
          { min: 2, max: 15, message: '长度在 2 到 15 个字符', trigger: 'blur' }
        ]
      },
      // 编辑分类表单的验证规则
      editFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
          { min: 2, max: 15, message: '长度在 2 到 15 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 得到商品分类的数据
    async getCateList () {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      // console.log(res)
      if (res.meta.status !== 200) return this.$message.error('获取商品分类数据失败')
      this.cateList = res.data.result
      this.total = res.data.total
    },
    // 改变每页显示多少条数据
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 改变当前页码值
    handleCurrentChange (newNum) {
      this.queryInfo.pagenum = newNum
      this.getCateList()
    },
    // 打开添加分类dialog
    showAddDialog () {
      this.getParentCateList()
      this.addDialogVisible = true
    },
    // 得到父级分类列表数据
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', {
        params: {
          type: 2
        }
      })
      // console.log(res)
      this.parentCateList = res.data
    },
    // cascater选择器内容发生改变
    cascaterChange () {
      console.log(this.selectKeys)
      if (this.selectKeys.length > 0) {
        this.addForm.cat_pid = this.selectKeys[this.selectKeys.length - 1]
        this.addForm.cat_level = this.selectKeys.length
      } else {
        this.addForm.cat_pid = 0
        this.addForm.cat_level = 0
      }
    },
    // 点击确定按钮添加新的商品分类
    addNewCate () {
      this.$refs.addFormRef.validate(async value => {
        if (!value) return this.$message.error('校验失败')
        const { data: res } = await this.$http.post('categories', this.addForm)
        if (res.meta.status !== 201) return this.$message.error('添加分类失败')
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addDialogVisible = false
      })
    },
    // 重置添加分类表单
    addCateDialogReset () {
      this.$refs.addFormRef.resetFields()
      this.selectKeys = []
      this.addForm.cat_pid = 0
      this.addForm.cat_level = 0
    },
    // 根据ID查询编辑商品分类的数据
    async getGoodsInfoById (id) {
      const { data: res } = await this.$http.get('categories/' + id)
      if (res.meta.status !== 200) return this.$message.error('获取信息失败')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 点击确定按钮，校验信息，提交编辑的商品分类信息
    editGoodsInfo () {
      this.$refs.editFormRef.validate(async value => {
        if (!value) return this.$message.error('校验信息失败')
        const { data: res } = await this.$http.put('categories/' + this.editForm.cat_id, {
          cat_name: this.editForm.cat_name
        })
        console.log(this.editForm)
        console.log(res)
        if (res.meta.status !== 200) return this.$message.error('更新失败')
        this.$message.success('更新成功')
        this.getCateList()
        this.editDialogVisible = false
      })
    },
    // 删除商品分类信息
    async deleteGoodsInfo (id) {
      const value = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // console.log(value)
      if (value !== 'confirm') return this.$message.info('取消删除')
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除商品失败')
      this.$message.success('删除商品成功')
      this.getCateList()
    }
  },
  created () {
    this.getCateList()
  }
}
</script>

<style lang="less" scoped>
.el-table {
  margin-top: 20px;
}
.el-cascader{
  width: 100%;
}
</style>
