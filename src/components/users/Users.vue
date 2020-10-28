<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item><a href="/">用户管理</a></el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <!-- 搜索与添加区域 -->
      <el-row :gutter="15">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="userInfo.query"
            clearable
            @clear="getUserList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="centerDialogVisible = true"
            >添加用户
          </el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="userList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="username" label="姓名"> </el-table-column>
        <el-table-column prop="email" label="邮箱"> </el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="userStateChange(scope.row)"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <!-- 编辑用户按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              @click="showEditForm(scope.row.id)"
            ></el-button>
            <!-- 删除用户按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              @click="deleteUserInfo(scope.row.id)"
            ></el-button>
            <!-- 分配用户权限按钮 -->
            <el-button
              type="warning"
              icon="el-icon-setting"
              @click="showAssignRolesDialog(scope.row)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="userInfo.pagenum"
        :page-sizes="[1, 2, 3, 4]"
        :page-size="2"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>

      <!-- 添加用户对话框 -->
      <el-dialog
        title="添加用户"
        :visible.sync="centerDialogVisible"
        width="50%"
        center
        @close="resetForm('addFormRef')"
      >
        <el-form
          :model="addForm"
          :rules="addFormRules"
          ref="addFormRef"
          label-width="70px"
          status-icon
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input type="password" v-model="addForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="centerDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
          <el-button @click="resetForm('addFormRef')">重置</el-button>
        </span>
      </el-dialog>
      <!-- 修改用户对话框 -->
      <el-dialog
        title="编辑用户"
        :visible.sync="editDialogVisible"
        width="50%"
        center
        @close="resetForm('editFormRef')"
        status-icon
      >
        <el-form
          :model="editForm"
          :rules="editFormRules"
          ref="editFormRef"
          label-width="70px"
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="editForm.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUserInfo">确 定</el-button>
          <el-button @click="resetForm('editFormRef')">重 置</el-button>
        </span>
      </el-dialog>
      <!-- 分配角色对话框 -->
      <el-dialog
        title="分配角色"
        :visible.sync="assignDialogVisible"
        width="50%"
        center
        status-icon
        @close="closeAssignDialog"
      >
        <el-tag>当前用户为：{{ user.username }}</el-tag>
        <el-tag type="success">当前角色为：{{ user.role_name }}</el-tag>
        <el-select v-model="selectedRoleId" placeholder="请选择要分配的角色">
          <el-option
            v-for="item in rolesList"
            :key="item.id"
            :label="item.roleName"
            :value="item.id"
          >
          </el-option>
        </el-select>
        <el-form ref="assignFormRef" label-width="70px"> </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="assignDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="assignRoles">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/
      if (regEmail.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的邮箱'))
    }
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^(13[0-9]|14[5|7]|15[0|1|2|3|4|5|6|7|8|9]|18[0|1|2|3|4|5|6|7|8|9])\d{8}$/
      if (regMobile.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的手机号'))
    }
    return {
      // 修改用户对话框
      editDialogVisible: false,
      // 添加用户对话框默认隐藏
      centerDialogVisible: false,
      // 查询信息
      userInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userList: [],
      total: 0,
      // 添加用户的表单
      addForm: {
        username: 'moon',
        password: '123456',
        email: '123456@qq.com',
        mobile: '18888888888'
      },
      // 添加用户的验证规则
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 编辑用户的表单
      editForm: {
        username: '',
        email: '',
        mobile: ''
      },
      // 编辑用户的验证规则
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 分配角色对话框
      assignDialogVisible: false,
      // 需要被分配角色的用户信息
      user: {},
      // 所有角色的数据列表
      rolesList: [],
      // 已选中的角色Id值
      selectedRoleId: ''

    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('users', { params: this.userInfo })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.userList = res.data.users
      this.total = res.data.total
    },
    // 监听pagesize
    handleSizeChange(newSize) {
      this.userInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听页码
    handleCurrentChange(current) {
      this.userInfo.pagenum = current
      this.getUserList()
    },
    // 监听switch状态
    async userStateChange(userInfo) {
      // console.log(userInfo)
      const { data: res } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message({
        message: res.meta.msg,
        type: 'success'
      })
    },
    // 添加监听关闭添加用户对话框的时间
    // 重置表单
    resetForm(formName) {
      this.$refs[formName].resetFields()
    },
    // 点击按钮，添加新用户
    addUser() {
      // 进行表单预校验
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        // 可以发起用户的网络请求
        const { data: res } = await this.$http.post('users', this.addForm)
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.msg)
        }
        this.$message({
          message: res.meta.msg,
          type: 'success'
        })
        // 隐藏添加用户的对话框
        this.centerDialogVisible = false
        // 重新加载用户列表
        this.getUserList()
      })
    },
    // 显示编辑用户对话框，并将原本数据渲染到输入框中
    async showEditForm(id) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get(`users/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('res.meta.msg')
      }
      this.editForm = res.data
    },
    // 更新用户信息
    async editUserInfo() {
      // 进行预验证
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        // 发起修改用户请求
        const { data: res } = await this.$http.put(`users/${this.editForm.id}`, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        if (res.meta.status !== 200) {
          return this.$message.error('res.meta.msg')
        }
        // 隐藏添加用户的对话框
        this.editDialogVisible = false
        // 重新加载用户列表
        this.getUserList()
        this.$message({
          message: res.meta.msg,
          type: 'success'
        })
      })
    },
    // 删除用户信息
    async deleteUserInfo(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // .then(() => {
      //   this.$message({
      //     type: 'success',
      //     message: '删除成功!'
      //   })
      // }).catch(() => {
      //   this.$message({
      //     type: 'info',
      //     message: '已取消删除'
      //   })
      // })
      if (confirmResult !== 'confirm') {
        return this.$message({
          type: 'info',
          message: '已取消删除'
        })
      }
      const { data: res } = await this.$http.delete(`users/${id}`)
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message({
        message: res.meta.msg,
        type: 'success'
      })
      // 重新加载用户列表
      this.getUserList()
    },
    // 显示分配角色对话框
    async showAssignRolesDialog(user) {
      this.user = user
      // 展示对话框之前，获取所有角色的列表
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.rolesList = res.data
      this.assignDialogVisible = true
    },
    // 分配角色
    async assignRoles() {
      if (this.selectedRoleId === '') {
        return this.$message.error('请选择一个角色')
      }
      const { data: res } = await this.$http.put(`users/${this.user.id}/role`, {
        rid: this.selectedRoleId
      })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.getUserList()
      this.$message({
        message: res.meta.msg,
        type: 'success'
      })
      this.assignDialogVisible = false
    },
    // 监听分配角色对话框的关闭
    closeAssignDialog() {
      this.selectedRoleId = ''
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 10px;
}
</style>
