<template>
    <div class="login_container">
        <div class="login_box">
            <!-- 头像-->
            <div class="avatar_box">
                <img src="../assets/dou.png">
            </div>
            <!--登录表单区-->
            <el-form ref="loginformref" :model="loginform" :rules="loginrules" label-width="80px" class="loginform" >
                <!-- 用户名和密码 -->
                <el-form-item label="用户名"  prop="username" >
                    <el-input prefix-icon="el-icon-user" v-model="loginform.username" ></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input  prefix-icon="el-icon-lock" v-model="loginform.password" type="password" ></el-input>
                </el-form-item>
                <!-- 按钮-->
                <el-form-item class="btns">
                <el-button type="primary" @click="login">登录</el-button>
                <el-button type="info" @click="reset">重置</el-button>
                </el-form-item>
            </el-form>

        </div>
    </div>
</template>

<script>
export default {
  data () {
    return {
      loginform: {
        username: 'admin',
        password: '123456'
      },

      loginrules: {
        username: [
          // { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 5, max: 15, message: '长度在 5 到 15 个字符', trigger: 'blur' }
        ],
        password: [
          // { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 5, max: 15, message: '长度在 5 到 15 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    reset () { // 重置表单
      this.$refs.loginformref.resetFields()
    },
    login () { // 登入
      this.$refs.loginformref.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('login', this.loginform)
        if (res.meta.status !== 200) return this.$message.error('登陆失败')
        this.$message.success('登陆成功')

        window.sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
      })
    }
  }
}

</script>

<style lang="less" scoped>
.login_container {
    background-color: #2b4b6b ;
    height:100%;
}

.login_box {
    width:450px;
    height:300px;
    background-color: #fff;
    border-radius: 3px;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
}

.avatar_box {
    height: 130px;
    width: 130px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 0 10px #bbb;
    position: relative;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #fff;
    img {
        width: 100%;
        height: 100%;
        border-radius: 50%;
        background-color: rgba(68, 67, 71, 0.301);
    }
}

.btns{
    display: flex;
    justify-content: flex-end;
}

.loginform{
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 0 20px;
    box-sizing: border-box;
}

</style>
