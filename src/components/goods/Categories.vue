<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <tree-table class="treeTable" :data="cateList" :columns="columns" :selection-type="false" :expand-type="false"
                  show-index index-text="#" border>
        <template slot="isok" slot-scope="scope">
          <i v-if="scope.row.cat_deleted === false" class="el-icon-success" style="color: lightgreen"></i>
          <i v-else class="el-icon-error" style="color: red"></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag v-if="scope.row.cat_level === 1" type="success">二级</el-tag>
          <el-tag v-if="scope.row.cat_level === 2" type="warning">三级</el-tag>
        </template>
        <template slot="opt" slot-scope="scope">
          <el-button size="mini" type="primary" @click="showEditDialog(scope.row.id)"
                     icon="el-icon-edit">编辑
          </el-button>
          <el-button size="mini" type="danger" icon="el-icon-delete"
                     @click="removeCateById(scope.row.cat_id)">删除
          </el-button>
        </template>
      </tree-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryinfo.pagenum"
        :page-sizes="[3, 5, 15, 30]"
        :page-size="100"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addDialogClose"
    >
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader
            clearable
            change-on-select
            :props="cascaderProps"
            v-model="selectedKeys"
            :options="parenCateList"
            expand-trigger="hover"
            @change="parentCateChange"></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="addCateDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCate">确 定</el-button>
  </span>
    </el-dialog>
  </div>

</template>

<script>export default {
  data() {
    return {
      cateList: [],
      selectedKeys: [],
      parenCateList: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      addForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名', trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
        ]
      },
      addCateDialogVisible: false,
      queryinfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      columns: [{
        label: '分类名称',
        prop: 'cat_name'
      }, {
        label: '是否有效',
        type: 'template',
        template: 'isok'
      }, {
        label: '排序',
        type: 'template',
        template: 'order'
      }, {
        label: '操作',
        type: 'template',
        template: 'opt'
      }
      ]
    }
  },
  created() {
    this.getCategoriesList()
  },
  methods: {
    async getCategoriesList() {
      const { data: res } = await this.$http.get(`categories`, { params: this.queryinfo })
      if (res.meta.status !== 200) {
        this.$message.error('查询失败')
        return
      }
      console.log(res)
      this.cateList = res.data.result
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryinfo.pagesize = newSize
      this.getCategoriesList()
    },
    handleCurrentChange(newPage) {
      this.queryinfo.pagenum = newPage
      this.getCategoriesList()
    },
    showAddCateDialog() {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get(`categories`, { params: { type: 2 } })
      if (res.meta.status !== 200) {
        this.$message.error('查询失败')
        return
      }
      console.log(res)
      this.parenCateList = res.data
    },
    parentCateChange() {
      if (this.selectedKeys.length === 0) {
        this.addForm.cat_pid = 0
        this.addForm.cat_level = 0
        return this.$message.error('请选择父级分类！')
      }
      this.addForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
      this.addForm.cat_level = this.selectedKeys.length
    },
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
      this.selectedKeys = []
      this.addForm.cat_pid = 0
      this.addForm.cat_level = 0
    },

    async addCate() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加失败')
        }
        console.log(res)
        this.$message.success('成功！')
        this.getCategoriesList()
        this.addCateDialogVisible = false
      })
    },
    async removeCateById(catid) {
      const result = await this.$confirm('此操作将永久删除该角色, 是否继续?', '删除角色', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (result !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete(`categories/${catid}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除成功！')
      this.getCategoriesList()
    }
  }
}
</script>

<style lang="less" scoped>
  .treeTable {
    margin-top: 12px;
  }

  .el-cascader {
    width: 100%;
  }
</style>
