<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input placeholder="请输入内容"
                    v-model="queryinfo.query" clearable @clear="getOrdersList">
            <el-button slot="append" icon="el-icon-search" @click="getOrdersList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary">添加用户</el-button>
        </el-col>
      </el-row>
      <el-table
        stripe
        :data="orderList">
        <el-table-column
          type="index">
        </el-table-column>
        <el-table-column
          prop="order_number"
          label="订单编号">
        </el-table-column>
        <el-table-column
          prop="order_price"
          label="订单价格">
        </el-table-column>
        <el-table-column
          prop="order_pay"
          label="是否付款">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.order_pay === 1" type="danger">未付款</el-tag>
            <el-tag v-else type="success">已付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="is_send"
          label="是否发货">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.is_send === '是'" type="success">已发货</el-tag>
            <el-tag v-else type="danger">未发货</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="create_time "
          label="下单时间">
          <template slot-scope="scope">{{scope.row.create_time | dataFormat}}</template>
        </el-table-column>
        <el-table-column
          prop="role_name"
          label="操作" width="180px">
          <template slot-scope="scope">
            <el-tooltip :enterable="false" effect="dark" content="编辑信息" placement="top">
              <el-button size="mini" type="primary" @click="showEditDialog(scope.row.id)"
                         icon="el-icon-edit"></el-button>
            </el-tooltip>
            <el-tooltip :enterable="false" effect="dark" content="分配角色" placement="top">
              <el-button @click="show(scope.row)" size="mini" type="warning" icon="el-icon-location"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryinfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryinfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>

    <el-dialog
      title="修改信息"
      :visible.sync="editDialogVisble"
      width="50%"
      @close="editDialogVisble = false">
      <!--      内容主体区-->
      <el-form :model="editForm" :rules="addFormRules" ref="editFormRef"
               label-width="70px">
        <el-form-item label="省市区/县" prop="username">
          <el-input v-model="editForm.username"></el-input>
        </el-form-item>
        <el-form-item label="详细地址" prop="password">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
      </el-form>
      <!--      按钮区-->
      <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisble = false">取 消</el-button>
    <el-button type="primary" @click="editDialogVisble = false">确 定</el-button>
  </span>
    </el-dialog>
    <el-dialog
      title="物流进度"
      :visible.sync="DialogVisble"
      width="50%"
      @close="DialogVisble = false">
      <!--      内容主体区-->
      <el-card>
        <div v-for="(item,i) in kdData " :key="i">
          <p>{{item.time}}</p>
          <p>{{item.context}}</p>
        </div>
      </el-card>
    </el-dialog>
  </div>
</template>

<script>export default {
  data() {
    return {
      queryinfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      kdData: [],
      total: 0,
      orderList: [],
      editDialogVisble: false,
      DialogVisble: false,
      editForm: {},
      addFormRules: {
        username: [
          { required: true, message: '请输入省市区/县', trigger: 'blur' }
        ],
        // 验证密码是否合法
        password: [
          { required: true, message: '请输入详细地址', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getOrdersList()
  },
  methods: {
    async getOrdersList() {
      const { data: res } = await this.$http.get('orders', { params: this.queryinfo })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange(newsize) {
      this.queryinfo.pagesize = newsize
      this.getOrdersList()
    },
    handleCurrentChange(newpage) {
      this.queryinfo.pagenum = newpage
      this.getOrdersList()
    },
    showEditDialog() {
      this.editDialogVisble = true
    },
    async show(row) {
      this.DialogVisble = true
      const { data: res } = await this.$http.get(`kuaidi/1106975712662`)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.kdData = res.data
      console.log(res.data)
    }
  }
}
</script>

<style lang="less" scoped>

</style>
