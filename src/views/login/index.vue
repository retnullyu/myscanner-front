<template>
  <div class="login-container">
    <el-form
      ref="loginForm"
      :model="loginForm"
      :rules="loginRules"
      class="login-form"
      auto-complete="on"
      label-position="left"
    >
      <h3 class="login_title">retnullの武器库</h3>
      <el-form-item prop="username">
        <span class="svg-container">
          <svg-icon icon-class="user" />
        </span>
        <el-input
          v-model="loginForm.username"
          class="form-input"
          name="username"
          type="text"
          auto-complete="on"
          placeholder="请输入用户名"
        />
      </el-form-item>

      <el-form-item prop="password">
        <span class="svg-container">
          <svg-icon icon-class="password" />
        </span>
        <el-input
          v-model="loginForm.password"
          :type="pwdType"
          class="form-input"
          name="password"
          auto-complete="on"
          placeholder="请输入密码"
          @keyup.enter.native="handleLogin"
        />
        <span class="show-pwd" @click="showPwd">
          <svg-icon :icon-class="pwdType === 'password' ? 'eye' : 'eye-open'" />
        </span>
      </el-form-item>

      <el-row :gutter="20">
        <el-col :span="16">
          <!-- prod -->
           <!-- <el-form-item prop="codeText"> -->
            <!-- prod -->
            <el-form-item >
            <span class="svg-container">
              <svg-icon icon-class="user" />
            </span>
            <el-input
              v-model="loginForm.codeText"
              class="code-text"
              name="codeText"
              type="text"
              maxlength="100"
              auto-complete="on"
              placeholder="请输入验证码"
            />
          </el-form-item>
        </el-col>
        <el-col :span="8">
          <el-tooltip
            content="[ 点击 ] 刷新验证码"
            placement="right"
            effect="light"
          >
            <el-image
              :src="codeUrl"
              style="cursor: pointer; border-radius: 5px"
              fit="fit"
              @click="changeImageCode"
            >
              <div slot="error" class="image-slot">
                <i class="el-icon-picture-outline" />
              </div>
            </el-image>
          </el-tooltip>
        </el-col>
      </el-row>

      <el-form-item>
        <el-button
          :loading="loading"
          type="primary"
          style="width: 100%"
          @click.native.prevent="handleLogin"
          >登录</el-button
        >
      </el-form-item>

    </el-form>
  </div>
</template>

<script>
export default {
  name: "Login",
  data() {
    const validateUsername = (rule, value, callback) => {
      if (value.length < 2) {
        callback(new Error("用户名不能小于2位"));
      } else {
        callback();
      }
    };
    const validatePass = (rule, value, callback) => {
      if (value.length < 4) {
        callback(new Error("密码不能小于5位"));
      } else {
        callback();
      }
    };
    const validateCodeText = (rule, value, callback) => {
      if (value.length !== 4) {
        callback(new Error("验证码只能是4位"));
      } else {
        callback();
      }
    };
    return {
      loginForm: {
        username: "",
        password: "",
        codeKey: "",
        codeText: "",
      },
      codeUrl: "",
      loginRules: {
        username: [
          { required: true, trigger: "blur", validator: validateUsername },
        ],
        password: [
          { required: true, trigger: "blur", validator: validatePass },
        ],
        codeText: [
          { required: true, trigger: "blur", validator: validateCodeText },
        ],
      },
      loading: false,
      pwdType: "password",
      redirect: undefined,
    };
  },
  watch: {
    $route: {
      handler: function (route) {
        this.redirect = route.query && route.query.redirect;
      },
      immediate: true,
    },
  },
  created() {
    // prod
    // this.randomCodeKey();
    // this.changeImageCode();
    // prod
    
  
  },
  mounted(){
  this.handleLogin()
  },
  methods: {
    // 显示密码
    showPwd() {
      if (this.pwdType === "password") {
        this.pwdType = "";
      } else {
        this.pwdType = "password";
      }
    },
    // 登录
    handleLogin() {
      this.loginForm.username = "root"
      this.loginForm.password = "root"
      const _this = this;
      _this.$refs.loginForm.validate((valid) => {
        if (valid) {
          _this.loading = true;
          _this.$store
            .dispatch("Login", this.loginForm)
            .then(() => {
              _this.loading = false;
              _this.$router.push({ path: this.redirect || "/dashboard" });
            })
            .catch((msg) => {
              this.randomCodeKey();
              this.changeImageCode();
              _this.loading = false;
            });
        } else {
          this.randomCodeKey();
          this.changeImageCode();

          console.log("error submit!!");
          return false;
        }
      });
    },
    changeImageCode() {
      var arr = [
        process.env.VUE_APP_BASE_API,
        "/auth/verify/code",
        "/",
        this.loginForm.codeKey,
        "?r=",
        Math.ceil(Math.random() * 100),
      ];
      var str = arr.join("");
      this.codeUrl = str;
    },
    // 随机 生成 18位 字符串
    randomCodeKey() {
      var s = [];
      var hexDigits = "0123456789abcdefghijklmnopqrstuvwxyz";
      for (var i = 0; i < 24; i++) {
        s[i] = hexDigits.substr(Math.floor(Math.random() * 0x10), 1);
      }
      s[14] = "4"; // bits 12-15 of the time_hi_and_version field to 0010
      s[19] = hexDigits.substr((s[19] & 0x3) | 0x8, 1); // bits 6-7 of the clock_seq_hi_and_reserved to 01
      var uuid = s.join("");
      this.loginForm.codeKey = uuid;
    },
  },
};
</script>

