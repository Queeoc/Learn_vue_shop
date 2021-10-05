<template>
  <div>
    <!--面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>活动管理</el-breadcrumb-item>
      <el-breadcrumb-item>活动列表</el-breadcrumb-item>
      <el-breadcrumb-item>活动详情</el-breadcrumb-item>
    </el-breadcrumb>

    <!--  卡片视图区-->
    <el-card>
      <!-- 搜索框 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <div>
            <el-input
              placeholder="请输入内容"
              v-model="qurryinfo.query"
              clearable
              @input="getUserList"
            >
              <el-button
                slot="append"
                icon="el-icon-search"
                @click="getUserList"
              ></el-button>
            </el-input>
          </div>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="adddialogVisible = true"
            >添加用户</el-button
          >
        </el-col>
      </el-row>

      <!-- 用户列表区 -->
      <el-table :data="userlist" border stripe>
        <el-table-column label="#" type="index"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态" prop="mg_state">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="userStateChanged(scope.row)"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="200px">
          <template slot-scope="scope">
            <el-tooltip
              effect="dark"
              content="修改角色"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="primary"
                icon="el-icon-edit"
                size="mini"
                @click="showeditdialog(scope.row.id)"
              ></el-button>
            </el-tooltip>

            <el-tooltip
              effect="dark"
              content="删除角色"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
                @click="removeuser(scope.row.id)"
              ></el-button>
            </el-tooltip>

            <el-tooltip
              effect="dark"
              content="分配角色"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="warning"
                icon="el-icon-setting"
                size="mini"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!--  分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="qurryinfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="qurryinfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>

      <!--添加用户对话框 -->
      <el-dialog
        title="添加用户"
        :visible.sync="adddialogVisible"
        width="50%"
        @close="adddiologclose"
      >
        <!-- 主体区 -->
        <el-form
          :model="addform"
          :rules="addformrules"
          ref="addformref"
          label-width="70px"
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addform.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addform.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addform.email"></el-input>
          </el-form-item>
          <el-form-item label="电话" prop="mobile">
            <el-input v-model="addform.mobile"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="adddialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="adduser">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 修改用户对话框 -->
      <el-dialog
        title="修改用户"
        :visible.sync="editdialogVisible"
        width="50%"
        @close="editdialogclose"
      >
        <!--主体区-->
        <el-form
          :model="editform"
          :rules="editformrules"
          ref="editformref"
          label-width="70px"
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="editform.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editform.email"></el-input>
          </el-form-item>
          <el-form-item label="电话" prop="mobile">
            <el-input v-model="editform.mobile"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区-->
        <span slot="footer" class="dialog-footer">
          <el-button @click="editdialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUserinfo">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      qurryinfo: {
        qurry: '',
        // 页数
        pagenum: 1,
        // 每页多少条数据
        pagesize: 5
      },
      userlist: [],
      total: 0,
      // 控制用户对话框
      adddialogVisible: false,
      // 表单数据
      addform: {},
      // 验证规则
      addformrules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 5,
            max: 15,
            message: '长度在 5 到 15 个字符之间',
            trigger: 'blur'
          }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 8,
            max: 15,
            message: '长度在 5 到 15 个字符之间',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          {
            min: 5,
            max: 20,
            message: '长度在 5 到 15 个字符之间',
            trigger: 'blur'
          }
        ],
        mobile: [
          { required: true, message: '请输入电话', trigger: 'blur' },
          {
            min: 5,
            max: 15,
            message: '长度在 5 到 15 个字符之间',
            trigger: 'blur'
          }
        ]
      },
      editdialogVisible: false,
      editform: {},
      editformrules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          {
            min: 5,
            max: 20,
            message: '长度在 5 到 15 个字符之间',
            trigger: 'blur'
          }
        ],
        mobile: [
          { required: true, message: '请输入电话', trigger: 'blur' },
          {
            min: 5,
            max: 15,
            message: '长度在 5 到 15 个字符之间',
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
    async getUserList() {
      // 获取用户列表数据
      const { data: res } = await this.$http.get('users', {
        params: this.qurryinfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('failed')
      }
      this.userlist = res.data.users
      this.total = res.data.total
      console.log(res)
    },
    // 监听 pagesize 改变的事件
    handleSizeChange(newSize) {
      console.log(newSize)
      this.qurryinfo.pagesize = newSize
      this.getUserList()
    },
    // 监听 页码值 改变的事件
    handleCurrentChange(newpage) {
      console.log(newpage)
      this.qurryinfo.pagenum = newpage
      this.getUserList()
    },
    // 监听switch开关状态的改变
    async userStateChanged(userinfo) {
      console.log(userinfo)
      const { data: res } = await this.$http.put(
        `users/${userinfo.id}/state/${userinfo.mg_state}`
      )
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('更新用户失败')
      }
      this.$message.success('更新用户状态成功')
    },
    // 添加用户对话框关闭
    adddiologclose() {
      this.$refs.addformref.resetFields()
    },
    // 修改用户对话框关闭
    editdialogclose() {
      this.$refs.editformref.resetFields()
    },
    adduser() {
      this.$refs.addformref.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        // 添加用户的网络请求
        const { data: res } = await this.$http.post('users', this.addform)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败')
        }
        this.$message.success('添加用户成功')
        this.adddialogVisible = false // 关闭表单
        this.getUserList() // 重置列表
      })
    },
    // 查询修改用户并弹窗
    async showeditdialog(id) {
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户失败')
      }
      this.editform = res.data
      this.editdialogVisible = true
    },
    // 修改用户结束并上传
    editUserinfo() {
      this.$refs.editformref.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        // 修改用户的网络请求
        const { data: res } = await this.$http.put(
          'users/' + this.editform.id,
          { email: this.editform.email, mobile: this.editform.mobile }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改用户失败')
        }
        this.editdialogVisible = false // 关闭表单
        this.getUserList() // 重置列表
        this.$message.success('更新用户成功')
      })
    },
    // 根据id删除对应的用户
    async removeuser(id) {
      // 弹窗并提示用户
      const confirmresult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      )
        .then(async () => {
          const { data: res } = await this.$http.delete('users/' + id)
          if (res.meta.status !== 200) {
            return this.$message.success('删除失败')
          }
          this.$message.success('删除成功')
          this.getUserList()
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
      console.log(confirmresult)
    }
  }
}
</script>

<style lang="less" scoped></style>
