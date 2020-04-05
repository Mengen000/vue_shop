<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img src="../assets/logo.jpg" alt />
      </div>
      <!-- 登录表单区域 -->
      <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules" class="login_form">
          <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user">></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input v-model="loginForm.password" type="password" prefix-icon="iconfont icon-3702mima"></el-input>
        </el-form-item>
        <!-- 按钮区域 -->
        <el-form-item class="btns">
            <el-button type="primary" @click="login">登录</el-button>
            <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  name: "Login",
  data(){
      return{
        //   登录表单数据对象
        loginForm:{
            username:'admin',
            password:'123456'
        },
        // 表单验证规则对象
        loginFormRules:{
            // 验证用户名是否合法
            username:[
                { required: true, message: '请输入用户名', trigger: 'blur' },
                { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
            ],
            // 验证密码是否合法
            password:[
                { required: true, message: '请输入密码', trigger: 'blur' },
                { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
            ]
        }
      }
  },
  methods:{
    //   重置登录表单
    resetLoginForm(){
        // console.log(this)
        this.$refs.loginFormRef.resetFields();
    },
    login(){
        this.$refs.loginFormRef.validate(async(valid)=>{
            if(!valid) return;
            const {data:res}=await this.$http.post('login',this.loginForm);
            if(res.meta.status!==200) return this.$message.error('登陆失败');
            this.$message.success('登陆成功')
            // 1 登陆成功之后的token，保存到客户端sessionStorage中
            //  1.1 项目中除了登录之外的其它api接口，必须登录之后才能访问
            //  1.2 token 只应在当前网站打开期间生效，所以将token保存在sessionStorage中
            console.log(res)
            window.sessionStorage.setItem("token",res.data.token)
            // 2 通过编程式导航跳转到后台主页，路由地址是/home
            this.$router.push("/home")
        })
    }
  }
};
</script>
<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}
.login_box {
  width: 450px;
  height: 300px;
  background: #ffffff;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  .avatar_box {
    height: 100px;
    width: 100px;
    border: 10px double #666;
    border-radius: 50%;
    padding: 10px;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);

    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eeeeee;
    }
  }
}
.login_form{
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 0 20px;
    box-sizing: border-box;
}
.btns{
    display: flex;
    justify-content: flex-end;
}
</style>