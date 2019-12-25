<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!--        提示-->
      <el-alert show-icon
                title="只有三级的分类才能设置参数！"
                type="warning">
      </el-alert>
      <!--        选择商品分类-->
      <el-row class="selectCate">
        <el-col>
          <span class="selecteSpan">选择商品分类:</span>
          <el-cascader
            clearable
            change-on-select
            :props="cascaderProps"
            v-model="selectedKeys"
            :options="parenCateList"
            expand-trigger="hover"
            @change="parentCateChange" @clearCheckedNodes="clearSelected">
          </el-cascader>
        </el-col>
      </el-row>
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <template>
            <el-button @click="addDialogVisble = true" type="primary" size="mini"
                       :disabled="this.selectedKeys.length !== 3 ? true : false">添加参数
            </el-button>
            <!--            表格-->
            <el-table
              :data="manyParams"
              border
              strige>
              <el-table-column type="expand">
                <template slot-scope="scope">
                  <el-tag @close="handleClose(i,scope.row)" closable v-for="(item,i) in scope.row.attr_vals" :key="i" >{{item}}</el-tag>
                  <el-input
                    class="input-new-tag"
                    v-if="scope.row.inputVisible"
                    v-model="scope.row.inputValue"
                    ref="saveTagInput"
                    size="small"
                    @keyup.enter.native="handleInputConfirm(scope.row)"
                    @blur="handleInputConfirm(scope.row)"
                  >
                  </el-input>
                  <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                </template>
              </el-table-column>
              <el-table-column type="index">
              </el-table-column>
              <el-table-column label="参数名称" prop="attr_name">
              </el-table-column>
              <el-table-column
                label="操作"
              >
                <template slot-scope="scope">
                  <el-tooltip :enterable="false" effect="dark" content="编辑" placement="top">
                    <el-button size="mini" type="primary" @click="showEditDialog(scope.row.attr_id)"
                               icon="el-icon-edit"></el-button>
                  </el-tooltip>
                  <el-tooltip :enterable="false" effect="dark" content="删除" placement="top">
                    <el-button size="mini" type="danger" icon="el-icon-delete"
                               @click="removeById(scope.row.attr_id)"></el-button>
                  </el-tooltip>
                </template>
              </el-table-column>
            </el-table>
          </template>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <template>
            <el-button @click="addDialogVisble = true" type="primary" size="mini"
                       :disabled="this.selectedKeys.length !== 3 ? true : false">添加属性
            </el-button>
            <el-table
              :data="onlyParams"
              border
              strige>
              <el-table-column type="expand">
                <template slot-scope="scope">
                  <el-tag @close="handleClose(i,scope.row)" closable v-for="(item,i) in scope.row.attr_vals" :key="i" >{{item}}</el-tag>
                  <el-input
                    class="input-new-tag"
                    v-if="scope.row.inputVisible"
                    v-model="scope.row.inputValue"
                    ref="saveTagInput"
                    size="small"
                    @keyup.enter.native="handleInputConfirm(scope.row)"
                    @blur="handleInputConfirm(scope.row)"
                  >
                  </el-input>
                  <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                </template>
              </el-table-column>
              <el-table-column type="index">
              </el-table-column>
              <el-table-column label="参数名称" prop="attr_name">
              </el-table-column>
              <el-table-column
                label="操作">
                <template slot-scope="scope">
                  <el-tooltip :enterable="false" effect="dark" content="编辑" placement="top">
                    <el-button size="mini" type="primary" @click="showEditDialog(scope.row.attr_id)"
                               icon="el-icon-edit"></el-button>
                  </el-tooltip>
                  <el-tooltip :enterable="false" effect="dark" content="删除" placement="top">
                    <el-button size="mini" type="danger" icon="el-icon-delete"
                               @click="removeById(scope.row.attr_id)"></el-button>
                  </el-tooltip>
                </template>
              </el-table-column>
            </el-table>
          </template>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisble"
      width="50%"
      @close="addDialogClosed">
      <!--      内容主体区-->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef"
               label-width="70px">
        <el-form-item :label="titleText" prop="attr_Name">
          <el-input v-model="addForm.attr_Name"></el-input>
        </el-form-item>
      </el-form>
      <!--      按钮区-->
      <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisble = false">取 消</el-button>
    <el-button type="primary" @click="AddParams">确 定</el-button>
  </span>
    </el-dialog>
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisble"
      width="50%"
      @close="editDialogClosed">
      <!--      内容主体区-->
      <el-form :model="editForm" :rules="addFormRules" ref="editFormRef"
               label-width="70px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <!--      按钮区-->
      <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisble = false">取 消</el-button>
    <el-button type="primary" @click="editParams">确 定</el-button>
  </span>
    </el-dialog>

  </div>
