<template>
  <div class="logo-page">
    <div class="login">
      <div class="login-introduce">
        <h1>{{ $t('welcome') }}</h1>
        <p>{{ $t('login.introduce1') }}</p>
        <p>{{ $t('login.introduce2') }}</p>
        <p>{{ $t('login.introduce3') }}</p>
      </div>
      <div class="login-body">
        <el-image
          class="login-logo"
          :src="require('@/assets/images/cat2bug-logo.gif')"
        ></el-image>
        <el-form ref="loginForm" :model="loginForm" :rules="loginRules" class="login-form">
          <div class="login-form-title">
            <h3 class="title">{{$t("system-name")}}</h3>
            <span class="version">{{ version }}</span>
          </div>
          <el-form-item prop="username">
            <el-input
              v-model="loginForm.username"
              type="text"
              auto-complete="off"
              :placeholder="$t('account')"
            >
              <svg-icon slot="prefix" icon-class="user" class="el-input__icon input-icon" />
            </el-input>
          </el-form-item>
          <el-form-item prop="password">
            <el-input
              v-model="loginForm.password"
              type="password"
              auto-complete="off"
              :placeholder="$t('password')"
              @keyup.enter.native="handleLogin"
            >
              <svg-icon slot="prefix" icon-class="password" class="el-input__icon input-icon" />
            </el-input>
          </el-form-item>
          <el-form-item prop="code" v-if="captchaEnabled">
            <el-input
              v-model="loginForm.code"
              auto-complete="off"
              :placeholder="$t('verification-code')"
              style="width: 63%"
              @keyup.enter.native="handleLogin"
            >
              <svg-icon slot="prefix" icon-class="validCode" class="el-input__icon input-icon" />
            </el-input>
            <div class="login-code">
              <img :src="codeUrl" @click="getCode" class="login-code-img"/>
            </div>
          </el-form-item>
          <div class="between-row">
            <el-checkbox v-model="loginForm.rememberMe">{{$t("remember-password")}}</el-checkbox>
            <div class="lang-group">
              <svg-icon icon-class="lang_zh_CN" @mouseenter="changeLang('zh_CN')" />
              <svg-icon icon-class="lang_en_US" @mouseenter="changeLang('en_US')" />
            </div>
          </div>
          <el-form-item style="width:100%;">
            <el-button
              :loading="loading"
              size="medium"
              type="primary"
              style="width:100%;"
              @click.native.prevent="handleLogin"
            >
              <span v-if="!loading">{{$t("login")}}</span>
              <span v-else>{{$t("loading")}}...</span>
            </el-button>
            <div style="float: right;" v-if="register">
              <router-link class="link-type" :to="'/register'">{{$t("register-now")}}</router-link>
            </div>
          </el-form-item>
        </el-form>
        <span class="login-copyright">Copyright © 2023-2024 cat2bug.com All Rights Reserved.</span>
      </div>
    </div>
    <el-image
      class="login-mouse"
      style="width: 150px;"
      :src="require('@/assets/images/cat2bug-mouse.gif')"
    ></el-image>
  </div>
</template>

<script>
import { getCodeImg } from "@/api/login";
import Cookies from "js-cookie";
import { encrypt, decrypt } from '@/utils/jsencrypt'
import 'element-ui/lib/theme-chalk/display.css';
import {getVersion} from "@/api/version";
const I18N_LOCALE_KEY='i18n-locale'

