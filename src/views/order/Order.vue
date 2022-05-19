<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- card区域 -->
    <el-card>
      <!-- 搜索区域 -->
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- table表格区域 -->
      <el-table
        :data="orderList"
        stripe border
        style="width: 100%">
        <el-table-column type="index" label="序号"> </el-table-column>
        <el-table-column prop="order_number" label="订单编号" > </el-table-column>
        <el-table-column prop="order_price" label="订单价格" > </el-table-column>
        <el-table-column prop="order_pay" label="是否付款" >
          <template v-slot="scope">
            <el-tag type="success" v-if="scope.row.order_pay === '1'">已付款</el-tag>
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货" > </el-table-column>
        <el-table-column prop="create_time" label="下单时间" >
          <template v-slot="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" >
          <template>
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="editDialogVisible = true">编辑</el-button>
            <el-button type="success" icon="el-icon-delete" size="mini" @click="showProcess">查看</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
      <!-- 编辑的dialog区域 -->
      <el-dialog
        title="修改信息"
        :visible.sync="editDialogVisible" @close='editDialogClose'
        width="50%" >
        <!-- 主体内容表单区域 -->
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" class="demo-editForm">
          <el-form-item label="省市区/县" prop="address1">
            <el-cascader
              v-model="editForm.address1"
              :options="cityData"
              :props="{ expandTrigger: 'hover' }" >
            </el-cascader>
          </el-form-item>
          <el-form-item label="详细地址" prop="address2">
            <el-input v-model="editForm.address2"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editDialogVisible = false">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 查看物流数据dialog -->
      <el-dialog
        title="查询物流信息"
        :visible.sync="processDialogVisible"
        width="50%" >
        <!-- timeline时间线 -->
        <el-timeline >
          <el-timeline-item
            v-for="(activity, index) in processInfo"
            :key="index"
            :timestamp="activity.time">
            {{activity.context}}
          </el-timeline-item>
        </el-timeline>
        <span slot="footer" class="dialog-footer">
          <el-button @click="processDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="processDialogVisible = false">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
import cityData from './citydata.js'
export default {
  data () {
    return {
      // 查询订单数据列表所需参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 订单数据列表数据
      orderList: [],
      // 订单数据列表数据总数
      total: 0,
      // 打开编辑dialog所需布尔值
      editDialogVisible: false,
      // 打开查询dialog所需布尔值
      processDialogVisible: false,
      // 修改表单中的数据
      editForm: {
        address1: '',
        address2: ''
      },
      // 物流信息的数据
      processInfo: [
        {
          time: '2018-05-10 09:39:00',
          ftime: '2018-05-10 09:39:00',
          context: '已签收,感谢使用顺丰,期待再次为您服务',
          location: ''
        },
        {
          time: '2018-05-10 08:23:00',
          ftime: '2018-05-10 08:23:00',
          context: '[北京市]北京海淀育新小区营业点派件员 顺丰速运 95338正在为您派件',
          location: ''
        },
        {
          time: '2018-05-10 07:32:00',
          ftime: '2018-05-10 07:32:00',
          context: '快件到达 [北京海淀育新小区营业点]',
          location: ''
        },
        {
          time: '2018-05-10 02:03:00',
          ftime: '2018-05-10 02:03:00',
          context: '快件在[北京顺义集散中心]已装车,准备发往 [北京海淀育新小区营业点]',
          location: ''
        },
        {
          time: '2018-05-09 23:05:00',
          ftime: '2018-05-09 23:05:00',
          context: '快件到达 [北京顺义集散中心]',
          location: ''
        },
        {
          time: '2018-05-09 21:21:00',
          ftime: '2018-05-09 21:21:00',
          context: '快件在[北京宝胜营业点]已装车,准备发往 [北京顺义集散中心]',
          location: ''
        },
        {
          time: '2018-05-09 13:07:00',
          ftime: '2018-05-09 13:07:00',
          context: '顺丰速运 已收取快件',
          location: ''
        },
        {
          time: '2018-05-09 12:25:03',
          ftime: '2018-05-09 12:25:03',
          context: '卖家发货',
          location: ''
        },
        {
          time: '2018-05-09 12:22:24',
          ftime: '2018-05-09 12:22:24',
          context: '您的订单将由HLA（北京海淀区清河中街店）门店安排发货。',
          location: ''
        },
        {
          time: '2018-05-08 21:36:04',
          ftime: '2018-05-08 21:36:04',
          context: '商品已经下单',
          location: ''
        }
      ],
      editFormRules: {
        address1: [
          { required: true, message: '请选择省市区县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请输入详细地址', trigger: 'blur' }
        ]
      },
      // 省市县数据
      cityData
    }
  },
  methods: {
    // 得到订单数据列表数据
    async getOrderList () {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('得到订单数据列表数据失败')
      }
      console.log(res, '订单数据列表数据')
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    // 改变每页显示条数
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    // 改变当前页码
    handleCurrentChange (newNum) {
      this.queryInfo.pagenum = newNum
      this.getOrderList()
    },
    // 编辑表单重置
    editDialogClose () {
      this.$refs.editFormRef.resetFields()
      this.editDialogVisible = false
    },
    showProcess () {
      this.processDialogVisible = true
      console.log(this.processInfo, '物流信息')
    }
    /* // 打开查看物流信息dialog,,,,,,,API接口有问题
    async showProcess () {
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      if (res.meta.status !== 200) return this.$message.error('查询物流信息失败')
      this.$message.success('查询物流信息成功')
      console.log(res, '物流信息数据')
    } */
  },
  created () {
    this.getOrderList()
  }
}
</script>

<style lang="less" scoped>
.el-table{
  margin-top: 20px;
}
.el-pagination{
  margin-top: 20px;
}
.el-cascader {
  width: 100%;
}
</style>