<style rel="stylesheet/scss" lang="scss">
$bg: #2d3a4b;
$light_gray: #eee;

/* reset element-ui css */
.login-container {
  .el-input {
    display: inline-block;
    height: 47px;
    width: 85%;
    font-size: 20px !important;
    input {
      background: transparent;
      border: 0px;
      -webkit-appearance: none;
      border-radius: 0px;
      padding: 12px 5px 12px 15px;
      color: $light_gray;
      height: 47px;
      &:-webkit-autofill {
        -webkit-text-fill-color: #fff !important;
      }
    }
  }
  .el-form-item {
    border: 1px solid rgba(255, 255, 255, 0.1);
    background: rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    color: #454545;
  }
}
</style>

<style rel="stylesheet/scss" lang="scss" scoped>
$dark_gray: #889aa4;
$light_gray: #eee;
.login-container {
  position: fixed;
  height: 100%;
  width: 100%;
  background-size: cover;
  background: url("../../assets/images/198979.jpg") center center no-repeat;
  .login-form {
    position: absolute;
    left: 0;
    right: 0;
    width: 520px;
    max-width: 100%;
    padding: 35px 35px 15px 35px;
    margin: 120px auto;
  }
  .tips {
    font-size: 18px;
    color: #fff;
    margin-bottom: 10px;
    span {
      &:first-of-type {
        margin-right: 16px;
      }
    }
  }
  .svg-container {
    padding: 6px 5px 6px 15px;
    color: $dark_gray;
    vertical-align: middle;
    width: 30px;
    display: inline-block;
  }
  .title {
    font-size: 26px;
    font-weight: 400;
    color: $light_gray;
    margin: 0px auto 40px auto;
    text-align: center;
    font-weight: bold;
  }
  .el-textarea__inner {
    font-size: 20px;
  }
  .show-pwd {
    position: absolute;
    right: 10px;
    top: 7px;
    font-size: 16px;
    color: $dark_gray;
    cursor: pointer;
    user-select: none;
  }
  .login_title {
    // margin: 0px auto 40px auto;
    // text-align: center;
    // color: #29e05d;
    // background: hsl(0, 50%, 45%);
    padding: 20px;
    font: 300%/1 Rockwell, serif;
    font-weight: bold;
    display: block;
    width: 500px;
    height: 80px;
    // color: white;
    color: rgba(224, 70, 27, 0.5);
    text-shadow: 1px 1px black, 2px 2px black, 3px 3px black, 4px 4px black,
      5px 5px black, 6px 6px black, 7px 7px black, 8px 8px black;
  }
}
</style>
