<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input placeholder="请输入内容"
                    v-model="queryinfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="showAddUserDialog">添加用户</el-button>
        </el-col>
      </el-row>
      <el-table
          stripe
          :data="users">
          <el-table-column
            type="index">
          </el-table-column>
          <el-table-column
            prop="username"
            label="用户名">
          </el-table-column>
          <el-table-column
            prop="role_name"
            label="角色">
          </el-table-column>
          <el-table-column
            prop="email"
            label="邮箱">
          </el-table-column>
          <el-table-column
            prop="mobile"
            label="手机号">
          </el-table-column>
          <el-table-column
            prop="mg_state"
            label="状态">
          <!--          作用域插槽 -->
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="changeUserStute(scope.row)"
            >
            </el-switch>
          </template>
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
              <el-button size="mini" type="danger" icon="el-icon-delete" @click="showDeleteDialog(scope.row.id)"></el-button>
            </el-tooltip>
            <el-tooltip :enterable="false" effect="dark" content="分配角色" placement="top">
              <el-button @click="setRole(scope.row)" size="mini" type="warning" icon="el-icon-setting"></el-button>
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
    <!--    添加用户的对话框-->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisble"
      width="50%"
      @close="addDialogClosed">
      <!--      内容主体区-->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef"
               label-width="70px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!--      按钮区-->
      <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisble = false">取 消</el-button>
    <el-button type="primary" @click="checkAddUser">确 定</el-button>
  </span>
    </el-dialog>
    <!--    修改用户对话框-->
    <el-dialog
      title="修改信息"
      :visible.sync="editDialogVisble"
      width="50%"
      @close="editDialogClosed">
      <!--      内容主体区-->
      <el-form :model="editForm" :rules="addFormRules" ref="editFormRef"
               label-width="70px" >
        <el-form-item label="用户名" prop="username" >
          <el-input v-model="editForm.username" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!--      按钮区-->
      <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisble = false">取 消</el-button>
    <el-button type="primary" @click="checkEditUser">确 定</el-button>
  </span>
    </el-dialog>
<!--    删除用户-->
    <el-dialog
      title="删除用户"
      :visible.sync="deleteDialogVisble"
      width="30%"
      center
    >
      <span class="delete-span">确认删除？</span>
      <span slot="footer" class="dialog-footer">
    <el-button @click="deleteDialogVisble = false">取 消</el-button>
    <el-button type="primary" @click="checkDeleteUser">确 定</el-button>
  </span>
    </el-dialog>
<!--    分配角色-->
    <el-dialog
      title="分配角色"
      :visible.sync="setRoleDialogVisible"
      width="50%"
    >
      <div>
        <p>当前的用户:{{userInfo.username}}</p>
        <p>当前的角色:{{userInfo.role_name}}</p>
        <p>分配新角色:
          <el-select v-model="selectedID" placeholder="请选择">
            <el-option
              v-for="item in rolesList"
              :key="item.id"
              :label="item.roleName"
              :value="item.id">
            </el-option>
          </el-select>
        </p>
      </div>
      <span slot="footer" class="dialog-footer">
    <el-button @click="setRoleDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
  </span>
    </el-dialog>

  </div>
</template>

<script>export default {
  data() {
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+/
      if (regEmail.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的邮箱！'))
    }
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^1([38]\d|5[0-35-9]|7[3678])\d{8}$/
      if (regMobile.test(value)) {
        return cb()
      }
      cb(new Error('输入合法的手机号'))
    }
    return {
      queryinfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editForm: {
        // username: '',
        // email: '',
        // mobile: ''
      },
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        // 验证密码是否合法
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      users: [],
      total: 0,
      addDialogVisble: false,
      editDialogVisble: false,
      deleteDialogVisble: false,
      setRoleDialogVisible: false,
      userInfo: {},
      deleteId: 0,
      rolesList: [],
      selectedID: {}
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('users', { params: this.queryinfo })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.users = res.data.users
      this.total = res.data.total
      console.log(this.users)
    },
    handleSizeChange(newsize) {
      this.queryinfo.pagesize = newsize
      this.getUserList()
    },
    handleCurrentChange(newpage) {
      this.queryinfo.pagenum = newpage
      this.getUserList()
    },
    async changeUserStute(userState) {
      console.log(userState)
      const { data: res } = await this.$http.put(`users/${userState.id}/state/${userState.mg_state}`)
      if (res.meta.status !== 200) {
        userState.mg_state = !userState.mg_state
        return this.$message.error(res.meta.msg)
      }
      this.$message.success('更新成功！')
    },
    showAddUserDialog() {
      this.addDialogVisble = !this.addDialogVisble
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    checkAddUser() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加失败')
        }
        this.$message.success('添加用户成功！')
        this.addDialogVisble = false
        this.getUserList()
      })
    },
    async showEditDialog(id) {
      this.editDialogVisble = true
      const { data: res } = await this.$http.get(`users/${id}`)
      if (res.meta.status !== 200) {
        this.$message.error('查询失败')
        return
      }
      console.log(res)
      this.editForm = res.data
    },
    checkEditUser() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`users/${this.editForm.id}`, { email: this.editForm.email,
          mobile: this.editForm.mobile })
        if (res.meta.status !== 201) {
          this.$message.error('修改失败')
        }
        this.$message.success('修改用户信息成功！')
        this.editDialogVisble = false
        this.getUserList()
      })
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    showDeleteDialog(id) {
      this.deleteDialogVisble = true
      this.deleteId = id
      console.log(this.deleteId)
    },
    async checkDeleteUser() {
      const { data: res } = await this.$http.delete(`users/${this.deleteId}`)
      if (res.meta.status !== 200) {
        this.$message.error('删除失败')
      }
      this.$message.success('删除用户成功！')
      this.deleteDialogVisble = false
      this.getUserList()
    },
    async setRole(userInfo) {
      this.userInfo = userInfo
      const { data: res } = await this.$http.get(`roles`)
      if (res.meta.status !== 200) {
        this.$message.error('获取角色列表失败')
        return
      }
      this.rolesList = res.data
      this.setRoleDialogVisible = true
    },
    async saveRoleInfo() {
      if (!this.selectedID) {
        return this.$message.error('请选择角色！')
      }
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, { rid: this.selectedID })
      if (res.meta.status !== 200) {
        return this.$message.error('更新用户角色失败')
      }
      this.$message.success('更新用户角色成功')
      this.getUserList()
      this.setRoleDialogVisible = false
    }
  }
}
</script>

<style scoped>
.delete-span
{
  display:flex;
  flex-direction:row;
  justify-content: center;
}
</style>
