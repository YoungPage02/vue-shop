<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- card区域 -->
    <el-card class="box-card">
      <!-- 注意区域 -->
      <el-alert class="attention" :closable='false'
        title="注意：只允许为第三级分类设置相关参数"
        type="warning">
      </el-alert>
      <el-row>
        <el-col>
          <span>选择商品分类：</span>
          <!-- 级联选择器 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="{ expandTrigger: 'hover' ,value:'cat_id' ,label:'cat_name', children:'children'}"
            @change="handleChange">
          </el-cascader>
        </el-col>
      </el-row>
      <!-- tabs标签页 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick" >
        <!-- 动态参数 -->
        <el-tab-pane label="动态参数" name="many" >
          <!-- 添加动态参数按钮 -->
          <el-button type="primary" :disabled="isBtnDisable" @click="addDialogVisible = true">动态参数</el-button>
          <el-table :data="manyParamsData" style="width: 100%" stripe border>
            <el-table-column type="expand">
              <template v-slot="scope">
                <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" size="medium" closable @close="handlerClose(i,scope.row)"> {{ item }} </el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="blurInputConfirm(scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" label="序号">
            </el-table-column>
            <el-table-column prop="attr_name" label="参数名称" min-width="570">
            </el-table-column>
            <el-table-column label="操作" min-width="570">
              <template v-slot="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 静态属性 -->
        <el-tab-pane label="静态属性" name="only" >
          <!-- 添加静态参数按钮 -->
          <el-button type="primary" :disabled="isBtnDisable" @click="addDialogVisible = true">静态属性</el-button>
          <el-table :data="onlyParamsData" style="width: 100%" stripe border>
            <el-table-column type="expand">
              <template v-slot="scope">
                <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" size="medium" closable @close="handlerClose(i,scope.row)"> {{ item }} </el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="blurInputConfirm(scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" label="序号"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称" min-width="570"></el-table-column>
            <el-table-column label="操作" min-width="570">
              <template v-slot="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
      <!-- 添加dialog表单 -->
      <el-dialog :title=" '添加' + textTitle " :visible.sync="addDialogVisible" width="50%" @close='addDialogReset'>
        <!-- form表单 主题内容 -->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" class="demo-addForm">
          <el-form-item :label="textTitle + ':' " prop="attr_name">
            <el-input v-model="addForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addGoodsParams">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 编辑dialog表单 -->
      <el-dialog :title=" '编辑' + textTitle " :visible.sync="editDialogVisible" width="50%" @close='editDialogReset'>
        <!-- form表单 主题内容 -->
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" class="demo-editForm">
          <el-form-item :label="textTitle + ':' " prop="attr_name">
            <el-input v-model="editForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editGoodsParams">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 商品分类数据列表
      cateList: [],
      // 级联选择器选中的id
      selectedCateKeys: [],
      // tabs标签页选中的选项:
      activeName: 'many',
      // 商品动态参数
      manyParamsData: [],
      // 商品静态参数
      onlyParamsData: [],
      // 添加dialog所需布尔值
      addDialogVisible: false,
      // 编辑dialog所需布尔值
      editDialogVisible: false,

      // 添加表单双向绑定数据
      addForm: {},
      // 编辑表单双向绑定数据
      editForm: {},
      // 添加dialog校验规则
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' },
          { min: 3, max: 12, message: '长度在 3 到 12 个字符', trigger: 'blur' }
        ]
      },
      // 编辑表单校验规则
      editFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' },
          { min: 3, max: 12, message: '长度在 3 到 12 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 得到商品分类数据
    async getGoodsCate () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取商品信息失败')
      this.cateList = res.data
    },
    // 级联选择器内容改变
    handleChange () {
      // 清空非三级的商品分类
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyParamsData = []
        this.onlyParamsData = []
      }
      console.log(this.selectedCateKeys)
      this.getGoodsParams()
    },
    // tabs标签页改变选项卡
    handleTabClick () {
      if (this.selectedCateKeys.length === 3) {
        this.getGoodsParams()
      }
      console.log(this.activeName)
    },
    // 得到商品参数数据
    async getGoodsParams () {
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
        params: {
          sel: this.activeName
        }
      })
      if (res.meta.status !== 200) return this.$message.error('获得商品参数失败')
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 判断文本框与按钮的显示与隐藏
        item.inputVisible = false
        // 文本框双向绑定数据
        item.inputValue = ''
      })
      console.log(res.data)
      if (this.activeName === 'many') {
        this.manyParamsData = res.data
      } else {
        this.onlyParamsData = res.data
      }
    },
    // 重置添加表单数据
    addDialogReset () {
      this.$refs.addFormRef.resetFields()
    },
    // 重置编辑表单数据
    editDialogReset () {
      this.$refs.editFormRef.resetFields()
    },
    // 校验并添加商品参数
    addGoodsParams () {
      this.$refs.addFormRef.validate(async value => {
        if (!value) return this.$message.error('校验失败')
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) return this.$message.error('添加商品失败')
        this.getGoodsParams()
        this.addDialogVisible = false
      })
    },
    // 校验编辑表单数据，并将编辑后的数据传送到服务器
    editGoodsParams () {
      this.$refs.editFormRef.validate(async value => {
        if (!value) return this.$message.error('校验失败')
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) return this.$message.error('编辑商品失败')
        this.getGoodsParams()
        this.editDialogVisible = false
      })
    },
    // 打开编辑表单，通过id得到表单数据
    async showEditDialog (id) {
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${id}`, {
        params: {
          attr_sel: this.activeName
        }
      })
      if (res.meta.status !== 200) this.$message.error('校验失败')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 删除商品参数
    async removeParams (id) {
      const value = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (value !== 'confirm') return this.$message.info('取消删除')
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${id}`)
      if (res.meta.status !== 200) return this.$message.error('删除参数失败')
      this.$message.success('删除商品成功')
      this.getGoodsParams()
    },
    // 摁下enter键，添加商品分类参数运行的方法
    handleInputConfirm (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      /* const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) return this.$message.error('添加失败')
      this.$message.success('添加成功') */
      this.saveAttr_vals(row)
    },
    // 点击+new tag按钮
    showInput (row) {
      row.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 失去焦点，input内容清零
    blurInputConfirm (row) {
      row.inputValue = ''
      row.inputVisible = false
    },
    // 删除tag参数的数据
    handlerClose (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttr_vals(row)
    },
    // 将attr_vals的数据保存到服务器中
    async saveAttr_vals (row) {
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) return this.$message.error('操作失败')
      this.$message.success('操作成功')
    }

  },
  created () {
    this.getGoodsCate()
  },
  computed: {
    isBtnDisable () {
      if (this.selectedCateKeys.length === 0) {
        return true
      } else {
        return false
      }
    },
    // 三级商品分类的id
    cateId () {
      if (this.selectedCateKeys.length !== 3) {
        return null
      } else {
        return this.selectedCateKeys[2]
      }
    },
    // add.edit表单提示
    textTitle () {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>

<style lang="less" scoped>
.attention {
  margin-bottom: 15px;
}
.el-table {
  margin-top :20px
}
.el-tag {
  margin: 15px;
  font-size: 14px;
}
.input-new-tag {
  margin: 15px;
  width: 120px;
}
.button-new-tag {
  margin: 15px;
}
</style>
