<template>
    <div>
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>角色管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
      </el-breadcrumb>
      <el-card>
        <el-button type="primary" @click="addDialogVisble = true">添加角色</el-button>
        <el-table
          :data="rolesList"
          border
          strige>
          <el-table-column type="expand">
            <template slot-scope="scope">
              <el-row :class="['bdbottom',i1 ===0 ? 'bdtop' : '','vcenter']"
                      v-for="(item1,i1) in scope.row.children" :key="item1.id">
<!--                渲染一级权限-->
                <el-col :span="5">
                  <el-tag @close="removeRightById(scope.row,item1.id)" closable>{{item1.authName}}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <el-col :span="19">
                  <el-row :class="[i2 === 0 ? '' : 'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                    <el-col :span="6">
                      <el-tag @close="removeRightById(scope.row,item2.id)" closable type="success">{{item2.authName}}</el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
<!--                    三级权限-->
                    <el-col :span="18">
                        <el-tag @close="removeRightById(scope.row,item3.id)" closable type="warning" v-for="(item3) in item2.children" :key="item3.id">{{item3.authName}}</el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
            </template>
          </el-table-column>
          <el-table-column type="index">
          </el-table-column>
          <el-table-column
            prop="roleName"
            label="角色名称"
          >
          </el-table-column>
          <el-table-column
            prop="roleDesc"
            label="角色描述"
          >
          </el-table-column>
          <el-table-column
            label="操作" width="180px">
            <template slot-scope="scope">
              <el-tooltip :enterable="false" effect="dark" content="编辑" placement="top">
                <el-button size="mini" type="primary" @click="showEditDialog(scope.row.id)"
                           icon="el-icon-edit"></el-button>
              </el-tooltip>
              <el-tooltip :enterable="false" effect="dark" content="删除" placement="top">
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRoleById(scope.row.id)"></el-button>
              </el-tooltip>
              <el-tooltip :enterable="false" effect="dark" content="分配权限" placement="top">
                <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)"></el-button>
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>
      </el-card>

      <el-dialog
        title="分配权限"
        :visible.sync="SetRightDialogVisble"
        width="50%"
        @close="SetRightDialogVisble = false">
        <!--      内容主体区-->
        <el-tree ref="treeRef" default-expand-all show-checkbox node-key="id" :default-checked-keys="defKeys" :data="rightslist" :props="treeProps"></el-tree>
        <!--      按钮区-->
        <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisble = false">取 消</el-button>
    <el-button type="primary" @click="allotRights"  >确 定</el-button>
  </span>
      </el-dialog>

    </div>
</template>

<script>export default {
  data() {
    return {
      rolesList: [],
      rightslist: [],
      roleId: '',
      defKeys: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      addDialogVisble: false,
      SetRightDialogVisble: false,
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' },
          { min: 1, max: 10, message: '长度在 1 到 10 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取失败！')

      this.rolesList = res.data
      console.log(res.data)
    },
    checkAddRole() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加失败')
        }
        this.$message.success('添加角色成功！')
        this.addDialogVisble = false
        this.getRolesList()
      })
    },
    addDialogClosed() {
      this.addDialogVisble = false
    },
    async removeRoleById(id) {
      const result = await this.$confirm('此操作将永久删除该角色, 是否继续?', '删除角色', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (result !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除成功！')
      this.getRolesList()
    },
    async removeRightById(role, RightId) {
      const result = await this.$confirm('此操作将永久删除该角色, 是否继续?', '删除角色', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (result !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${RightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除成功！')
      role.children = res.data
    },
    async showSetRightDialog(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取失败！')
      this.rightslist = res.data
      this.defKeys = []
      this.getLeafKeys(role, this.defKeys)
      this.SetRightDialogVisble = true
    },
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      console.log(keys)
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) return this.$message.error('添加权限失败！')
      this.$message.success('添加权限成功！')
      this.SetRightDialogVisble = false
      this.getRolesList()
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag
{
  margin: 7px;
}
  .bdtop{
    border-top: 1px solid #eeeeee;
  }
  .bdbottom{
    border-bottom: 1px solid #eeeeee;

  }
  .vcenter{
    display:flex;
    align-items: center;
  }
</style>