</template>

<script>export default {
  data() {
    return {
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      attr_id: 0,
      selectedKeys: [],
      addDialogVisble: false,
      editDialogVisble: false,
      attrId: 0,
      addForm: {
        attr_Name: ''
      },
      editForm: {
      },
      parenCateList: [],
      activeName: 'many',
      manyParams: [],
      onlyParams: [],
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数', trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 1 到 10 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getParentCateList()
  },
  methods: {
    async parentCateChange() {
      this.getParamsData()
    },
    async getParamsData() {
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.onlyParams = []
        this.manyParams = []
        return
      }
      const cateId = this.selectedKeys[2]
      console.log(this.selectedKeys)
      const { data: res } = await this.$http.get(`categories/${cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败!')
      }
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        item.inputVisible = false
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyParams = res.data
      }
      if (this.activeName === 'only') {
        this.onlyParams = res.data
      }
      console.log(res.data)
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get(`categories`, { params: { type: 3 } })
      if (res.meta.status !== 200) {
        this.$message.error('查询失败')
        return
      }
      console.log(res)
      this.parenCateList = res.data
    },
    handleTabClick() {
      this.getParamsData()
    },
    clearSelected() {
      this.selectedKeys = []
    },
    addDialogClosed() {
      this.addDialogVisble = false
      this.$refs.addFormRef.resetFields()
    },
    editDialogClosed() {
      this.editDialogVisble = false
      this.$refs.editFormRef.resetFields()
    },
    AddParams() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const cateId = this.selectedKeys[2]
        const { data: res } = await this.$http.post(`categories/${cateId}/attributes`,
          {
            attr_name: this.addForm.attr_Name,
            attr_sel: this.activeName
          })
        if (res.meta.status !== 201) {
          this.$message.error('添加失败')
        }
        this.$message.success('添加成功！')
        this.addDialogVisble = false
        this.getParamsData()
      })
    },
    editParams() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const cateId = this.selectedKeys[2]
        const { data: res } = await this.$http.put(`categories/${cateId}/attributes/${this.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName
          })
        console.log(this.editForm.attr_name)
        if (res.meta.status !== 201) {
          this.$message.error('修改失败')
        }
        this.$message.success('修改成功！')
        this.editDialogVisble = false
        this.getParamsData()
      })
    },
    async showEditDialog(attr_id) {
      this.editDialogVisble = true
      this.attr_id = attr_id
      const cateId = this.selectedKeys[2]
      console.log(attr_id)
      const { data: res } = await this.$http.get(`categories/${cateId}/attributes/${attr_id}`, { params: { attr_sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('查询失败')
      }
      this.editForm = res.data
      console.log(this.editForm.attr_name)
    },
    async removeById(id) {
      const cateId = this.selectedKeys[2]
      const { data: res } = await this.$http.delete(`categories/${cateId}/attributes/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除成功')
      this.getParamsData()
    },
    async handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      const cateId = this.selectedKeys[2]
      const { data: res } = await this.$http.put(`categories/${cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) {
        return this.$message.error('添加失败')
      }
      return this.$message.success('修改参数成功')
    },
    showInput(row) {
      row.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    async handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      const cateId = this.selectedKeys[2]
      const { data: res } = await this.$http.put(`categories/${cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) {
        return this.$message.error('添加失败')
      }
      return this.$message.success('修改参数成功')
    }
  },
  computed: {
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态参数'
    }
  }
}
</script>

<style lang="less" scoped>
  .selectCate {
    margin-top: 12px;
    display: flex;
    flex-direction: row;
    justify-items: center;
  }

  .selecteSpan {
    font-size: small;
    margin-right: 12px;
  }
  .el-tag
  {
    margin: 10px;
  }
  .input-new-tag {
    width: 120px;
    margin-left: 10px;

  }
</style>
