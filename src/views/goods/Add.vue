<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- card区域 -->
    <el-card>
      <!-- 消息提示区域 -->
      <el-alert
        title="添加商品信息" center :closable="false"
        type="info">
      </el-alert>
      <!-- 步骤条 -->
      <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- form表单 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" class="demo-addForm" label-position="top">
        <!-- tabs标签页 -->
        <el-tabs tab-position="left" v-model="activeIndex" :before-leave="beforeTabsLeave" @tab-click="checkTabs">
          <!-- 基本信息区域 -->
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model.number="addForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model.number="addForm.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model.number="addForm.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="addForm.goods_cat"
                :options="cateList"
                :props="{ expandTrigger: 'hover' , value:'cat_id' , label:'cat_name' , children: 'children'}"
                @change="handleChange">
              </el-cascader>
            </el-form-item>
          </el-tab-pane>
          <!-- 商品参数区域 -->
          <el-tab-pane label="商品参数" name="1">
            <el-form-item :label="item.attr_name" v-for="(item) in manyTableData" :key="item.attr_id">
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox :label="cb" border v-for="(cb,i) in item.attr_vals" :key="i"></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <!-- 商品属性区域 -->
          <el-tab-pane label="商品属性" name="2">
            <el-form-item :label="item.attr_name" v-for="(item) in onlyTableData" :key="item.attr_id">
              <el-input v-model="item.attr_vals" placeholder="请输入内容"></el-input>
            </el-form-item>
          </el-tab-pane>
          <!-- 商品图片区域 -->
          <el-tab-pane label="商品图片" name="3">
            <el-upload
              class="upload-demo"
              action="http://127.0.0.1:8888/api/private/v1/upload"
              :on-preview="handlePreview"
              :on-remove="handleRemove" :headers="handleHeaders" :on-success="handleSuccess"
              list-type="picture">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <!-- 商品内容区域 -->
          <el-tab-pane label="商品内容" name="4">
            <!-- 富文本编辑器组件 -->
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <el-button type="primary" class="addbtn" @click="addGoods">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
      <!-- 观察图片大图 -->
      <el-dialog
        title="图片"
        :visible.sync="previewDialogVisible"
        width="50%">
        <img :src="previewUrl" class="previewPic">
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
    // 步骤条状态
      activeIndex: 0,
      // 添加商品的数据
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_number: 0,
        goods_weight: 0,
        // 商品分类级联选择器所选的id
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      // 为上传商品图片添加token值
      handleHeaders: {
        Authorization: window.sessionStorage.getItem('token')
      },
      // 商品分类数据
      cateList: [],
      // 一级商品动态参数数据
      manyTableData: [],
      // 一级商品静态属性数据
      onlyTableData: [],
      /* // 商品分类级联选择器所选的id
      selectedKeys: [], */
      // 添加商品的规则
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ]
      },
      // 图片展示图的地址
      previewUrl: '',
      // 打开图片大图所需布尔值
      previewDialogVisible: false
    }
  },
  methods: {
    // 得到商品分类数据
    async getGoodsCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('得到商品分类数据失败')
      this.cateList = res.data
    },
    // 商品分类级联选择器内容改变
    handleChange () {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
      console.log(this.addForm.goods_cat)
    },
    // 判断tabs是否可以切换
    beforeTabsLeave (activeName, oldActiveName) {
      console.log(activeName, oldActiveName)
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        return false
      }
    },
    // 切换tabs标签
    async checkTabs () {
      // 切换到商品参数区域
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: {
            sel: 'many'
          }
        })
        if (res.meta.status !== 200) return this.$message.error('获取参数失败')
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(',')
        })
        this.manyTableData = res.data
        console.log(this.manyTableData)
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: {
            sel: 'only'
          }
        })
        if (res.meta.status !== 200) return this.$message.error('获取参数失败')
        /* res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        }) */
        this.onlyTableData = res.data
        console.log(this.onlyTableData, '这是静态属性数据')
      }
    },
    // 查看商品图片缩略图
    handlePreview (file) {
      console.log(file)
      this.previewUrl = file.response.data.url
      this.previewDialogVisible = true
    },
    // 取消商品图片缩略图
    handleRemove (file) {
      const filePath = file.response.data.tmp_path
      const i = this.addForm.pics.findIndex(x =>
        x.pic === filePath
      )
      this.addForm.pics.splice(i, 1)
      console.log(this.addForm)
    },
    // 上传图片成功
    handleSuccess (res) {
      const picInfo = { pic: res.data.tmp_path }
      this.addForm.pics.push(picInfo)
      console.log(this.addForm)
    },
    // 添加商品
    addGoods () {
      this.$refs.addFormRef.validate(async value => {
        if (!value) {
          return this.$message.error('请填写表单必填数据')
        }
        // 执行添加商品的业务逻辑
        //  深拷贝
        const newAddForm = JSON.parse(JSON.stringify(this.addForm))
        newAddForm.goods_cat = newAddForm.goods_cat.join(',')
        // 添加动态参数数据
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(',')
          }
          this.addForm.attrs.push(newInfo)
        })
        // 添加静态参数数据
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.addForm.attrs.push(newInfo)
        })
        newAddForm.attrs = this.addForm.attrs
        console.log(newAddForm, '深拷贝后添加商品的数据')
        const { data: res } = await this.$http.post('goods', newAddForm)
        console.log(res, '得到的数据')
        if (res.meta.status !== 201) return this.$message.error('添加商品失败')
        this.$message.success('添加商品成功')
        this.$router.push('/goods')
      })
    }
  },
  created () {
    this.getGoodsCateList()
  },
  computed: {
    cateId () {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
.previewPic {
  width: 100%;
}
.addbtn {
  margin-top: 15px;
}
</style>
