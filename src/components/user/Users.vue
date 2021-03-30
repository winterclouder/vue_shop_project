<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home}' }">首頁</el-breadcrumb-item>
      <el-breadcrumb-item>用戶管理</el-breadcrumb-item>
      <el-breadcrumb-item>用戶列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            v-model="queryInfo.query"
            clearable
            placeholder="請輸入内容"
            class="input-with-select"
            @clear="getUserList()"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList()"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" size="default" @click="addDialogVisible"
            >新增用戶</el-button
          >
        </el-col>
      </el-row>
      <el-table :data="userlist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="信箱" prop="email"></el-table-column>
        <el-table-column label="電話" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="狀態" prop="mg_state">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="stateChanged(scope.row)"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180">
          <template slot-scope="{row}">
            <el-button
              type="primary"
              size="mini"
              icon="el-icon-edit"
              @click="showEditDialog(row.id)"
            ></el-button>
            <el-button
              type="danger"
              size="mini"
              icon="el-icon-delete"
              @click="removeUserId(row.id)"
            ></el-button>
            <el-tooltip
              content="分配角色"
              :enterable="false"
              placement="top"
              effect="dark"
            >
              <!-- content to trigger tooltip here -->
              <el-button
                type="warning"
                size="mini"
                icon="el-icon-setting"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page.sync="queryInfo.pagenum"
        :page-sizes="[2, 4, 8, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- popup -->
    <el-dialog
      title="新增用戶"
      :visible.sync="dialogVisible"
      width="60%"
      @close="addDialogColosed"
    >
      <!-- 內容 -->
      <el-form
        :model="addForm"
        ref="addFormRef"
        :rules="addFormRules"
        label-width="100px"
      >
        <el-form-item label="使用者名稱" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密碼" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="email" prop="email">
          <el-input v-model="addForm.email" prop="email"></el-input>
        </el-form-item>
        <el-form-item label="手機" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>

      <!-- 底部 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible()">取 消</el-button>
        <el-button type="primary" @click="addUser()">
          確 定
        </el-button>
      </span>
    </el-dialog>
    
    <el-dialog
      title="修改使用者"
      :visible.sync="editDialogVisble"
      width="60%"
      @close="editDialogColsed"
    >
     <el-form :model="editForm" ref="editFormRef" :rules="editFormRules" label-width="100px">
       <el-form-item label="使用者名稱">
         <el-input v-model="editForm.username" disabled></el-input>
       </el-form-item>
       <el-form-item label="email" prop="email">
         <el-input v-model="editForm.email"></el-input>
       </el-form-item>
       <el-form-item label="手機" prop="mobile">
         <el-input v-model="editForm.mobile"></el-input>
       </el-form-item>
     </el-form>
      <span slot="footer">
        <el-button @click=" editDialogVisble = false">取消</el-button>
        <el-button type="primary" @click="editUserInfo">確定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    var checkEmail = (rule, value, callback) => {
      // 驗證 email
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) {
        return callback()
      }
      callback(new Error('請輸入合法email'))
    }
    var checkMobile = (rule, value, callback) => {
      // 驗證手機
      const regMobile = /^09\d{8}$/
      if (regMobile.test(value)) {
        return callback()
      }
      callback(new Error('請輸入合法手機'))
    }
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      dialogVisible: false,
      editDialogVisble: false,
      userlist: [],
      total: 0,
      editForm: {},
      addForm: {
        username: null,
        password: null,
        email: null,
        mobile: null
      },
      addFormRules: {
        username: [
          { required: true, message: '請輸入使用者名稱', trigger: 'blur' },
          { min: 3, max: 10, message: '請輸入3-10', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '請輸入密碼', trigger: 'blur' },
          { min: 3, max: 10, message: '請輸入3-10', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '請輸入 email', trigger: 'blur' },
          {
            validator: checkEmail,
            message: '請輸入合法 email',
            trigger: 'blur'
          }
        ],
        mobile: [
          { required: true, message: '請輸入 mobile', trigger: 'blur' },
          {
            validator: checkMobile,
            message: '請輸入合法手機號碼',
            trigger: 'blur'
          }
        ]
      },
      editFormRules: {
        password: [
          { required: true, message: '請輸入密碼', trigger: 'blur' },
          { min: 3, max: 10, message: '請輸入3-10', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '請輸入 email', trigger: 'blur' },
          {
            validator: checkEmail,
            message: '請輸入合法 email',
            trigger: 'blur'
          }
        ],
        mobile: [
          { required: true, message: '請輸入 mobile', trigger: 'blur' },
          {
            validator: checkMobile,
            message: '請輸入合法手機號碼',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    editUserInfo() {
      // edit user info and submit
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return 
        // submit to http
        const params = {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        }
        const {data: res} = await this.$http.put('users/' + this.editForm.id, params)
        if (res.meta.status !== 200) return this.$message.error('更新失敗')
        this.editDialogVisble = false
        this.$message.success('使用者更新成功')
      })
    },
    editDialogColsed() {
      // user close form event
      this.$refs.editFormRef.resetFields()
    },
    async showEditDialog(id) {
      // console.log(id)
      const {data: res} = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('修改失敗')
      }
      this.editForm = res.data
      this.editDialogVisble = true
    },
    async removeUserId(id) {
      const confirmResult = await this.$confirm(
        '是否刪除使用者',
        '提示',
        {
          confirmButtonText: '確定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已經取消刪除')
      }

      const {data: res} = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('刪除失敗')
      }
      this.$message.error('刪除成功')
      this.getUserList()
    },
    showSettingDialog() {

    },
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          return this.$message.error('新增使用者失敗')
        }
        this.$message.success('新增使用者成功')
        this.dialogVisible = false
        this.getUserList()
      })
    },
    addDialogColosed() {
      this.$refs.addFormRef.resetFields()
    },
    addDialogVisible() {
      this.dialogVisible = !this.dialogVisible
    },
    // switch change
    async stateChanged(row) {
      const { data: res } = await this.$http.put(
        `/users/${row.id}/state/${row.mg_state}`
      )
      if (res.meta.status !== 200) {
        row.mg_state = !row.mg_state
        return this.$message.error('更新失敗')
      }
      this.$message.success('更新成功')
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagesize = newPage
      this.getUserList()
    },
    async getUserList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) return this.$message.error('獲取用戶失敗')
      this.userlist = res.data.users
      this.total = res.data.total
    }
  }
}
</script>

<style lang="less" scoped></style>
