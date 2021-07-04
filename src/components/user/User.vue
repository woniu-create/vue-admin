<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <el-table :data="userList" stripe border>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <!-- {{ scope.row }} -->
            <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
             <el-button type="primary" icon="el-icon-edit" size="mini" @click="editdialogVisible = true"></el-button>
             <el-button type="danger" icon="el-icon-delete" size="mini"></el-button>
             <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
               <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
             </el-tooltip>
        </el-table-column>
      </el-table>
    <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum"
    :page-sizes="[1, 2, 4, 10]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
    </el-pagination>
    </el-card>

    <!-- 添加用户的对话框 -->
    <el-dialog title="提示" :visible.sync="addDialogVisible"  width="50%" @close="addDialogClosed">
      <!-- 内容主体区 -->
    <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
      <el-form-item label="用户名" prop="username"> <el-input v-model="addForm.username"></el-input></el-form-item>
      <el-form-item label="密码" prop="password"> <el-input v-model="addForm.password"></el-input></el-form-item>
      <el-form-item label="邮箱" prop="email"> <el-input v-model="addForm.email"></el-input></el-form-item>
      <el-form-item label="手机" prop="mobile"> <el-input v-model="addForm.mobile"></el-input></el-form-item>
    </el-form>
      <!-- 按钮区 -->
     <span slot="footer" class="dialog-footer">
      <el-button @click="addDialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="addUser">确 定</el-button>
     </span>
   </el-dialog>

   <!-- 编辑用户对话框 -->
   <el-dialog title="提示" :visible.sync="editdialogVisible" width="50%">
     <span>这是一段信息</span>
     <span slot="footer" class="dialog-footer">
    <el-button @click="editdialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editdialogVisible = false">确 定</el-button>
  </span>
</el-dialog>
   </div>
</template>

<script>
export default {
  data() {
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) {
        return callback()
      }
      callback(new Error('请输入合法的邮箱'))
    }
    var checkMobile = (rule, value, callback) => {
      const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      // const regMobile = /^1(3[0-9]|4[5,7]|5[0,1,2,3,5,6,7,8,9]|6[2,5,6,7]|7[0,1,7,8]|8[0-9]|9[1,8,9])\d{8}$/
      if (regMobile.test(value)) {
        return callback()
      }
      callback(new Error('请输入合法的手机号'))
    }
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userList: [],
      total: 0,
      addDialogVisible: false,
      editdialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
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
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      console.log(res.data.users)
      this.userList = res.data.users
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
    //   console.log(value)
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange(newPage) {
      console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    async userStateChanged(userInfo) {
    //   console.log(userInfo)
      const { data: res } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
      if (res.meta.status !== 200) {
        userInfo.mg_state = !userInfo.mg_state
        return this.$message.error('更新用户状态失败')
      }
      return this.$message.success('更新用户状态成功')
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    addUser() {
      // valid的作用是；校验各个输入字段是否合法，合法则可以发请求
      // log出来的值为true说明校验都通过，false说明不通过
      this.$refs.addFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败!')
        }
        this.$message.success('添加用户成功!')
        this.addDialogVisible = false
        this.getUserList()
      })
    }
  }
}
</script>

<style lang="less" scoped></style>
