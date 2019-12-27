<template>
    <div>
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品列表</el-breadcrumb-item>
      </el-breadcrumb>
      <el-card>
        <el-row :gutter="20">
          <el-col :span="7">
            <el-input v-model="queryInfo.query" placeholder="请输入内容"
                       clearable @clear="getGoodsList" >
              <el-button slot="append" icon="el-icon-search" @click="getGoodsList" ></el-button>
            </el-input>
          </el-col>
          <el-col :span="4">
            <el-button type="primary" @click="goAddPage" >添加商品</el-button>
          </el-col>
        </el-row>
<!--        表格区域-->
        <el-table
          stripe
          :data="goodsList">
          <el-table-column
            type="index">
          </el-table-column>
          <el-table-column
            prop="goods_name"
            label="商品名">
          </el-table-column>
          <el-table-column
            prop="goods_price"
            label="价格" width="95px">
          </el-table-column>
          <el-table-column
            prop="goods_weight"
            label="商品重量" width="105px">
          </el-table-column>
          <el-table-column
            prop="add_time"
            label="创建时间" width="125px">
            <template slot-scope="scope">{{scope.row.add_time | dataFormat}}</template>
          </el-table-column>
          <el-table-column
            prop="role_name"
            label="操作" width="180px">
            <template slot-scope="scope">
              <el-tooltip :enterable="false" effect="dark" content="编辑信息" placement="top">
                <el-button size="mini" type="primary" @click="showEditDialog(scope.row.id)"
                           icon="el-icon-edit"></el-button>
              </el-tooltip>
              <el-tooltip :enterable="false" effect="dark" content="删除用户" placement="top">
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeGoodById(scope.row.goods_id)"></el-button>
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum"
          :page-sizes="[1, 2, 5, 10]"
          :page-size="queryInfo.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>
      </el-card>
    </div>
</template>

<script>export default {
  data() {
    return {
      goodsList: [],
      total: 0,
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      }
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList() {
      const { data: res } = await this.$http.get(`goods`, { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败!')
      }
      this.goodsList = res.data.goods
      this.total = res.data.total
      console.log(this.goodsList)
    },
    handleSizeChange(newsize) {
      this.queryInfo.pagesize = newsize
      this.getGoodsList()
    },
    handleCurrentChange(newpage) {
      this.queryInfo.pagenum = newpage
      this.getGoodsList()
    },
    async removeGoodById(goodId) {
      const result = await this.$confirm('此操作将永久删除该角色, 是否继续?', '删除角色', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (result !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete(`goods/${goodId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除成功！')
      this.getGoodsList()
    },
    goAddPage() {
      this.$router.push('/goods/add')
    }
  }
}
</script>

<style lang="less" scoped>

</style>
