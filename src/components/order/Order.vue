<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索框 -->
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query">
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="search"
            ></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <el-table :data="orderList" style="width: 100%" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="order_number" label="订单编号" width="250">
        </el-table-column>
        <el-table-column prop="order_price" label="订单价格"></el-table-column>
        <el-table-column prop="pay_status" label="是否付款">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.pay_status === '0'" type="danger"
              >未付款</el-tag
            >
            <el-tag v-else type="success">已付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货"></el-table-column>
        <el-table-column label="下单时间">
          <template slot-scope="scope">
            <el-tag>{{ scope.row.create_time | dateFormat }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <el-tooltip content="修改订单地址" placement="top" :enterable="false">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showBox"
            ></el-button>
          </el-tooltip>
          <el-tooltip content="查看订单进度" placement="top" :enterable="false">
            <el-button
              type="success"
              icon="el-icon-location"
              size="mini"
              @click="showProgressBox"
            ></el-button>
          </el-tooltip>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
      >
      </el-pagination>
      <!-- 修改地址对话框 -->
      <el-dialog
        title="修改地址"
        :visible.sync="addressDialogVisible"
        width="50%"
        @close="addressDialogClosed"
      >
        <el-form
          :model="addressForm"
          :rules="addressFormRules"
          ref="addressFormRef"
          label-width="100px"
        >
          <el-form-item label="省市区/县" prop="address1">
            <el-cascader
              v-model="addressForm.address1"
              :options="cityData"
              :props="{ expandTrigger: 'hover' }"
            ></el-cascader>
          </el-form-item>
          <el-form-item label="详细地址" prop="address2">
            <el-input v-model="addressForm.address2"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addressDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addressDialogVisible = false"
            >确 定</el-button
          >
        </span>
      </el-dialog>
      <!-- 展示物流进度的对话框 -->
      <el-dialog
        title="物流进度"
        :visible.sync="progressDialogVisible"
        width="50%"
        @close="progressDialogClosed"
      >
        <!-- <el-form
          :model="pogressForm"
          :rules="pogressFormRules"
          ref="pogressFormRef"
          label-width="100px"
        >
        </el-form> -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="progressDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="progressDialogVisible = false"
            >确 定</el-button
          >
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
import cityData from './citydata'
export default {
  data() {
    return {
      // 订单列表
      orderList: [],
      // 查询参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      // 订单总数
      total: 0,
      // 修改地址对话框
      addressDialogVisible: false,
      // 地址表单
      addressForm: {
        address1: [],
        address2: ''
      },
      // 地址表单规则
      addressFormRules: {
        address1: [
          { required: true, message: '请选择省市区', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请输入详细地址', trigger: 'blur' }
        ]
      },
      // 省市区级联数据
      cityData,
      // 物流进度对话框
      progressDialogVisible: false,
      // 物流进度信息
      progressInfo: []
    }
  },
  created() {
    this.getOrderList()
  },
  methods: {
    // 获取订单列表
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败')
      }
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    // 订单列表页数大小发生变化
    handleSizeChange(val) {
      this.queryInfo.pagesize = val
      this.getOrderList()
    },
    // 订单列表页码发生变化
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getOrderList()
    },
    // 搜索订单
    search() {
      console.log(this.queryInfo)
    },
    // 显示地址修改对话框
    showBox() {
      this.addressDialogVisible = true
    },
    // 地址对话框关闭
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields()
    },
    // 显示进度对话框
    async showProgressBox() {
      const result = await this.$http.get('/kuaidi/1106975712662').catch(() => { })
      console.log(result)
      // if (res.meta.status !== 200) {
      //   return this.$message.error('获取物流失败')
      // }
      // this.progressInfo = res.data
      this.progressDialogVisible = true
    },
    // 物流进度对话框关闭
    progressDialogClosed() {

    }
  }
}
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