export default {
  name: "Login",
  data() {
    return {
      lang: null,
      systemVersion: null,
      randDelay: Math.random()*10,
      codeUrl: "",
      loginForm: {
        username: "",
        password: "",
        rememberMe: false,
        code: "",
        uuid: ""
      },
      loginRules: {
        username: [
          { required: true, trigger: "blur", message: this.$i18n.t('please-enter-your-account') }
        ],
        password: [
          { required: true, trigger: "blur", message: this.$i18n.t('please-enter-your-account') }
        ],
        code: [{ required: true, trigger: "change", message: this.$i18n.t('please-enter-verification-code') }]
      },
      loading: false,
      // 验证码开关
      captchaEnabled: true,
      // 注册开关
      register: true,
      redirect: undefined
    };
  },
  watch: {
    $route: {
      handler: function(route) {
        this.redirect = route.query && route.query.redirect;
      },
      immediate: true
    }
  },
  computed: {
    version: function (){
      return this.systemVersion?'V'+this.systemVersion:'';
    }
  },
  created() {
    this.getSystemVersion();
    this.getCode();
    this.getCookie();
    this.lang = this.$cache.local.get(I18N_LOCALE_KEY);
    this.lang = this.lang||'zh_CN';
    this.$i18n.locale = this.lang;
  },
  methods: {
    getSystemVersion() {
      getVersion().then(res=>{
        this.systemVersion = res;
      })
    },
    getCode() {
      getCodeImg().then(res => {
        this.captchaEnabled = res.captchaEnabled === undefined ? true : res.captchaEnabled;
        if (this.captchaEnabled) {
          this.codeUrl = "data:image/gif;base64," + res.img;
          this.loginForm.uuid = res.uuid;
        }
      });
    },
    getCookie() {
      const username = Cookies.get("username");
      const password = Cookies.get("password");
      const rememberMe = Cookies.get('rememberMe')
      this.loginForm = {
        username: username === undefined ? this.loginForm.username : username,
        password: password === undefined ? this.loginForm.password : decrypt(password),
        rememberMe: rememberMe === undefined ? false : Boolean(rememberMe)
      };
    },
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          this.loading = true;
          if (this.loginForm.rememberMe) {
            Cookies.set("username", this.loginForm.username, { expires: 30 });
            Cookies.set("password", encrypt(this.loginForm.password), { expires: 30 });
            Cookies.set('rememberMe', this.loginForm.rememberMe, { expires: 30 });
          } else {
            Cookies.remove("username");
            Cookies.remove("password");
            Cookies.remove('rememberMe');
          }
          this.$store.dispatch("Login", this.loginForm).then(() => {
            this.$router.push({ path: this.redirect || "/" }).catch(()=>{});
          }).catch(() => {
            this.loading = false;
            if (this.captchaEnabled) {
              this.getCode();
            }
          });
        }
      });
    },
    changeLang(lang){
      this.$i18n.locale = lang;
      this.$cache.local.set(I18N_LOCALE_KEY,lang);
    }
  }
};
</script>

<style rel="stylesheet/scss" lang="scss">

@media screen and (max-width: 980px) {
  .login {
    justify-content: center;
  }
  .login-introduce {
    display: none;
  }
  .login-body {
    padding-left: 0px;
  }
}

@media screen and (min-width: 980px) {
  .login {
    justify-content: right;
    margin-left: 90px;
    margin-right: 90px;
  }
  .login-introduce {
    display: inline;
  }
  .login-body {
    padding-left: 90px;
  }
}

body {
  overflow: hidden;
}
.logo-page {
  height: 100%;
  overflow: hidden;
}
.login {
  display: flex;
  //justify-content: right;
  align-items: center;
  height: 100%;
  //background-image: url("../assets/img/login-background.jpg");
  background-size: cover;
  background-color: white;
}
.login-form-title {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: flex-end;
  padding: 20px;
  .title {
    margin: 0px 10px;
    text-align: center;
    color: #707070;
  }
  .version {
    font-size: 14px;
    color: #707070;
  }
}

.login-introduce {
  float: right;
  max-width: 45%;
  font-size: 22px;
  padding-right: 80px;
  border-right: 1px solid #DCDFE6;
}
.login-mouse {
  position: absolute;
  bottom: 0px;
  position: absolute;
  animation: move 16s linear;
  animation-iteration-count: infinite;
  left: 100%;
  @-webkit-keyframes move {
    0%   {
      left: 100%;
      transform: rotateY(0deg);
    }
    15%  {
      left: calc(0% - 200px);
      transform: rotateY(0deg);
    }
    41% {
      left: calc(0% - 200px);
      transform: rotateY(-180deg);
    }
    55%   {
      left: 100%;
      transform: rotateY(-180deg);
    }
    56%  {
      left: 100%;
      transform: rotateY(0deg);
    }
    100%  {
      left: 100%;
      transform: rotateY(0deg);
    }
  }
}

.login-body {
  float: right;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.login-logo {
  width: 500px;
  height: 400px;
  z-index: 2;
  margin-top: -100px;
}
.login-form {
  margin-top: -77px;
  border-radius: 6px;
  background: #ffffff;
  width: 500px;
  margin-bottom: 10px;
  padding: 25px 25px 5px 25px;
  border:3px solid #5A5A59;
  .el-input {
    height: 38px;
    input {
      height: 38px;
    }
  }
  .input-icon {
    height: 39px;
    width: 14px;
    margin-left: 2px;
  }
}
.login-tip {
  font-size: 13px;
  text-align: center;
  color: #bfbfbf;
}
.login-code {
  width: 33%;
  height: 38px;
  float: right;
  img {
    cursor: pointer;
    vertical-align: middle;
  }
}
.login-copyright {
  font-family: Arial;
  font-size: 14px;
  color: #606266;
}
.login-code-img {
  height: 38px;
}
.between-row {
  width: 100%;
  display: inline-flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
  .lang-group {
    display: inline-flex;
    flex-direction: row;
    justify-content: flex-end;
    > * {
      font-size: 22px;
      margin-left: 3px;
    }
  }
}
</style>
