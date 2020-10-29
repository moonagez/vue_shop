<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-alert
        title="注意:仅允许为第三级分类设置相关参数"
        type="warning"
        :closable="false"
        show-icon
      >
      </el-alert>

      <!-- 选择商品分类 -->
      <el-row>
        <el-col class="cat_opt">
          <span>选择商品分类：</span>
          <!-- 选择商品分类的级联选择框 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="cateProps"
            @change="handleChange"
          ></el-cascader>
        </el-col>
      </el-row>
      <!-- tab页签 -->
      <el-tabs v-model="activeName" @tab-click="handleClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            :disabled="isBtnDisabled"
            @click="showAddDialogVisible"
          >
            添加参数
          </el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="props">
                <!-- 循环渲染tag标签 -->
                <el-tag
                  :key="i"
                  v-for="(tag, i) in props.row.attr_vals"
                  closable
                  @close="handleClose(i, props.row)"
                >
                  {{ tag }}
                </el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="props.row.inputVisible"
                  v-model="props.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(props.row)"
                  @blur="handleInputConfirm(props.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(props.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                  >修改</el-button
                >
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="deleteParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button
            type="primary"
            :disabled="isBtnDisabled"
            @click="showAddDialogVisible"
            >添加属性</el-button
          >
          <!-- 静态属性表格 -->
          <el-table :data="onlyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="props">
                <!-- 循环渲染tag标签 -->
                <el-tag
                  :key="i"
                  v-for="(tag, i) in props.row.attr_vals"
                  closable
                  @close="handleClose(i, props.row)"
                  >>
                  {{ tag }}
                </el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="props.row.inputVisible"
                  v-model="props.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(props.row)"
                  @blur="handleInputConfirm(props.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(props.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                  >修改</el-button
                >
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="deleteParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数对话框 -->
    <el-dialog
      :title="'添加' + titleName"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
        status-icon
      >
        <el-form-item :label="titleName" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数对话框 -->
    <el-dialog
      :title="'修改' + titleName"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
        status-icon
      >
        <el-form-item :label="titleName" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 商品分类列表
      cateList: [],
      // 级联选择框的配置对象
      cateProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id'
      },
      // 级联选择框双向绑定到的数组
      selectedCateKeys: [],
      // 被激活的页签名称
      activeName: 'many',
      // 动态参数的数据
      manyTableData: [],
      // 静态属性得出数据
      onlyTableData: [],
      // 添加属性对话框
      addDialogVisible: false,
      // 添加参数表单
      addForm: {
        attr_name: '',
        attr_vals: ''
      },
      // 添加参数的验证表单规则
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      // 修改参数对话框
      editDialogVisible: false,
      editForm: {
        attr_name: ''
      },
      editFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取所有的商品分类列表
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.cateList = res.data
    },
    // 级联选择框选择项改变触发
    handleChange() {
      this.getParamsData()
    },
    // tab页签点击事件的处理函数
    handleClick() {
      // console.log(this.activeName)
      this.getParamsData()
    },
    async getParamsData() {
      // 选中的不是三级分类
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        // 防止在二级分类下操作参数
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      // console.log(this.selectedCateKeys)
      // 根据当前的id请求参数
      const { data: res } = await this.$http.get(`categories/${this.cateID}/attributes`, {
        params: { sel: this.activeName }
      })
      if (res.meta.status !== 200) return this.$message.error('获取参数失败')

      // 处理tag标签
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 控制文本框的显示与隐藏
        item.inputVisible = false
        // 文本框的绑定值
        item.inputValue = ''
      })

      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else this.onlyTableData = res.data
    },
    // 显示添加参数对话框
    showAddDialogVisible() {
      this.addDialogVisible = true
    },
    // 添加参数对话框关闭时触发
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 添加参数
    addParams() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.cateID}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加失败！')
        }
        this.$message({
          type: 'success',
          message: '添加成功！'
        })
        this.getParamsData()
        this.addDialogVisible = false
      })
    },
    // 显示修改参数的对话框
    async showEditDialog(attrId) {
      // 根据 ID 查询参数并拿到表单中
      const { data: res } = await this.$http.get(`categories/${this.cateID}/attributes/${attrId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数失败')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 修改参数对话框关闭时触发
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 点击按钮，修改参数
    editParams() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.cateID}/attributes/${this.editForm.attr_id}`, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        this.$message({
          type: 'success',
          message: res.meta.msg
        })
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    // 删除对话框
    async deleteParams(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message({
          type: 'info',
          message: '已取消删除'
        })
      }
      const { data: res } = await this.$http.delete(`categories/${this.cateID}/attributes/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message({
        type: 'success',
        message: '删除成功!'
      })
      this.getParamsData()
    },
    // new tag 回车键或失去焦点触发输入框提交
    async handleInputConfirm(data) {
      if (data.inputValue.trim().length === 0) {
        data.inputValue = ''
        data.inputVisible = false
        return
      }
      // 如果没有return，则则证明输入的内容需要后续处理
      data.attr_vals.push(data.inputValue.trim())
      data.inputValue = ''
      data.inputVisible = false
      // 保存到数据库
      const { data: res } = await this.$http.put(`categories/${this.cateID}/attributes/${data.attr_id}`, {
        attr_name: data.attr_name,
        attr_sel: data.attr_sel,
        attr_vals: data.attr_vals.join(' ')
      })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('添加失败')
      }
      this.$message({
        type: 'success',
        message: '添加成功'
      })
    },
    // 显示new tag输入框
    showInput(data) {
      data.inputVisible = true
      // 让文本框自动获得焦点
      // $nextTick 方法的作用。就是当页面上的元素被重新渲染之后，才会指定回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 删除标签
    async handleClose(i, data) {
      // 数组中删除该项
      data.attr_vals.splice(i, 1)
      const { data: res } = await this.$http.put(`categories/${this.cateID}/attributes/${data.attr_id}`, {
        attr_name: data.attr_name,
        attr_sel: data.attr_sel,
        attr_vals: data.attr_vals.join(' ')
      })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message({
        type: 'success',
        message: '删除成功'
      })
    }
  },
  computed: {
    // 根据是否选中三级分类判断按钮是否可以点击
    isBtnDisabled() {
      if (this.selectedCateKeys.length !== 3) return true
      else return false
    },
    // 分类ID
    cateID() {
      if (this.selectedCateKeys.length !== 0) {
        return this.selectedCateKeys[this.selectedCateKeys.length - 1]
      }
      return null
    },
    // 计算添加对话框的标题
    titleName() {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 5px;
}
.el-tag + .el-tag {
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>
